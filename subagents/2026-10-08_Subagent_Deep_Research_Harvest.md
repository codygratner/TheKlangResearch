# 🧬 The Klang Suite: Subagent Deep DSP & Architecture Soul Harvest
> **Date:** 2026-10-08  
> **Source:** Autonomous Pro Research Subagents (Gemini 3.1 Pro)  
> **Status:** Permanent Technical Archive (Decoupled from Daily Dev Logs)  

---
## 🛰️ TBD-16 Hardware Architecture & ToadTracker Synergy
**Subagent Conversation ID:** 111d83b8-5047-4909-ac00-2e8b0238ccce  

I have investigated the architecture of both ToadTracker and The Klang Suite (specifically the TBD-16 hardware milestone). Here is a structured summary of the findings and mapping strategies:

### 1. Hardware Constraints of the TBD-16
*   **Processors:** Dual-architecture with an ESP32-P4 RISC-V @ 400 MHz dedicated to Audio DSP, and an RP2350B @ 150 MHz handling UI, sequencing, and the display.
*   **Memory:** Features PSRAM, but strict engineering guardrails demand **zero dynamic allocation** (`malloc`, `free`, `new`, `std::vector::push_back`) in the audio path.
*   **Display:** 2.4" OLED, 240x240 resolution. It renders ultra-crisp 1-bit monochrome vector graphics.
*   **Controls:** 
    *   **4 Endless Push-Rotary Encoders**.
    *   **30-Button Grid**: Includes 16 Tactile RGB Step Buttons, D-pad, A/B/X/Y switches, L/R Page buttons, Record, and Transport switches.

### 2. ToadTracker Architecture Mapping
*   **Multi-Core & Concurrency:** Core 0 (or the RP2350B) handles the UI and screen rendering, while Core 1 (the ESP32-P4) manages the Audio DMA buffer synthesis. Communication is strictly via Single-Producer Single-Consumer (SPSC) lock-free ring buffers or atomics.
*   **Input Abstraction:** Operations are mapped to an 8-button minimum logical standard (D-pad, A, B, Opt, Edit, etc.).
*   **DSP Optimization:** Avoids all standard `<cmath>` transcendentals. Uses a custom `FastMath.h` for branchless math (e.g., `fastTanh`, phase accumulation) to prevent CPU stalls and denormal issues.

### 3. Recommendations: Distilling the "4-Control Mega Modules" to TBD-16
The Klang Suite's recent "v0.4.0 Neo-Slate" update deliberately adopted a **4-Controls-Per-Card architecture**, which perfectly anticipates the TBD-16 paradigm.

*   **1:1 Encoder-to-Parameter Mapping:** Each module/card from The Klang Suite (e.g., Voice 1, Pre-Amp Console, specific FX slots) translates to a single screen "page". The 4 parameters on that card map 1:1 to the 4 physical endless push-encoders.
*   **OLED UX (The 4-Bar Display):** Distill the desktop Neo-Slate UI down to 4 stacked horizontal meter bars on the 240x240 OLED. This monochrome 1-bit vector layout provides immediate, high-contrast feedback for the 4 active encoder values.
*   **Card Navigation:** Utilize the 16 tactile step buttons for instantaneous "Direct Card Jumps" (e.g., Buttons 1-8 for core Synth cards, 9-12 for FX slots), bypassing slow sequential scrolling. The dedicated Left/Right Screen buttons handle sequential page cycling.
*   **Tactile Gestures (Replacing Desktop Clicks):**
    *   *Turn Encoder:* Smooth parameter adjustment.
    *   *Push Encoder (Click):* Replaces the desktop "Double-Click/Alt-Click" to instantly reset a parameter to its default value.
    *   *Push-and-Hold (or Opt Switch + Push):* Replaces the desktop "Right-Click". This triggers the "Popover Callouts" (like the Master Limiter), temporarily hijacking the OLED and 4 encoders to control the popover's sub-parameters until released.
*   **The d6 Randomizer:** Map the contextual d6 randomization engine to a dedicated A/B/X/Y hardware switch. Pressing it randomizes the currently active 4-parameter card on the screen.
*   **DSP Porting:** Ensure The Klang Suite's DSP blocks use ToadTracker's `FastMath.h` instead of standard `std::sin` or `std::pow` to maintain performance on the 400 MHz ESP32-P4, and strictly adhere to the pre-allocation initialization rules.

