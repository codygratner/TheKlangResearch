# QA Autopsy: N'kai Webview IPC Bridge & Buried Toast Feedback
> **Date:** 2026-10-09  
> **Platform:** Google Antigravity / Electron / Chromium Side-Panel Webview (`file:///`)  
> **Target Subsystem:** N'kai Asymmetric Sidecar (`nkai_deep_research.html`), Local Loopback Micro-Bridge (`tools/nkai_bridge.py` on `:4040`)  
> **Classification:** Novel Platform Trap (Webview Sandbox, PNA, DOM Tab Invisibility)

---

### 1. Symptoms & Initial Defect
During interactive triage testing of the N'kai Deep Research sidecar, the user attempted to click `[ ✅ APPROVE Option C ]` on **Flight 1: Topic 6.16 Backend Persistence & State Management**.
- The approve button was clicked in the side panel.
- No visible notification or toast appeared on screen.
- No response was triggered in the IDE chat agent.
- The user reasonably concluded that the button had failed to fire or that the loopback IPC bridge had broken.

---

### 2. Root Cause & Platform Trap
A forensic audit by a Tier 1 Pro QA subagent (`44dc9b8c`) identified three compounding factors:

1. **The Buried Toast Trap (Hidden-Tab Invisibility)**:
   - The notification container (`#audio-toast`) was statically nested inside `#sec-audio-lab` on Tab 1 (`#tab-loaders`).
   - When the user was actively working in `#tab-workshop` (Active Workshop) or `#tab-audit` (Sanity Audit), calls to `showAudioToast(msg)` modified an invisible element inside a hidden tab (`display: none;`).
   - Although the IPC dispatch succeeded, the user received **zero visual feedback**, producing the false impression of an ignored click.

2. **Static DOM Pre-Population Desynchronization**:
   - In `nkai_deep_research.html`, lines 950–967 bound the Active Workshop header to `Topic 6.16: Backend Persistence`.
   - However, `#workshop-card-mount` contained legacy static HTML pre-populated with `Topic 6.15: Industry Workflow Precedents` and an approval button calling `setAuditVerdict(window.currentWorkshopTopic || 't15', 'APPROVED', this)`.
   - If the DOM rendered before `setWorkshopTopic('t16')` completed its dynamic injection, clicking the button mistakenly submitted an approval for `t15` rather than `t16`.

3. **Chromium Loopback IPC & Private Network Access (PNA) Headers**:
   - Modern Chromium enforces strict Private Network Access policies. When an untrusted/local `file:///` page fetches `http://127.0.0.1:4040`, Chromium sends an `Origin: null` header and triggers an `OPTIONS` preflight request with `Access-Control-Request-Private-Network: true`.
   - If the local bridge does not reply with `Access-Control-Allow-Private-Network: true` and wildcard/null CORS origin reflection, the fetch is silently dropped before hitting application logic.

---

### 3. Minimal Reproducible Proof
Automated verification was conducted using Microsoft Edge in headless mode via the Chrome DevTools Protocol (CDP WebSocket connection to remote debugging port 9226):

```javascript
// Programmatic click execution against ws-btn-approve
const btn = document.getElementById('ws-btn-approve');
btn.click();
```

- Headless CDP evaluation confirmed that when the bridge was online, the request returned HTTP 200 and appended a valid JSONL entry to `.agents/pipeline/inbox/action_queue.jsonl`.
- However, inspecting `window.getComputedStyle(document.getElementById('audio-toast')).display` confirmed the toast container was completely unrendered on the screen.

---

### 4. Permanent Architectural Invariant / Rule
1. **Universal Floating Toast HUD**: User-facing action feedback inside Antigravity sidecars MUST NEVER live inside individual tab containers or section cards. All toast/dispatch notices MUST render into a persistent, fixed HUD injected at the root of `document.body`:
   ```css
   #nkai-floating-toast {
       position: fixed;
       bottom: 24px;
       right: 24px;
       z-index: 99999;
       backdrop-filter: blur(12px);
       pointer-events: none;
   }
   ```
2. **Synchronized Static Mounts**: Static HTML placeholders inside dynamic hero mounts (`#workshop-card-mount`) MUST match the active default topic on load (`t16`), preventing race-condition desynchronizations before `DOMContentLoaded`.
3. **PNA & CORS Header Guarantee**: All local loopback IPC servers (`tools/nkai_bridge.py`) MUST return:
   - `Access-Control-Allow-Private-Network: true`
   - `Access-Control-Allow-Origin: *` (or reflection of `Origin: null`)
   - `Access-Control-Allow-Methods: GET, POST, OPTIONS`

---

### 5. Concrete Resolution & Fix
1. Injected `#nkai-floating-toast` dynamically into `document.body` with smooth slide/fade cubic-bezier keyframe animations.
2. Refactored `setAuditVerdict` and `dispatchNkaiAction` to execute a unified `onSuccessUI` callback that triggers *only* upon confirmed IPC delivery (or fallback modal display).
3. Pre-populated `#workshop-card-mount` with complete Topic 6.16 HTML.
4. Verified end-to-end with automated Edge CDP testing.
