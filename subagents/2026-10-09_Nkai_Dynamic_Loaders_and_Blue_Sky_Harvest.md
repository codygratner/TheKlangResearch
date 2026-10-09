# 🌌 N'kai (NKAI): Dynamic Loaders, Persistent Drawers & Blue-Sky Innovations Soul Harvest

**Date:** 2026-10-09  
**Source Subagents:**
- `dbdeba46-88b7-4d2c-bc73-a5ee702c33a4` (Tier 1 Pro High: Blue-Sky Systems Architect)
- `9930aa56-ee03-4247-b872-e8f58151f77e` (Tier 2 Flash: Dynamic Data & Architecture Explorer)
- `bd4fd761-1b6f-4f38-9f15-fe0fe149864f` (Tier 2 Flash: Contrarian & Devil's Advocate)

---

## 1. Executive Summary & Synthesis

### Question 1: Header Roadmaps & Dynamic Loading
- Inlining multi-repo roadmaps into individual sidecar HTML files inflates templates by ~30KB–50KB and burns thousands of generation tokens.
- **The Local File CORS Wall:** In Chromium/Electron webviews, `fetch('roadmap.json')` in `file:///` contexts fails with opaque origin `null` (RFC 6454).
- **The Solution (The Hybrid Universal Adapter):** In Antigravity Extension mode, loads via Node.js Sidecar SDK / HTTP IPC. In standalone offline mode, loads via dynamic `<script src="data/roadmap.data.js">` setting `window.NKAI_DATA.roadmap = {...}`. 100% CORS-proof, zero build steps.
- The Roadmap is rendered inside a **Right-Anchored Slide-Over Drawer**, supporting multi-repo tabs (`The Klang Suite`, `ToadTracker`, `N'kai`), checklist progress meters, and deep-link anchor hashes (`#tks-v0.4.0`).

### Question 2: Header Sessions & Paused Triage History
- Paused triage packages in `.agents/sidecar/packages/*.json` currently suffer from zero UI visibility.
- **The Solution:** A persistent header button: `📦 Sessions [1 PAUSED]` with a glowing amber badge whenever undecided items exist.
- Clicking opens the **Sessions & History Slide-Over Drawer**:
  - **Pinned Top Section:** Paused triage packages featuring a **1-click `[ ▶️ Resume in Workshop ]` button** that dynamically hot-reloads the package into N'kai's active hero stage and decision composer.
  - **Bottom Section:** Historical archived scorecards with authority signatures and harvest links.
  - Sourced from `packages/sessions_manifest.data.js` and `.json`.

### Question 3: Blue-Sky Features & Next-Level UX
1. **Interactive Command Palette (`Cmd+K` / `Ctrl+K`):** Spotlight-style fuzzy jumper across all roadmap milestones, active triage cards, and past harvest docs.
2. **Bi-Directional Action Prompters:** Quick-action pills on cards (e.g. `[ 🔬 Deep Dive ]`, `[ ⚖️ Pros/Cons ]`, `[ 🔨 Plan ]`) that inject pre-structured prompts into the chat clipboard.
3. **Procedural Web Audio Haptics (0 KB Audio):** Subtle, satisfying mechanical detent ticks and major-third harmonic approval chimes synthesized via native Web Audio API oscillators (with master mute toggle).
4. **2x2 Decision Matrix View:** Toggles post-mortem lists into an interactive Effort vs Impact quadrant (Quick Wins vs Strategic Bets vs Fill-Ins vs Strategy Graveyard).

---

## 2. Contrarian Audit & The Pragmatic 80/20 Standard

| Proposed Feature | The Inevitable Pitfall | The Pragmatic 80/20 Standard |
| :--- | :--- | :--- |
| **Dynamic JSON Roadmap** | `fetch('roadmap.json')` fails on `file:///` due to Chromium SOP | **Dual-Emission Generator:** Output both `.json` and synchronous `.data.js` |
| **Persistent Sessions Browser** | Ephemeral `localStorage` wiped on IDE webview cache reset | **Git-Backed Filesystem:** Canonical store is `.agents/sidecar/packages/*.json` |
| **Audio Haptics** | 0 dBFS blast during high-gain audio DSP testing | **100% Silent by Default:** Strictly opt-in via persistent UI toggle |
| **Header Real-Estate** | Crowding header with 10 buttons wraps into 4 lines in 380px panels | **Right-Anchored Slide Drawers:** Icon+badge triggers that collapse cleanly |

---

## 3. Universal Schemas & Data Contracts

### `roadmap.schema.json`
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "NKAI Multi-Repo Roadmap",
  "type": "object",
  "properties": {
    "version": { "type": "string" },
    "last_updated": { "type": "string", "format": "date-time" },
    "repositories": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" },
          "url": { "type": "string", "format": "uri" },
          "milestones": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "version": { "type": "string" },
                "title": { "type": "string" },
                "status": { "enum": ["planned", "active", "completed"] },
                "anchor": { "type": "string" },
                "tasks": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "id": { "type": "string" },
                      "description": { "type": "string" },
                      "status": { "enum": ["todo", "in_progress", "done"] }
                    },
                    "required": ["id", "description", "status"]
                  }
                }
              },
              "required": ["version", "title", "status", "anchor"]
            }
          }
        },
        "required": ["id", "name", "milestones"]
      }
    }
  },
  "required": ["version", "repositories"]
}
```

### `sessions_manifest.schema.json`
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "NKAI Sessions Registry",
  "type": "object",
  "properties": {
    "registry_version": { "type": "string" },
    "last_synced": { "type": "string", "format": "date-time" },
    "active_paused_count": { "type": "integer" },
    "sessions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "title": { "type": "string" },
          "date": { "type": "string", "format": "date-time" },
          "status": { "enum": ["PAUSED", "IN_PROGRESS", "COMPLETED"] },
          "authority": { "type": "string" },
          "script_file": { "type": "string" },
          "json_file": { "type": "string" },
          "harvest_doc": { "type": "string" },
          "total_items": { "type": "integer" },
          "scorecard": {
            "type": "object",
            "properties": {
              "approved": { "type": "integer" },
              "tabled": { "type": "integer" },
              "killed": { "type": "integer" },
              "pending": { "type": "integer" }
            }
          }
        },
        "required": ["id", "title", "status", "json_file"]
      }
    }
  }
}
```
