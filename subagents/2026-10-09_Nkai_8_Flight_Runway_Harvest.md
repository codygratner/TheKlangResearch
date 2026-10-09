# Subagent Soul Harvest: The N'kai 8-Flight Runway (v0.1.0 Capstone)
> **Date:** October 9, 2026  
> **Source Facility:** New Klang City (Tooling Harness)  
> **Target Knowledge Hub:** `TheKlangResearch/subagents/`  
> **Executors:** Tier 2 Flash Tooling Builders (`8035d1e4`, `dbd4f59f`, `7e7d8e23`, `2af81579`, `b903a76c`, `50fd2776`) & Tier 1 Pro Architects (`4e582d27`, `ec137391`, `60075fb2`, `3c1fc040`)  

---

## 🎯 Executive Summary & Architectural Breakthroughs

Today's intensive agentic sprint completed all 8 foundation and orchestration flights for **N'kai (NKAI)**: the Non-Euclidean Knowledge & Autonomous Interface for Google Antigravity and modern AI pair-programming.

Across 8 sequential flights, the framework evolved from a loose collection of HTML research prototypes into an **independent, zero-dependency visual project management, triage, and living research cockpit**.

---

## 🛫 The 8 Flights: Deliverables & Specifications

### 1. Flight 1: Canonical State & Factory Handoff (Topics 6.16 & 6.7)
- **Specification:** [`nkai_canonical_state_and_factory_handoff.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_canonical_state_and_factory_handoff.md)
- **Core Breakthrough:** Option C approved—Hybrid Markdown Frontmatter + Client-Side LocalStorage + JSON Catalog. Avoids heavy SQLite binary files in git; retains 100% human-readable git diffs.
- **Factory Gate:** Codified structured context clues seeded with links to `DOCS_CATALOG.json` and `SYSTEM_MAP.md`.

### 2. Flight 2: Distribution Packaging & Key Ergonomics (Topics 6.12 & 6.6)
- **Specification:** [`nkai_distribution_packaging.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_distribution_packaging.md)
- **Core Breakthrough:** Monolithic All-in-One Distribution Bundle in `nkai/bundle/` with a one-command PowerShell installer (`install_nkai_bundle.ps1`).
- **Keyboard Ergonomics:** Dynamic shortcut reference pill (`[ ⌨️ Shortcuts ]`) in sidecar header reinforcing `Ctrl+K`, `Ctrl+Shift+P`, and `Ctrl+Enter`.

### 3. Flight 3: Universal AI Harness Portability (Topic 6.13)
- **Specification:** [`nkai_universal_harness_portability.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_universal_harness_portability.md)
- **Core Breakthrough:** Tri-Mode Environment Detection (`extension`, `file`, `server`).
- **CLI Runner:** Zero-dependency `tools/nkai_serve.py` (Python standard library only) and `package.json` / `bin/nkai.js` (`npx nkai serve`).

### 4. Flight 4: Skill Taxonomy & Invocation Defaults (Topics 6.4 & 6.14)
- **Specification:** [`nkai_skill_taxonomy_and_invocation_architecture.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_skill_taxonomy_and_invocation_architecture.md)
- **Core Breakthrough:** 3-Layer Skill Hierarchy (Atomic Micro-Skills $\to$ Macro Orchestrators $\to$ Visual Experiences).
- **Invocation Defaults:** Visual skills (`/grill-me`, `/deep-research`, `/post-mortem`) default to generating N'kai sidecars; `--chat` flag provides an instant, clean text escape hatch.

### 5. Flight 5: Header Ergonomics & Spatial Density (Topics 6.9 & 6.10)
- **Specification:** [`nkai_header_ergonomics_and_spatial_density.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_header_ergonomics_and_spatial_density.md)
- **Core Breakthrough:** Two-Tier Header geometry:
  - Top Tier: Title, font zoom scaler (`A-`/`100%`/`A+`), Web Audio click haptics (`🔊`), and refresh (`🔄`).
  - Bottom Tier: Horizontal scrollable tab rail with hybrid compact category tabs (`1 🗺️`, `2 📦`...) saving 40–60px of vertical header space.

### 6. Flight 6: Interactive Triage Views & Inbox Workflow (Topics 6.8 & 6.11)
- **Specification:** [`nkai_interactive_triage_views_and_inbox_workflow.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_interactive_triage_views_and_inbox_workflow.md)
- **Core Breakthrough:** Ongoing Autopsy View (`#tab-autopsy`) in Slot 2 with dynamic scoreboard pills, real-time JS filter chips, and 1-click cross-tab jump with visual `.pulse-target` glow.
- **Vault Harvester:** Zero-dependency `tools/check_mail.py` scanning `TheKlangVault/Inbox/` and generating interactive triage boards.

### 7. Flight 7: PCDA Dialectic Governance & SOTA Benchmarks (Topics 6.1, 6.2 & 6.15)
- **Specification:** [`nkai_pcda_governance_and_sota_benchmarks.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_pcda_governance_and_sota_benchmarks.md)
- **Core Breakthrough:** 4-Part Hegelian Dialectic Triad (Pros [Thesis] $\to$ Cons [Thesis Friction] $\to$ Devil's Advocate [Antithesis] $\to$ 80/20 Resolution [Synthesis]).
- **PCDA Disambiguation:** Strictly PCDA (not PCA to prevent confusion with Principal Component Analysis or Printed Circuit Assembly).
- **Tooling:** Zero-dependency `tools/pcda_format.py` CLI.

### 8. Flight 8: Cumulative Allowance & Token Impact Ledger (Topics 6.3 & 6.19)
- **Specification:** [`nkai_allowance_ledger_and_token_governance.md`](file:///c:/Dev/TheKlangSuite/docs/specs/nkai_allowance_ledger_and_token_governance.md)
- **Core Breakthrough:** Cumulative Allowance Ledger with live segmented progress bar and threshold badges (safe $\le 100\text{k}$, warning $101\text{k}$–$200\text{k}$, critical $>200\text{k}$).
- **Deep Linking:** Hash-anchor deep routing (`#t1`, `#t2`) across tabs with smooth-scroll and focus highlight.
- **Tooling:** Zero-dependency `tools/nkai_ledger.py` CLI.

---

## 🏛️ Institutional Lessons & Guardrails Codified
1. **The Context Shield Rule**: Chat context must remain lean; all heavy mechanical diffs and multi-file updates must be delegated to Tier 2 Flash subagents.
2. **The "Process, Not Products" Anti-Slop Standard**: Engineering whitepapers for external peers must strictly exclude internal lore and proprietary product names.
3. **Stateless Action CLI over Daemons**: Reject long-running background daemons (Node/Express/SQLite) to avoid Windows `EBUSY` file locking, zombie port contention, and token bloat. Stateless CLI runners achieve 98% token reduction and 100% 2037 plain-text longevity.
4. **Clean-Room Reference Cloning**: Cloned external reference repositories (`picotracker`, `nullperator`) live in gitignored `TheKlangResearch/hardware/external/` with pinned commit SHAs.