---

## 🛰️ Pitch-Tracked Harmonic Aliasing (Noise Engineering Style)
**Subagent Conversation ID:** 711ed52a-bd0b-44c4-8b7d-3df00a671c18  

<div class="card" style="margin-bottom: 2rem;">
  <h2>Active Workshop: Pitch-Tracked Aliasing (NE-Style)</h2>
  
  <div class="section">
    <h3>The 'Why'</h3>
    <p>Typical digital oscillators suffer from <em>inharmonic aliasing</em> when high-frequency harmonics exceed the Nyquist limit, reflecting back as metallic, dissonant noise. Modules like Noise Engineering's Basimilus Iteritas Alter embrace aliasing by employing a dynamically adjustable sample rate. By perfectly locking the sample clock to a multiple of the oscillator's fundamental pitch, all fold-over frequencies are mathematically forced to land on the harmonic series. The result is aggressive, biting digital grit that remains completely musical and tonally coherent.</p>
  </div>

  <div class="section">
    <h3>The DSP Math</h3>
    <p>Normally, aliased frequencies fold back according to the formula: <code>f_alias = | k * f_s - f_orig |</code>.</p>
    <p>If we force the sample rate (<code>f_s</code>) to be an exact integer multiple (<code>N</code>) of the fundamental frequency (<code>f_root</code>), we get <code>f_s = N * f_root</code>.</p>
    <p>If the original signal is harmonic, its overtones are <code>M * f_root</code>. The folded aliasing then becomes: <code>| k * N * f_root - M * f_root | = | (k*N - M) * f_root |</code>.</p>
    <p>Since <code>k, N, and M</code> are all integers, the resulting aliased frequency is always an integer multiple of <code>f_root</code>—i.e., a perfectly in-tune harmonic overtone.</p>
  </div>

  <div class="section">
    <h3>Implementation Cost</h3>
    <ul>
      <li><strong>CPU Overhead:</strong> Moderate to High. Requires computing variable rate decimation/interpolation per voice, or running a dedicated resampler per oscillator block.</li>
      <li><strong>Architecture Shift:</strong> You cannot run a fixed 48kHz / 96kHz process block naively. The phase accumulator increments must be adapted for dynamic sub-sample processing, or the oscillator must be generated at the pitch-tracked rate and then bandlimited-resampled back to the DAW's host sample rate.</li>
      <li><strong>Clock Management:</strong> Calculating exact fractional sample drops to align the variable clock with the fixed host audio buffer size requires precise state-tracking.</li>
    </ul>
  </div>

  <div class="section">
    <h3>Visual Diagram: Aliasing Behavior</h3>
    <div style="display: flex; gap: 1rem; margin-top: 1rem; align-items: stretch; font-family: sans-serif;">
      <div style="flex: 1; border: 1px solid #444; border-radius: 8px; padding: 1rem; background-color: #1e1e1e;">
        <h4 style="margin-top: 0; color: #ff5555; border-bottom: 1px solid #333; padding-bottom: 0.5rem; font-size: 1rem;">Standard Fixed Rate (48kHz)</h4>
        <div style="display: flex; flex-direction: column; gap: 0.5rem; margin-top: 1rem;">
          <div style="display: flex; justify-content: space-between; font-size: 0.85em; color: #aaa;">
            <span>Root (100Hz)</span>
            <span>Harmonics</span>
          </div>
          <div style="height: 4px; background: #333; position: relative; border-radius: 2px;">
            <div style="position: absolute; left: 10%; width: 4px; height: 16px; background: #88ff88; top: -6px;"></div>
            <div style="position: absolute; left: 30%; width: 4px; height: 12px; background: #88ff88; top: -4px;"></div>
            <div style="position: absolute; left: 50%; width: 4px; height: 12px; background: #88ff88; top: -4px;"></div>
            <div style="position: absolute; left: 83%; width: 4px; height: 12px; background: #ff5555; top: -4px;"></div>
            <div style="position: absolute; left: 42%; width: 4px; height: 12px; background: #ff5555; top: -4px;"></div>
          </div>
          <p style="font-size: 0.85em; color: #ccc; margin-top: 0.5rem; line-height: 1.4;">Aliased frequencies (red) fall randomly between harmonics, causing metallic dissonance.</p>
        </div>
      </div>
      
      <div style="flex: 1; border: 1px solid #444; border-radius: 8px; padding: 1rem; background-color: #1e1e1e;">
        <h4 style="margin-top: 0; color: #55ff55; border-bottom: 1px solid #333; padding-bottom: 0.5rem; font-size: 1rem;">Pitch-Tracked Clock (N &times; Root)</h4>
        <div style="display: flex; flex-direction: column; gap: 0.5rem; margin-top: 1rem;">
          <div style="display: flex; justify-content: space-between; font-size: 0.85em; color: #aaa;">
            <span>Root (100Hz)</span>
            <span>Harmonics</span>
          </div>
          <div style="height: 4px; background: #333; position: relative; border-radius: 2px;">
            <div style="position: absolute; left: 10%; width: 4px; height: 16px; background: #88ff88; top: -6px;"></div>
            <div style="position: absolute; left: 30%; width: 4px; height: 12px; background: #88ff88; top: -4px;"></div>
            <div style="position: absolute; left: 50%; width: 4px; height: 12px; background: #88ff88; top: -4px;"></div>
            <div style="position: absolute; left: 70%; width: 4px; height: 12px; background: #ffaa00; top: -4px;"></div>
            <div style="position: absolute; left: 90%; width: 4px; height: 12px; background: #ffaa00; top: -4px;"></div>
          </div>
          <p style="font-size: 0.85em; color: #ccc; margin-top: 0.5rem; line-height: 1.4;">Aliased frequencies (orange) fold perfectly onto existing harmonic grid intervals.</p>
        </div>
      </div>
    </div>
  </div>
