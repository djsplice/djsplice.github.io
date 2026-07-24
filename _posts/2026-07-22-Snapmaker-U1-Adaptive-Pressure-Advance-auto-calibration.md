---
title: Snapmaker U1 Adaptive Pressure Advance auto-calibration
date: 2026-07-22
author: Jeff Barrows
tags: [3d-printing, snapmaker, orcaslicer, klipper, pressure-advance]
excerpt: "Use the U1 flow coil + macros to build an Orca Adaptive PA table in ~25 minutes of extrusion — no magnifying glass required."
article_header:
  type: overlay
  background_image:
    gradient: radial-gradient(circle, rgba(238,174,202,1) 0%, rgba(148,187,233,1) 100%);
layout: article
key: 2026-07-22-u1-adaptive-pa-autocal
embed:
  send: false
  remove: true
share: true
---

*Use the U1 flow coil + macros to build an Orca Adaptive PA table in ~25 minutes of extrusion — no magnifying glass required.*

<div>{%- include extensions/youtube.html id='TzlhBpQkZEI' -%}</div>

**Video walkthrough:** [Snapmaker U1 Adaptive PA auto-cal on YouTube](https://www.youtube.com/watch?v=TzlhBpQkZEI)

<!--more-->

The other day I started wondering about what specific types of sensors were on my Snapmaker U1, how they were being used, and if I was missing any settings so that I could take full advantage of them. There's nothing worse than paying for something you're not using!

The Snapmaker U1 website doesn't have a lot of deep technical specs, however it does claim a few things: mainly **Smooth Print Starts with Smart Calibration**. Agreed. But how?

1. **Toolhead Offset Calibration** — inductance coil Z probing (`probe_inductance_coil`) driven by `xyz_offset_calibration` macros + Python
2. **Vibration Compensation** — `LIS2DW` accelerometer → input shaper / resonance testing
3. **Fine-tuned Extrusion using Pressure Advance** — same inductance coil + Snapmaker `[flow_calibrator]` and `FLOW_MEASURE_K` (custom Klipper extension in `klippy/extras/flow_calibrator.py`, not mainline Klipper)
4. **Automatic mesh bed leveling** — inductance coil as the probe
5. **Filament runout sensor** — `filament_motion_sensor` (encoder-style; often optical/IR + wheel) — detects motion pulses while extruding

## Pressure Advance

OK, lots of sensors and capabilities, but one in particular caught my eye - **Pressure Advance**.

For me, running calibration prints to determine Pressure advance has been a huge pain, and even after several prints, examining print lines with a magnifying glass, etc... I've never felt confident in my ability to accurately/consistently judge the correct value, plus it's time consuming!

![Printed pressure advance line test — several K values look similarly good](/assets/images/apa-pa-line-test.png)
<div align="center"><sup>*0.015 looks good, but 0.020 does too — 0.00 looks ok too!*</sup></div>

While I _want_ to calibrate PA (_pressure advance_) for all of my filaments, I just can't bring myself to do it - instead I go YOLO and slap a generic 0.018 to 0.020 on it and call it a day, or more recently just use whatever's set in the manufacturers provided Orca profile setting.

### Pressure Advance auto-calibration to the rescue!

So how does Pressure Advance calibration work on my U1, and am I using it correctly? As it turns out, the answer is no, I'm not using this feature at all!

This is **printer-side Dynamic Flow Calibration** (a single PA per tool from the coil residual). It is **not** the same as Orca's **Adaptive Pressure Advance** table later in this post.

Here's my understanding of how it works:

1. Within OrcaSlicer, you must disable the `Enable Pressure Advance` setting in your filament profile. If it's set, the printer will use the setting found in the sliced gcode; if it's not set, it will try and use values stored in `printer_data/config/snapmaker/flow_calibrator.json`.

![OrcaSlicer filament setting — Enable Pressure Advance](/assets/images/apa-orca-enable-pa.png){: width="400"}

2. You have to slice the model and 'Upload' it to the printer (not 'Upload and Print')
3. You have to start the print job from the printer and select **Enable Dynamic Flow Calibration** from the printers screen.
4. This has the printer perform a flow calibration routine before the actual print starts
   1. Once completed the routine saves the Pressure Advance value (*per toolhead*) on the printer in `printer_data/config/snapmaker/flow_calibrator.json`. Default values are 0.02; you can see that I ran PA calibration on `extruder` and `extruder3`:

```json
{
  "factor": {
    "extruder": 0.023725,
    "extruder1": 0.02,
    "extruder2": 0.02,
    "extruder3": 0.025014
  }
}
```

I haven't fully traced reload behavior; treat these as per-tool stored defaults until you lock PA into an Orca filament profile.

If you want to use this calibration value forever, you need to grab it from the console output, or from `flow_calibrator.json`, re-enable `Enable Pressure Advance` in OrcaSlicer, save the value AND save the settings as a Custom Profile.

## Adaptive Pressure Advance - Auto-Calibration

Adaptive Pressure Advance is a beta feature in Orca Slicer, and is designed to dynamically adjust nozzle pressure correction values based on changing print speeds and accelerations. As an example, if you print infill at a higher speed and acceleration it will benefit from a different PA setting vs when printing lower speed/accels for outer walls. Being able to adjust PA value for variable print speeds/accelerations during print time should help improve overall quality of print features.

So now that I figured out how to enable Pressure Advance calibration, I started wondering if I could use the probe to help generate an Adaptive Pressure Advance profile...

If you think generating a single PA test and accurately evaluating it is tedious, try building an [Adaptive Pressure Advance profile](https://github.com/OrcaSlicer/OrcaSlicer/wiki/adaptive_pressure_advance_calib)...  The OrcaSlicer documentation suggests that you need to [print 12 pressure advance tests](https://github.com/OrcaSlicer/OrcaSlicer/wiki/adaptive_pressure_advance_calib#how-to-calibrate-the-adaptive-pressure-advance-model) in order to generate a profile (4 tests for each Acceleration value).

This is a bonafide nightmare. I did it once - it took me hours to print the tests and do my best to judge the correct PA value...

### Simplification of test data

I got to thinking that there must be a way to reduce the number of test prints and do some sort of math/interpolation to estimate interim values, so I started to experiment... I was on to something. Instead of printing 4 tests per Acceleration setting, I found that you could easily weed out a few test runs because they wouldn't provide any meaningful data:

- **Low Accel, Low Speed**: 1k accel, 50 mm/sec doesn't require any (or very minimal) pressure advance adjustments as nozzle pressure remains generally constant
- **Low Accel, High Speed**: 1k accel 300mm/sec - the Ellis PA test model won't show accurate results. Why? The toolhead will never reach 300mm/sec for each leg of the V shaped line!
  - If each leg is about 30mm in length - the max speed it will reach is about 175mm/sec
  - `minimum cruise ratio` (typically set to 50%) may also limit acceleration, working against accurate accel/speed measurements!

Cutting test count is good, but why not be smarter about which tests we run? I came across a YouTube video on optimizing top surface flow without ironing ([watch](https://www.youtube.com/watch?v=mAxZt6s6R0E&t=326s)) that used a sparse experimental layout in the spirit of George Box’s work: instead of testing every combination of settings, you pick practical low/high values for each factor and measure a small set of corner points (plus a midpoint) so you still see main effects and interactions.

Adaptive PA is a two-factor problem — volumetric flow and acceleration — so the same idea applies. I used a **Box-style multi-point sample**: four envelope corners (low/high flow × low/high accel) plus a center print condition. It’s a deliberate sparse design so a few sensor runs cover the useful printing envelope. This is **not** a full quadratic response-surface fit; Orca interpolates the table rows while printing.

These are the **default** 5 test points in the AutoCal macros (based on Snapmaker '0.2 Standard'-ish speeds/accels; tune them to match your typical print profile):

| Test Point   | XY Speed | XY Accel    | Volumetric Flow (Q) | Role                                                   |
| ------------ | -------- | ----------- | ------------------- | ------------------------------------------------------ |
| `low_anchor` | 100 mm/s | 2000 mm/s²  | ~8.1 mm³/s          | Low flow at HQ outer wall acceleration - gentle motion |
| `high_flow`  | 250 mm/s | 2000 mm/s²  | ~20.4 mm³/s         | High flow at soft acceleration                         |
| `high_force` | 100 mm/s | 10000 mm/s² | ~8.1 mm³/s          | Low flow at high inner wall / infill acceleration      |
| `stress`     | 250 mm/s | 10000 mm/s² | ~20.4 mm³/s         | Combined high flow + high acceleration                 |
| `center`     | 200 mm/s | 6000 mm/s²  | ~16.3 mm³/s         | Mid-point print condition                              |

Volumetric flow assumes the macro default line geometry: 0.4 nozzle / 0.2 layer / 0.45 width.

### Running the U1 Adaptive PA AutoCal macro

Follow the install guide in the GitHub repo [README](https://github.com/djsplice/u1-adaptive-pa-autocal/blob/main/README.md), then run the full test suite for the desired toolhead/filament. Full walkthrough: [USER_GUIDE.md](https://github.com/djsplice/u1-adaptive-pa-autocal/blob/main/docs/USER_GUIDE.md).

Each test point is on the order of **~4–5 minutes** of heat + extrusion (depends on temp and K range), so expect a full 5-point suite around **~20–25 minutes** and roughly **15–20 g** of filament. Empty the purge bin first.

As an example: `APA_COIL_RUN_ALL EXTRUDER=1 TEMP=220` will run 5 full PA calibration tests using the default speed and acceleration parameters. You should see output similar to this in your Fluidd console:

```bash
22:52:11  // ========== APA TEST POINT START ==========
22:52:11  // Point: center | Tool T2 (extruder2) | MODE=MEASURE TEMP=220
22:52:11  // XY: speed=200.0 mm/s accel=6000.0 -> Q=16.2832 mm3/s
22:52:11  // After k/area lines, find the sign flip, then either:
22:52:11  // APA_FINISH_LAST K0=... A0=... K1=... A1=... MAX_ABS=...
22:52:11  // (FLOW=16.2832 ACCEL=6000.0 NAME=center filled automatically)
22:52:11  // ==========================================
22:52:11  // start flow calibration for extruder2
22:52:11  // start heating heater to 220 degre
22:52:37  // start measuring frequency
22:52:37  // start extruding
22:52:37  // k min: 0.005, max: 0.040, step: 0.005
22:53:08  // k0.005: area: 16395
22:53:49  // k0.010: area: 1278
22:54:30  // k0.015: area: -6811
22:55:11  // k0.020: area: -19159
22:55:49  // k0.025: area: -27580
22:56:27  // k0.030: area: -37033
22:57:06  // k0.035: area: -44343
22:57:34  // ========== APA TEST POINT MEASURE DONE ==========
22:57:34  // Point center finished. Find sign-flip in k/area lines ABOVE, then:
22:57:34  // APA_FINISH_LAST K0= A0= K1= A1= MAX_ABS=
22:57:34  // If you already started the next point, use full form instead:
22:57:34  // APA_FINISH_CELL K0= A0= K1= A1= FLOW=16.2832 ACCEL=6000.0 NAME=center MAX_ABS=
22:57:34  // Orca row shape: PA, 16.2832, 6000.0
22:57:34  // =================================================
```

Once the test completes, you need to run the `APA_FINISH_CELL` macro to generate the pressure advance entry for the Adaptive Pressure Advance table in Orca. You will find the command in the `APA TEST POINT MEASURE DONE` section and it will look something like:

`APA_FINISH_CELL K0= A0= K1= A1= FLOW=16.2832 ACCEL=6000.0 NAME=center MAX_ABS=`

You will need to find the `K0 A0 K1 A1` and `MAX_ABS` values in the test output. Simply look for the pairs of values that transition from a positive area value to a negative area value:

```bash
22:53:49  // k0.010: area: 1278   <- K0 and A0
22:54:30  // k0.015: area: -6811  <- K1 and A1
```

And the `MAX_ABS` is the largest absolute area value in this example (use the magnitude as a positive integer):

```bash
22:57:06  // k0.035: area: -44343  → MAX_ABS=44343
```

The complete `APA_FINISH_CELL` command for this test round is:

```text
APA_FINISH_CELL K0=0.010 A0=1278 K1=0.015 A1=-6811 FLOW=16.2832 ACCEL=6000.0 NAME=center MAX_ABS=44343
```

NOTE: Even though the largest area value was negative, you specify `MAX_ABS` as a normal positive integer.

Full macro output:

```bash
23:10:39  $ APA_FINISH_CELL K0=0.010 A0=1278 K1=0.015 A1=-6811 FLOW=16.2832 ACCEL=6000.0 NAME=center MAX_ABS=44343
23:10:39  // ========== APA FINISH: center ==========
23:10:39  // Bracket: k=0.01 area=1278.0 -> k=0.015 area=-6811.0
23:10:39  // K* (zero cross): 0.0107899616763506 -> rounded 0.011
23:10:39  // Confidence: 76/100 (good) | strong signal | bracket_balance=0.3159846705402398
23:10:39  // Tip: pass MAX_ABS=largest abs area in the sweep for better signal grading
23:10:39  // --- copy/paste into Orca Adaptive PA ---
23:10:39  // 0.011, 16.2832, 6000.0
23:10:39  // ----------------------------------------
23:10:40  // Fallback hint: after all points, use median of K* (not min/max alone)
23:10:40  // or center-point K* / stock FLOW_CALIBRATE as static PA
23:10:41  // ========================================
```

Simply copy/paste the Adaptive PA entry into Orca:

```text
0.011, 16.2832, 6000.0
```

(Format is `PA, flow_mm3_s, accel_mm_s2`.)

### Rinse and repeat: Finish all five points, then static PA

Repeat **sign-flip → `APA_FINISH_CELL` → paste row** for each of the five suite points (`low_anchor`, `high_flow`, `high_force`, `stress`, `center`). After the suite ends, use the template from each point’s `MEASURE DONE` banner (or the full `APA_FINISH_CELL` form with `FLOW` / `ACCEL` / `NAME` if you already started another cell).

Orca still wants a single static **Pressure advance** fallback when Adaptive PA is off or outside the table. Easy recipe: take the median of your five \(K^*\) values (or use the center-point \(K^*\) / a stock Dynamic Flow result).

Similarly for the **Adaptive PA for Overhangs** - you can start by halving the static **Pressure advance** fallback setting as a starting point. If your Static PA value is `0.016` set your Overhangs Adaptive PA value to something like `0.008` as a starting point.

BONUS: you can capture coil frequency waveforms while a test runs to your local computer for debugging, though it's not required for calibration. See [COIL_DATA_CAPTURE.md](https://github.com/djsplice/u1-adaptive-pa-autocal/blob/main/docs/COIL_DATA_CAPTURE.md).

## Links

- **Video:** [YouTube walkthrough](https://www.youtube.com/watch?v=TzlhBpQkZEI)
- **Repo:** [u1-adaptive-pa-autocal](https://github.com/djsplice/u1-adaptive-pa-autocal)
- **User guide:** [docs/USER_GUIDE.md](https://github.com/djsplice/u1-adaptive-pa-autocal/blob/main/docs/USER_GUIDE.md)
- **Methodology:** [docs/METHODOLOGY.md](https://github.com/djsplice/u1-adaptive-pa-autocal/blob/main/docs/METHODOLOGY.md)
- **Orca Adaptive PA wiki:** [adaptive_pressure_advance_calib](https://github.com/OrcaSlicer/OrcaSlicer/wiki/adaptive_pressure_advance_calib)

Community tooling — verify on your machine and filament. Not affiliated with Snapmaker or OrcaSlicer.
