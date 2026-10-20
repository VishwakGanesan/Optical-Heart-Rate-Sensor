# Optical Heart Rate Sensor

An analog photoplethysmography (PPG) front end that turns the tiny change in light passing through a fingertip into a clean digital pulse, one per heartbeat. It is built from discrete parts: an IR emitter/phototransistor pair, a 101× active band-pass amplifier, and a comparator with hysteresis that drives an LED in time with the pulse.

**Stack:** analog design · LM324 op-amps · LM339 comparator · Analog Discovery 2 (scope, FRA)
**Team:** Vishwak Ganesan, Andrea Guerra · ECE 20008, Purdue · Apr–May 2026

![Full schematic](docs/figures/full_schematic.png)

## Signal chain

```
IR204A LED ──finger──▶ PT204-6B ──▶ HPF 0.68 Hz ──▶ ×101 gain + LPF 3.39 Hz ──▶ LM339 + hysteresis ──▶ NMOS ──▶ LED
  (emitter)            (detector)   (removes DC)     (LM324, cardiac band)       (analog → digital)      (RFD3055)
```

| Stage | Design | Value |
|---|---|---|
| **IR emitter** | 5 V through 100 Ω ∥ 150 Ω = 60 Ω (two resistors so neither exceeds its 0.25 W rating) | I = (5 − 1.2) / 60 = **63.3 mA** (< 100 mA max), P = 0.24 W |
| **Detector** | PT204-6B phototransistor with a 10 kΩ potentiometer as emitter resistor, so the baseline sits well above V<sub>CE,sat</sub> | 14.4 mV p-p without a finger → **40.16 mV p-p** with one |
| **Target band** | 40–200 BPM ÷ 60 | 0.667–3.33 Hz |
| **High-pass** | 2.2 kΩ + 150 Ω, 100 µF | f<sub>c</sub> = 1 / 2πRC = **0.677 Hz** |
| **Low-pass** | 470 Ω, 100 µF | f<sub>c</sub> = **3.386 Hz** |
| **Gain** | non-inverting, R<sub>f</sub> = 100 kΩ, R<sub>1</sub> = 1 kΩ | 1 + R<sub>f</sub>/R<sub>1</sub> = **101 V/V** → ~3 V p-p |
| **Comparator** | LM339 with positive feedback, R<sub>1</sub> = 10 kΩ, R<sub>2</sub> = 1 MΩ, ±5 V | V<sub>hyst</sub> = (V<sub>DD</sub> − V<sub>SS</sub>)·R<sub>1</sub>/(R<sub>1</sub>+R<sub>2</sub>) = **99 mV** |
| **Comparator input RC** | 82 kΩ, 1 µF | f<sub>c</sub> = 1.94 Hz, mid-band |
| **Indicator** | RFD3055 NMOS low-side switch for the LED | — |

The hysteresis window was sized to sit above the noise floor but well below the beat amplitude, so slow baseline drift and small ripples can't make the comparator chatter.

## Results

| | |
|---|---|
| ![Filter + gain stage](docs/figures/bandpass_amp.png) | ![Comparator](docs/figures/comparator.png) |
| Band-pass gain stage (LM324) | Comparator with hysteresis and LED driver |
| ![Amplified PPG](docs/figures/filtered_signal.png) | ![Final output](docs/figures/final_output.png) |
| Amplified, filtered PPG signal (~3 V p-p) | Final digital output: **1.66 Hz (≈ 99.5 BPM), 9.78 V p-p** |

* The frequency-response analysis showed the passband sitting inside the 0.67–3.33 Hz target.
* The 40.16 mV p-p sensor signal came out as a 9.78 V p-p square wave at 1.66 Hz. That rate was checked against a manual pulse count and against function-generator stimulus over repeated runs.
* **Limitation:** there was no shielding from ambient light, so the output tracks the pulse but isn't a 1:1 clinical heart-rate reading.

Full write-up with procedure, calculations, and scope captures: [`docs/report.pdf`](docs/report.pdf).

## What I'd do next

* Add an optical shroud and modulate the IR LED (lock-in style) to reject ambient light.
* Feed the comparator output to a microcontroller timer to compute and display BPM.