</div>

---

## 🛰️ Alpha Juno IR3R05 Dual-Stage Cascaded Resonant VCF
**Subagent Conversation ID:** fe58682f-6336-4fe9-98cf-f0c4438d66d5  

<div class="card">
    <div class="card-header">
        <h3 class="card-title">Active Workshop: Dual-Stage Resonant VCF (Alpha Juno IR3R05)</h3>
    </div>
    <div class="card-body">
        <div class="section">
            <h4>The 'Why': Acid Bite and Low-End Preservation</h4>
            <p>Traditional 4-pole analog filters (like the Moog transistor ladder or the Roland IR3109 used in the Juno-106) employ a <strong>global negative feedback loop</strong> across all four 1-pole stages to create resonance. Because this feedback is active at DC, increasing resonance mathematically suppresses the low-frequency gain, causing the classic "bass drop" when resonance is cranked.</p>
            <p>The Roland IR3R05 chip (used in the Alpha Juno and JX-8P) takes a different approach. It cascades <strong>two 2-pole State Variable Filters (SVFs)</strong>. The resonance is localized within each 2-pole stage via bandpass feedback. Since a bandpass filter has zero gain at DC, this localized feedback does not attenuate the fundamental frequencies. This preserves massive low-end punch even at high resonance—a key ingredient in the famous Alpha Juno "Hoover" sound and aggressive acid basslines.</p>
        </div>
        
        <div class="section">
            <h4>The 'DSP Topology'</h4>
            <p>To recreate this in modern C++ DSP, we move away from 4-pole global feedback models and instead instantiate two identical Zero-Delay Feedback (ZDF) 2-pole SVFs in series.</p>
            <ul>
                <li><strong>Cascaded Stages:</strong> The audio signal passes through SVF 1, then SVF 2.</li>
                <li><strong>Shared Cutoff:</strong> Both stages share the exact same cutoff frequency (<i>f_c</i>).</li>
                <li><strong>Localized Q:</strong> The global Resonance parameter is mapped to the damping factor (<i>R</i> or <i>Q</i>) of each individual SVF simultaneously.</li>
                <li><strong>No Global Feedback:</strong> There is no macro-feedback wrapping from the output of Stage 2 back to the input of Stage 1.</li>
            </ul>
        </div>
        
        <div class="section">
            <h4>Klang Suite Integration</h4>
            <p>This topology can be seamlessly added as a selectable "Alpha 24dB" model in our 4-control filter module.</p>
            <p><strong>C++ Implementation Blueprint (Topology):</strong></p>
            <pre><code>// Instantiation
juce::dsp::StateVariableTPTFilter&lt;float&gt; stage1, stage2;

