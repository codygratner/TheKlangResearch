# 💀 QA Autopsy: The Phantom Bridge & In-Canvas Resolution
- **Subagent Conversation ID**: `e04485c4-78e3-43e6-8bed-477598118b18`
- **Role**: Principal Systems & AI Harness QA Architect (Pro High)
- **Target**: `c:\Dev\TheKlangSuite\.agents\sidecar\nkai_deep_research.html` & `tools/nkai_bridge.py`
- **Timestamp**: `2026-10-09T18:40:09Z`
- **Category**: Webview IPC, Local Loopbacks & Human-in-the-Loop Interaction Design

---

## 1. Symptoms & Initial Defect
When viewing an interactive sidecar canvas (`nkai_deep_research.html`) running as an Antigravity artifact in Chromium/Electron (`file:///C:/Users/...`):
1. The user clicked action buttons such as `[ ⚖️ Request Pros & Cons ]` or `[ 🔬 Research Online ]`.
2. The UI made an HTTP POST to `http://127.0.0.1:4040/api/action`.
3. `tools/nkai_bridge.py` appended the action JSON to `.agents/pipeline/inbox/action_queue.jsonl` and returned `HTTP 200 OK`.
4. The webview displayed a prominent toast: `⚡ Sent to Agent!`, played an audio chime, and lit up a green LED: `⚡ Bridge: Online (:4040) - Clicks dispatch directly to agent`.
5. **The Agent sat completely silent.** Antigravity's LLM engine is strictly event-driven and conversational. Between turns, the language model is halted, waiting for a native `USER_INPUT` event. Writing to a text file on disk cannot wake up an idle LLM or simulate keyboard input into the IDE chat box.
6. The user waited for a response that never came, concluding that the buttons and bridge were broken.

---

## 2. Root Cause & Platform Trap
* **The "Phantom Bridge" Fallacy**: The Python loopback bridge `:4040` was an architectural workaround attempting to bridge the gap between sandboxed `file:///` webviews and the active LLM agent. While it successfully handled file I/O, it created a phantom communication channel—promising autonomous agent execution without a delivery mechanism.
* **Fake Bridge vs Native UI Extension**: True Antigravity UI Extensions running in `plugin/` expose `window.sidecar.agent.sendMessage()`, which uses the IDE's internal WebSocket pipe to initiate genuine chat turns. In `file:///` mode, `window.sidecar` is undefined, and Electron's security sandbox prevents webviews from interacting with parent IDE processes.
* **The UI Dishonesty Trap**: Telling the user "Sent to Agent!" when the agent cannot see or respond to the message breaks developer trust and violates human-in-the-loop mental models.

---

## 3. Minimal Reproducible Proof
```javascript
// Webview sends telemetry to local Python process:
fetch('http://127.0.0.1:4040/api/action', {
    method: 'POST',
    body: JSON.stringify({ action: 'requestPCDA', topicId: 't16' })
});
// Python server writes to disk:
# tools/nkai_bridge.py:
with open('.agents/pipeline/inbox/action_queue.jsonl', 'a') as f:
    f.write(json.dumps(payload) + '\n')
# Result: action_queue.jsonl has the row, but IDE chat prompt is completely empty,
# and agent conversation transcript remains unchanged until user types in chat.
```

---

## 4. Permanent Architectural Invariant / Rule
> **DATA BELONGS IN THE CANVAS; ACTIONS BELONG IN THE CLIPBOARD.**
>
> 1. Never promise autonomous agent execution in `file:///` mode where native WebSocket hooks (`window.sidecar.agent.sendMessage()`) are absent.
> 2. Static or pre-computed data (pros/cons, research dossiers, benchmarks, trade-off matrices) must expand **in-canvas** with 0ms latency, zero token burn, and zero network dependency.
> 3. Dynamic actions requiring LLM generation must honestly format and highlight the prompt inside a dedicated, readonly `<input>` element (`onclick="this.select();"`), allowing 1-click highlighting and manual `Ctrl+C` / `Ctrl+V` handoff.
> 4. The local loopback bridge `:4040` is demoted strictly to an optional background telemetry logger.

---

## 5. Concrete Resolution & Implementation Snippet
```javascript
// 1. Honest In-Canvas Progressive Disclosure:
function toggleOptionAccordion(topicId, optLetter, panelType, btn) {
    const panel = document.getElementById(`expanded-${panelType}-${topicId}-${optLetter}`);
    if (!panel) return;
    const isExpanded = panel.style.display !== 'none';
    panel.style.display = isExpanded ? 'none' : 'block';
    btn.classList.toggle('active', !isExpanded);
    playHapticTick();
}

// 2. Rock-Solid Readonly Click-to-Highlight Input:
// <input type="text" readonly value="[Prompt]" onclick="this.select();" ondblclick="this.select();" />

// 3. Passive Telemetry (Non-Blocking):
if (window.nkaiBridgeOnline) {
    fetch(`http://127.0.0.1:${port}/api/action`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ action, topicId, prompt })
    }).catch(() => {});
}
```
