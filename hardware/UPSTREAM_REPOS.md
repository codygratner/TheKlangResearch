# 🎛️ Upstream Hardware Repositories Ledger

This ledger tracks third-party hardware reference implementations, vendor SDKs, and platform firmwares for the **dadamachines TBD-16** ecosystem (ESP32-P4 + RP2350B + ESP32-C6).

To prevent Git repository bloat, dirty submodule errors, and accidental sync into the Obsidian knowledge vault, **upstream source repositories are gitignored** and should be cloned into `hardware/upstream/`.

---

## Tracked Upstream Repositories

| Component | Upstream GitHub Repository | Default Branch | Upstream Author / License | Role in Ecosystem |
| :--- | :--- | :--- | :--- | :--- |
| **CTAG-TBD Platform** | [dadamachines/ctag-tbd](https://github.com/dadamachines/ctag-tbd) | `main` | dadamachines / Robert Manzke (GPLv3) | Audio codec drivers, ESP-IDF hardware layer, pinouts |
| **TBD-16 DSP Engine** | [dadamachines/tbd-16-dsp](https://github.com/dadamachines/tbd-16-dsp) | `main` | dadamachines (GPLv3) | ESP32-P4 audio DSP plugin host and algorithms |
| **TBD-16 UI Firmware** | [dadamachines/tbd-16-ui](https://github.com/dadamachines/tbd-16-ui) | `main` | dadamachines (GPLv3) | RP2350B 30-button, 4-knob, 2.4" OLED controller |

---

## Cloning Reference Code for Local Research

To clone these upstream repositories locally without affecting Git status:

```bash
mkdir -p hardware/upstream
cd hardware/upstream

git clone https://github.com/dadamachines/ctag-tbd.git CTAG-TBD
git clone https://github.com/dadamachines/tbd-16-dsp.git TBD-16-DSP
git clone https://github.com/dadamachines/tbd-16-ui.git TBD-16-UI
```

All folders inside `hardware/upstream/` and `hardware/external/` are automatically ignored by `.gitignore`.