// Parameter Update
float r = mapResonance(normalizedRes);
stage1.setCutoffFrequency(cutoffHz);
stage1.setResonance(r);
stage2.setCutoffFrequency(cutoffHz);
stage2.setResonance(r);

// Audio Loop (Per-Sample)
float sample = stage1.processSample(1, inputSample);
sample = stage2.processSample(1, sample);</code></pre>
        </div>

        <div class="section">
            <h4>Visual Comparison</h4>
            <pre><code class="language-mermaid">flowchart TD
    subgraph Traditional 4-Pole (Juno-106 / IR3109)
        direction LR
        A1[In] --> S1[1-Pole] --> S2[1-Pole] --> S3[1-Pole] --> S4[1-Pole] --> O1[Out]
        S4 -- Global Feedback (-k) --> A1
    end

    subgraph Dual-Stage 2-Pole (Alpha Juno / IR3R05)
        direction LR
        A2[In] --> SVF1["2-Pole SVF\n(Local Q)"] --> SVF2["2-Pole SVF\n(Local Q)"] --> O2[Out]
        SVF1 -. BP Feedback .-> SVF1
        SVF2 -. BP Feedback .-> SVF2
    end
</code></pre>
        </div>
    </div>
</div>

---

## 🛰️ Elektron Octatrack Tuned Comb Filter & Pre-Wavefolder APF Phase Dispersion
**Subagent Conversation ID:** f1403915-a4de-4db8-a543-120e1c36b9ab  

# DSP Architecture Research Summary

## 1. Elektron Octatrack Comb Filter vs. Mutable Instruments Rings for Karplus-Strong Basslines

### The "Muddy" Nature of Rings
Mutable Instruments Rings uses Modal Resonator and Sympathetic String algorithms—essentially banks of parallel band-pass filters or grids of coupled delay lines. While brilliant for acoustic, chime, and bell-like tones, its architecture encourages polyphonic overlapping, sympathetic resonance, and slow, complex decay tails. It lacks a strict, rigid monophonic phase reset or "choke" mechanism. For sequenced, driving basslines, these overlapping low-frequency tails accumulate, creating a loose and muddy low-end.

### The Tight, Industrial Punch of the Octatrack Comb Filter
The Octatrack's Comb Filter is a textbook, highly optimized Karplus-Strong delay line. It excels at driving basslines due to strict monophonic voice management and precise transient control.

**Key Architecture & Parameters:**
- **Tuned Delay Line (Pitch/Frequency):** Sets the delay time $T = 1/f_0$.
- **Feedback (-100% to +100%):** 
  - **Positive (+):** Reinforces all harmonics, yielding a sawtooth/string-like timbre.
  - **Negative (-):** Cancels even harmonics and reinforces odd harmonics, yielding a hollow, square-like, sub-heavy timbre ideal for bass.
- **Damping (LPF):** A high-frequency damping filter in the feedback loop attenuates high-end per cycle, mimicking physical acoustic energy loss.
- **Monophonic Choke / Phase Reset:** Crucially, upon re-trigger, the Octatrack forcefully flushes or chokes the ringing of the previous note, ensuring zero low-frequency overlap between consecutive 16th notes.

**Dual-Mode Operation:**
1. **As an Oscillator:** It can be excited by an internal, sharp transient impulse (a synthesized click or noise burst) triggered by the sequencer, functioning as an independent Karplus-Strong synth voice.
2. **As an Insert FX:** It processes incoming audio. The transients of the incoming signal (e.g., a drum loop) act as the physical exciter. The comb filter forces a specific pitch onto the atonal rhythm, transforming percussive loops into tuned, rhythmic basslines.

**Core DSP Equation (Comb Filter):**
$$y[n] = x[n] + g \cdot (y[n-D] * h_{LP}[n])$$
*Where $D$ is the delay time in samples ($F_s/f_0$), $g$ is the feedback coefficient, and $h_{LP}$ is the damping filter impulse response.*

---

## 2. Pre-Wavefolder All-Pass Filter (APF) Phase Dispersion

### The West-Coast Technique
In classic West-Coast synthesis (e.g., Buchla 259, Serge Wave Multipliers, Make Noise 0-Coast), rich timbres are generated not by subtracting harmonics with a VCF, but by adding them via wavefolding. A foundational technique is placing a tunable 1-pole or 2-pole All-Pass Filter (APF) immediately *before* an asymmetric wavefolder.

### How Phase Dispersion Alters Folding
1. **Phase Shift Without Amplitude Change:** An APF passes all frequencies at equal amplitude but introduces a frequency-dependent phase shift (group delay). 
2. **Reshaping the Waveform Peaks:** While the human ear is largely "phase-deaf" to a static harmonic spectrum, a wavefolder is strictly **amplitude-dependent**. By shifting the relative phases of a waveform's harmonics, the APF radically alters the physical geometry of the time-domain waveform, changing where and how high the peak amplitudes occur.
3. **Dynamic Formant Sweeps:** As the waveform's peaks are reshaped, different parts of the wave cross the wavefolder's fixed folding threshold at different times and intensities. Modulating the APF cutoff frequency dynamically morphs which harmonics are driven into the folds, resulting in distinct, vocal, vowel-like formant sweeps (phase-modulated folding) without ever using a traditional low-pass filter.

**Core DSP Equations:**

**1-Pole All-Pass Filter (Digital):**
$$H(z) = \frac{a + z^{-1}}{1 + a z^{-1}}$$
*Where $a$ controls the cutoff frequency of the phase shift.*

**Simplified Wavefolder (Sine Transfer Function):**
$$y[n] = \sin(k \cdot x_{APF}[n])$$
*Where $k$ is the drive/gain amount, and $x_{APF}[n]$ is the phase-dispersed output of the APF. In practice, piecewise linear or polynomial soft-clipping/folding functions are often used.*

---

## 🛰️ Pure C++20 Decoupling Audit for Embedded ESP32-P4 (libtks_dsp)
**Subagent Conversation ID:** f8e33cfd-3c3c-41ed-8dc8-77ddf1731e5d  

### JUCE Dependency Audit Results

I have audited the core C++ DSP codebase (`c:\Dev\TheKlangSuite\source\`) to evaluate decoupling the DSP engine from JUCE for the ESP32-P4 embedded hardware.

**Findings:** The DSP layer is **already 100% decoupled from JUCE**. 
- `ModularBlocks.h`, `DSPBlock.h`, `ModulationEngine.h`, `PlanterEngine.h`, and `FastMath.h` contain **absolutely zero** `juce::` namespace usages or JUCE header includes. 
- There are no `juce::dsp::*` filters, no `juce::SmoothedValue`, and no `juce::AudioBuffer` used in the core DSP layer.
- The DSP relies entirely on `TbdAudio::FastMath`, standard `<cmath>`, pre-allocated `std::vector`, lock-free `std::atomic`, and raw primitive arrays (`float*`).
- The JUCE boundary is strictly maintained in `FarmerProcessor.cpp`, which manually extracts raw pointers (`buffer.getWritePointer(0)`) and parses `juce::MidiBuffer` into primitive types (`velocity`, `midiNote`) before calling the pure C++ engine (`engine.processStereo(left, right, numSamples)` and `engine.trigger()`).

### Drop-in Decoupling Roadmap

Since the architecture is already cleanly separated, the decoupling roadmap is straightforward:

1. **CMake Target Isolation**: Create a new standalone static library target `add_library(libtks_dsp STATIC)` in CMake.
2. **Source Relocation**: Move the pure DSP headers (`ModularBlocks.h`, `DSPBlock.h`, `ModulationEngine.h`, `PlanterEngine.h`, `FastMath.h`) into an isolated `/dsp` subfolder to enforce a strict boundary preventing accidental JUCE inclusions in the future.
3. **Embedded API Wrapper**: Create a lightweight C++ wrapper for the ESP32-P4 firmware that maps hardware buffers and GPIO/ADC inputs directly to the existing `TbdAudio::ModularDrumEngine` public API:
   ```cpp
   class Tbd16Engine {
   public:
       void prepare(double sampleRate) { engine.init(sampleRate); }
       void process(float* left, float* right, int numSamples) { engine.processStereo(left, right, numSamples); }
       void setParam(int blockId, int paramIdx, float value) { engine.setPageParameter(blockId, paramIdx, value); }
       void trigger(float velocity, int note) { engine.trigger(velocity, note); }
   private:
       TbdAudio::ModularDrumEngine engine;
   };
   ```

### Effort Grade & Hardware Viability Assessment

- **Effort Grade:** **Low / Trivial**. The architectural heavy lifting (replacing standard library components with custom ZDF SVF filters and `FastMath`) has already been successfully implemented. No complex C++ rewrites or parameter smoothing replacements are required.
- **ESP32-P4 Hardware Viability:** **Excellent**. Because the DSP blocks strictly pre-allocate dynamic containers during initialization (`init()`), avoid blocking locks, and process data using cache-friendly contiguous raw `float*` buffers, the engine is fully real-time safe and highly portable. It will compile natively and run predictably on ESP32-P4's FreeRTOS / ESP-IDF toolchains with a deterministic memory footprint and excellent performance.

---

## 🛰️ Generative & Polymetric Sequencing Engines (Fugue Machine, Oxi One, Metropolix, Turing Machine)
**Subagent Conversation ID:** 2432806c-1797-4215-a597-31b4653f1e86  

Here is the structured technical proposal for integrating Generative & Alternative Sequencing Engines into The Klang Suite, specifically targeting the TKR-1 drum synth and TKF modulators.

# The Klang Suite: Generative Sequencing Architecture

This document outlines the core math, UX, and musical applications of four foundational generative paradigms, proposing how to integrate them via a unified "Klang-Brain" interface across TKR-1 (Triggers) and TKF (Continuous Modulation).

---

## 1. Multi-Playhead Counterpoint (Fugue Machine)
A single musical sequence read simultaneously by multiple virtual playheads, each operating with distinct time domains and transformations.

*   **Core Math:** A circular buffer of $N$ steps containing base sequence data. For $P$ playheads, each playhead $p$ maintains a phase index $idx_p$ calculated via the host clock: $idx_p = (clock \times div_p \times dir_p) \pmod N$. The output of a playhead is $Buffer[idx_p] \times scale_p + offset_p$.
*   **Musical Application:** 
    *   *TKR-1:* Complex, overlapping polyrhythmic percussion (e.g., hats running 16ths forward, snare running 8ths reversed, rimshot running dotted-8ths ping-pong).
    *   *TKF:* Phased LFOs where a single automation shape creates beautiful, interlocking modulation waves moving at different speeds.
*   **4-Control Mega Module (Fugue Engine):**
    1.  **Density:** Populates the base sequence buffer with triggers/events.
    2.  **Spread (Time):** Macro control that fans out the clock dividers (e.g., x1, x2, /2, /4) across the 4 playheads.
    3.  **Spread (Space):** Offsets the pitch, velocity, or modulation output range of each playhead.
    4.  **Directionality:** Morphs playhead directions (Forward $\rightarrow$ Ping-Pong $\rightarrow$ Reverse $\rightarrow$ Random).

---

## 2. Decoupled Polymetric Lanes (Matriceal / CYKLE)
Breaking the sequence into individual parameter lanes (Triggers, Velocity, Pitch, Gate) that each have arbitrary, decoupled lengths.

*   **Core Math:** Independent modulo counters. If Trigger length = $L_T$, Velocity length = $L_V$, Pitch length = $L_P$, the current state at tick $t$ is the intersection of $T[t \pmod{L_T}]$, $V[t \pmod{L_V}]$, and $P[t \pmod{L_P}]$. The overall pattern only repeats every $LCM(L_T, L_V, L_P)$ steps.
*   **Musical Application:**
    *   *TKR-1:* Ever-evolving drum grooves. A 5-step trigger pattern overlaid with a 7-step velocity pattern and a 3-step decay-time pattern creates a groove that feels cohesive but constantly shifts accents.
    *   *TKF:* Non-repeating modulators where the "shape" is 4 steps long but the "amplitude" is 5 steps long.
*   **4-Control Mega Module (Polymetric Engine):**
    1.  **Trigger Density:** Number of active events in the trigger lane.
    2.  **Lane Offset (Polymeter):** Expands the step-length differences between lanes (from locked 16/16/16 to decoupled 15/17/11).
    3.  **Parameter Variance:** Scales the depth of the modulation on the Velocity/Pitch lanes.
    4.  **Sync/Reset:** Probability that all lanes force-reset to step 1 to re-anchor the groove.

---

## 3. Stage-Based & Accumulators (Metropolix / M8)
Moving from absolute steps to "stages" that hold per a variable pulse count, combined with mathematical accumulators that mutate values per cycle.

*   **Core Math:** A sequence of stages $S_0, S_1... S_n$. Stage $i$ plays for $P_i$ pulses before advancing to $i+1$. 
    *   *Accumulator:* An internal variable $V$ updates via $V_t = (V_{t-1} + \Delta) \pmod{Max}$.
    *   *Sub-tick Mod:* Operations executed at fractions of a step (e.g., M8 tables applying pitch slides per hex-tick).
*   **Musical Application:**
    *   *TKR-1:* Instant ratcheting, trap-style hi-hat rolls, and structural fills. Accumulators can progressively raise the pitch of a snare roll or open a hi-hat decay over 4 bars.
    *   *TKF:* Stepped modulators that hold their value for irregular durations (e.g., hold for 3 pulses, hold for 1 pulse, hold for 4 pulses) while an accumulator creates a slow rising ramp underneath.
*   **4-Control Mega Module (Stage Engine):**
    1.  **Stage Pulses (Ratchets):** Morphs the number of pulses/repeats assigned to active stages.
    2.  **Accumulator Slope:** Sets the $\Delta$ added to pitch/cutoff/velocity every cycle (positive for rising, negative for falling).
    3.  **Table/Sub-tick Speed:** Determines how fast the sub-tick modulations run relative to the main clock.
    4.  **Probability (Conditionals):** Global threshold for "1-in-4" or "X% chance" conditions firing.

---

## 4. LFSR / Turing Machine (Labyrinth)
A Linear Feedback Shift Register generating a stream of controlled randomness via variable feedback probability.

*   **Core Math:** An $N$-bit shift register. On every clock tick, bits shift right. The new MSB is determined by checking a random float $R \in [0.0, 1.0]$ against a mutation probability $P_{mut}$. If $R < P_{mut}$, the MSB is a coin toss; otherwise, the LSB is fed back into the MSB. The integer value of the register is scaled to output.
*   **Musical Application:**
    *   *TKR-1:* Algorithmic, looping drum top-loops or bassline generation. Lock it to 0% for a repeating loop, nudge to 5% for an occasional mutated hit, or 100% for total white-noise chaos.
    *   *TKF:* The ultimate stepped-random LFO (Sample & Hold with memory). Beautiful for generative filter cutoff walks.
*   **4-Control Mega Module (LFSR Engine):**
    1.  **Probability (Lock/Write):** 0% (Locked Loop) to 100% (Random Write).
    2.  **Register Length:** 8, 16, or 32 bits (determines the maximum loop duration).
    3.  **Scale/Quantize:** Restricts the integer output to specific voltage ranges, scales, or drum voices.
    4.  **Slew (Glide):** Smooths the transition between stepped values.

---

## Integration: The "Klang-Brain" UI

To unify these paradigms without overwhelming the UI, we should introduce a dedicated **Klang-Brain** tab/page. 
*   **Engine Selector:** A central dropdown or stylized switch selects the active engine (Fugue, Polymetric, Stage, LFSR).
*   **Adaptive Mega Controls:** The 4 Mega Knobs/Sliders dynamically relabel and remap their DSP targets based on the selected engine, adhering to our 4-Control paradigm.
*   **Visualizer:** A dynamic circular or linear visualizer sits in the center, changing its representation (e.g., 4 orbiting playheads for Fugue, scrolling bits for LFSR, decoupled rings for Polymeter).

## Output Translation: TKR-1 vs. TKF

The Klang-Brain engine code will be shared in a core library, but its output is translated depending on the plugin host context:

1.  **TKR-1 (Trigger Mode):**
    *   The engine's normalized output $[0.0, 1.0]$ is heavily quantized.
    *   Generates discrete *Triggers* (Gates).
    *   Uses thresholding to determine Drum Select (e.g., $0.0 - 0.33$ = Kick, $0.34 - 0.66$ = Snare, $0.67 - 1.0$ = Hat).
    *   Passes discrete Velocity and Pitch values via MIDI/internal routing.
2.  **TKF (Continuous Modulation Mode):**
    *   The engine's normalized output $[0.0, 1.0]$ is treated as continuous CV.
    *   Employs interpolation (Slew/Glide) to smooth stepped values into LFO shapes.
    *   Output is mapped to arbitrary plugin parameters (Filter Cutoff, Drive, Delay Time) as a DAW-synced modulation source.

---

