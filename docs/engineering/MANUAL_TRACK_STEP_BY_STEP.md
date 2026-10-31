# Manual-track demo: step-by-step connection and run guide

This guide turns [MANUAL_TRACK_SERVO_SORTER_BUILD_GUIDE.md](MANUAL_TRACK_SERVO_SORTER_BUILD_GUIDE.md) into an ordered checklist: what to connect to what, in which order, how to test each part, and what to write down during a run.

Open the interactive version before you build: [`web/manual_track_visualizer.html`](../../web/manual_track_visualizer.html). It opens directly in a browser (double-click the file; no server, camera, or ESP32 needed). It has three tabs:

1. **Virtual line & guided run**: top view of the 800 × 250 mm board, the carrier moving between stations, a decision for Runs A–C plus two failure cases, the servo flap swinging into the three bins, and an ESP32 emulator that copies the logic and messages of `firmware/manual_sorter/esp32_manual_sorter.ino` (15 s expiry, edge-triggered button, held button does not repeat).
2. **Wiring, step by step**: the same 15 connection steps as section 3 below, one wire at a time, plus an optional overlay of forbidden connections.
3. **What is real today**: which parts of the flow exist in the repository and which are still planned.

The visualizer is a training aid. It produces no evidence, and its decision thresholds are placeholders.

For the physical build itself, open [`web/physical_build_guide.html`](../../web/physical_build_guide.html). It shows the breadboard hole by hole with each jumper's type, a 3D-style view of the whole bench with every cable (manual track, gravity chute, or motorized belt for comparison), a side view of the exit and swivel chute, a recordable bench test (36 tests, CSV export), a parts list, and what to 3D print.

## Scope

- The carrier is pushed by hand between stops. There is no powered conveyor, motor, or motor driver.
- One ESP32 controls only the final servo diverter.
- The caliper is manual. Webcam A records its display. The number you confirm is the measurement.
- Webcam B gives uncalibrated visual evidence. It does not measure millimetres.
- A physical button press is the only thing that moves the servo.

What the repository supports today: the specimens, the servo firmware, and this guide. You stage routes by typing `P`, `H`, or `R` in the Arduino Serial Monitor and record evidence in [`templates/manual_track_run_log.csv`](templates/manual_track_run_log.csv). The `/demo/` session API, two-webcam capture page, and serial bridge described in `.kiro/specs/manual-track-demo/` are **not implemented yet**.

---

## 0. What to 3D print

Required: the 9 specimens in `examples/cad/stl/` (list in section 1). Print them flat in white/light PLA, one batch, same settings: 0.2 mm layers, 3–4 walls, 20–40% infill, no supports, no scaling, no brim (or remove it fully). Measure widths at mid-height. Record printer, nozzle, material, and layer height.

Optional: carrier, track sections, caliper blocks, swivel chute, servo mount. The repository has no CAD for these; cardboard, foam board, or wood work fine. Sizes are in `web/physical_build_guide.html`, tab 5.

## 1. Prepare specimens (before any wiring)

1. Print from `examples/cad/stl/`: `slot_good_21p20` ×1, `slot_nominal_21p00` ×1, `slot_risk_20p70` ×2, `tab_good_19p80` ×1, `tab_nominal_20p00` ×1, `tab_risk_20p30` ×2, `repeatability_bar_20p00` ×1.
2. Label each part with a black marker using the IDs from `examples/cad/print_manifest.csv`: `S-GOOD`, `S-NOM`, `S-RISK`, `T-GOOD`, `T-NOM`, `T-RISK`, `RPT-20`. Add `-2` to the backup risk parts.
3. Pick the three live specimens:
   - Run A: `S-GOOD` with pair `T-GOOD` (CAD clearance 1.40 mm).
   - Run B: `S-RISK` with pair `T-RISK` (CAD clearance 0.40 mm).
   - Run C: `T-NOM` with a clearly disclosed visual-anomaly marker (for example a small black sticker). Label it `T-NOM-ANOM`. Call it an anomaly-detection fixture, not a real defect.
4. Repeatability screen: measure `RPT-20` ten times, re-seating it each time. Write down the spread (max − min). Use this as your uncertainty value.

## 2. Build the mechanical line

Mark these positions on the baseboard (distance from the left edge):

| Position | Station | Physical feature |
|---:|---|---|
| 80 mm | LOAD | Carrier start stop |
| 250 mm | CALIPER | Hard stop + two registration blocks for the specimen; Webcam A clamp beside it |
| 450 mm | VISION | Hard stop + dark matte pad; Webcam B overhead; diffuse light from both sides |
| 600 mm | DECISION | Laptop screen visible from here |
| 700 mm | DIVERTER | Hard stop just before the servo flap at the track exit |
| after 800 mm | BINS | PASS left, HOLD center, REJECT right (looking along the direction of travel) |

1. Carrier: 75 × 45 mm flat plate, 3–5 mm lip on three sides, front open, orientation arrow, carrier ID.
2. Rails: place both rails around the carrier with 2–3 mm total clearance. Push the carrier the full length. Fix the rails only once it slides without jamming, rotating, or lifting.
3. Add a raised stop (or a clear line) at each station.
4. Caliper station: glue two small blocks so the specimen is always measured in the same place.
5. Vision station: fix the dark pad. Put the carrier at the stop and mark its outline so every capture has the same pose.
6. Diverter: mount the servo beside or under the track exit. **Do not attach the flap yet.**

## 3. Wiring: connect in this order

Everything stays unplugged until step 12. The step numbers match the visualizer's wiring tab.

Jumper types for the recommended layout (ESP32 pressed into the breadboard in rows b and i, columns 3–17, USB to the left):

| ID | Jumper | From | To |
|---|---|---|---|
| W7 | M–M red | DC jack adapter + terminal | top + rail, col 52 |
| W8 | M–M black | DC jack adapter − terminal | top − rail, col 53 |
| C1 | capacitor legs | long leg: top + rail col 47 | stripe leg: top − rail col 47 |
| W1 | M–M black | a4 (ESP32 GND) | top − rail, col 36 |
| W4 | M–M brown | servo plug brown socket | top − rail, col 44 |
| W3 | M–M red | servo plug red socket | top + rail, col 42 |
| W2 | M–M orange | servo plug orange socket | a5 (ESP32 D13) |
| SW1 | button legs | e22, e24 | f22, f24 |
| W5 | M–M green | a12 (ESP32 D32) | j22 |
| W6 | M–M black | a24 | top − rail, col 38 |

That is 8 male–male jumpers. If the ESP32 does not fit on the breadboard with row a free, leave it off the board and use male–female jumpers for W1, W2, and W5 (socket end on the ESP32 pin). Female–female jumpers are not needed. Column numbers assume a 30-pin DEVKIT V1; follow the printed pin labels on your board.

| Step | From | To | Why | Check before moving on |
|---:|---|---|---|---|
| 0 | — | — | Unplug USB and the servo supply. Remove the flap. | Supply OFF, USB out. |
| 1 | ESP32 | Breadboard, across the center gap | Each pin gets its own row | No pins shorted across the gap |
| 2 | Servo supply **+** | Breadboard **+** rail | Servo power comes only from the external supply | Multimeter: voltage matches the servo label (5 V for SG90/MG90). Supply rated ≥ 1 A, 2 A preferred |
| 3 | Servo supply **−** | Breadboard **−** rail | Servo return | Polarity correct |
| 4 | Capacitor 470–1000 µF | Long leg to **+** rail, stripe leg to **−** rail, near the servo | Reduces voltage dips | Stripe on **−**. A reversed electrolytic capacitor can fail violently |
| 5 | **−** rail | ESP32 **GND** | Common signal reference. Only ground is shared | Continuity between supply − and ESP32 GND |
| 6 | Servo brown/black | **−** rail | Servo ground | Check your servo's colour code |
| 7 | Servo red | **+** rail | Servo power | Red goes to the external supply only, never to ESP32 3V3, 5V/VIN, or a GPIO |
| 8 | Servo orange/yellow | ESP32 **GPIO13** | PWM signal (`SERVO_PIN = 13`) | Your board exposes GPIO13. If not, pick another free output pin and change `SERVO_PIN` |
| 9 | Button leg 1 | ESP32 **GPIO32** | Confirmation input (`INPUT_PULLUP`) | On a 4-leg tactile button use two diagonal legs |
| 10 | Button leg 2 | ESP32 **GND** | Pressed = LOW | No resistor needed |
| 11 | — | — | Inspect | No wire from servo red to any ESP32 pin. Multimeter: + rail to − rail not shorted |
| 12 | ESP32 USB | Powered USB hub → laptop | Upload, serial at 115200, ESP32 power | Use a data cable. A new serial port appears |
| 13 | Webcam A, Webcam B | Separate hub ports | Three data devices in total | Both cameras open at the same time |
| 14 | — | — | Power-up order: upload sketch, open Serial Monitor, then switch servo supply ON | Servo holds 90° (HOLD) |

Board notes:

- GPIO13 and GPIO32 are fine on a classic ESP32 DevKit (ESP32-WROOM-32). On an **ESP32-S3**, GPIO26–32 are usually wired to flash/PSRAM. Do not use GPIO32 there; choose another free pin and change `CONFIRM_BUTTON_PIN`.
- Avoid strapping pins (GPIO0, 2, 5, 12, 15 on classic ESP32) for the button.
- Never connect a DC motor directly to an ESP32 pin. A conveyor is out of scope.

## 4. Upload the firmware and bench-test with the flap OFF

1. Arduino IDE → Boards Manager → install **esp32 by Espressif Systems**.
2. Library Manager → install **ESP32Servo**.
3. Open `firmware/manual_sorter/esp32_manual_sorter.ino`, select your exact board and port, upload.
4. Serial Monitor: **115200 baud**. Line ending can be anything; the sketch ignores characters other than P/H/R.
5. Expect:
   ```
   QualityRelay sorter ready.
   Type P, H, or R, then press the physical confirmation button.
   ```
6. Switch the servo supply ON. The horn should sit at 90°.

Run these tests and write the result in section 9:

| Test | Do | Expected Serial Monitor output | Expected servo |
|---|---|---|---|
| Stage only | Send `P` | `Route staged: P. Press confirmation button within 15 seconds.` | Does **not** move |
| Confirm | Press the button once | `AUTHORIZED MOVE -> route P, angle 35` then `Route consumed; stage the next specimen separately.` | Moves to 35° once |
| Held button | Send `H`, press and **hold** the button for 3 s | One `AUTHORIZED MOVE -> route H, angle 90` | Moves once, no repeat |
| No stage | Press the button with nothing staged | `No route staged; gate did not move.` | Does not move |
| Timeout | Send `R`, wait 16 s, then press | `Staged route expired; gate did not move.` then `No route staged; gate did not move.` | Does not move |
| Reset | Press EN/RESET | Ready banner again | Returns to 90° |

## 5. Calibrate the gate angles

1. Power OFF the servo supply. Attach the flap to the horn while the servo is at 90°, pointing straight along the track (HOLD = center bin).
2. Power ON. Stage `P` + button, then `R` + button. Watch for binding.
3. If the flap touches a stop, the track, or the carrier, power OFF and change `PASS_ANGLE` / `REJECT_ANGLE` in 5° steps. Keep every angle inside 0–180°, and never drive the flap against a hard stop.
4. If PASS swings toward the right bin, swap `PASS_ANGLE` and `REJECT_ANGLE`.
5. Re-upload and repeat until a part released at the diverter stop lands in the intended bin at each angle. Write the final angles in the log.

## 6. Set up the two webcams

1. Name them in the log: `CALIPER-CAM-A` and `VISION-CAM-B`. Note which physical USB port each one uses.
2. Webcam A: clamp it so the caliper display fills most of the frame. Tilt slightly to remove glare.
3. Webcam B: fix it overhead (or at a fixed oblique angle) above the vision pad. Lock resolution, focus, and exposure if your camera software allows it.
4. Open both cameras at the same time (for example with the operating system camera app plus a second app, or OBS with two sources). If one disappears, move it to another hub port.
5. Capture a reference image of `T-NOM` and of `S-NOM` at the vision stop. Save them as the vision references.
6. Name every saved image `<specimen>_<camera>_<YYYYMMDD-HHMMSS>_<n>.jpg`. Keep raw images out of Git (`captures/` and `data/` are already ignored).

## 7. Run the demonstration (one log row per specimen)

Before every run: the gate is at HOLD (90°), both webcam views are locked, the Serial Monitor is open, and the servo supply is ON.

| Step | Station | You do | You record |
|---:|---|---|---|
| 1 | LOAD (80 mm) | Put the labeled specimen in the carrier, arrow forward | run ID, specimen ID, pair ID, carrier ID |
| 2 | CALIPER (250 mm) | Push to the stop. Seat the specimen in the blocks. Measure 3 times. Hold the caliper so Webcam A sees the display and capture one still. Return the specimen in the same orientation | 3 readings, instrument ID, calibration ID, Webcam A file name, the value you confirm |
| 3 | VISION (450 mm) | Push to the stop. Do not touch the specimen. Capture one still from Webcam B | Webcam B file name, comparison with the reference (clear / anomaly candidate / unusable) |
| 4 | Fit (pair runs only) | Try the predefined fit (for example: tab enters the slot by hand without force) | Criterion and outcome PASS / FAIL / INDETERMINATE |
| 5 | DECISION (600 mm) | Apply the rules below | Decision and HOLD reasons |
| 6 | DECISION | Type the route letter in the Serial Monitor. Do not press the button yet | Letter sent |
| 7 | DIVERTER (700 mm) | Push to the stop. Check the Serial Monitor shows the expected route. Press the button once within 15 s | Serial line after the button |
| 8 | Bins | Slide the specimen off the carrier front past the flap | **Observed** bin, written separately from the requested route |
| 9 | Reset | Send `H` and press the button so the gate returns to 90°. Pull the carrier back to LOAD | Gate returned to HOLD: Y/N |

Decision rules (conservative, applied by hand today):

- **HOLD** if anything is missing, blurred, inconsistent, or uncertain: fewer than 3 readings, a large repeatability spread, an OCR candidate that differs from your confirmed number, a missing or unusable Webcam B image, the same image used for both cameras, a fit result that is FAIL or INDETERMINATE or absent.
- **PASS** only when all evidence is present and consistent, the fit is PASS, and a named person authorizes release.
- **REJECT** only when a person has confirmed the nonconformance by looking at the part. A camera result alone means HOLD.

Expected outcomes:

| Run | Specimen | Expected route | Why |
|---|---|---|---|
| A | `S-GOOD` + `T-GOOD` | PASS (after human authorization) | Consistent readings, clear image, fit PASS |
| B | `S-RISK` + `T-RISK` | HOLD | Dimensions plausible, but fit is tight/INDETERMINATE |
| C | `T-NOM-ANOM` | REJECT if a person confirms the anomaly, otherwise HOLD | Visual candidate alone cannot cause REJECT |

## 8. Failure demonstrations (after Runs A–C work)

| Demo | How | Expected |
|---|---|---|
| Webcam B missing | Unplug Webcam B before step 3 | HOLD, recapture requested |
| Blurred/covered image | Cover Webcam B | HOLD |
| Same image twice | Use the Webcam A file as the Webcam B evidence | Not independent → HOLD |
| OCR mismatch | Note an OCR/display value that differs from your reading | HOLD; re-read and confirm again |
| Timeout | Stage a letter and wait more than 15 s | `Staged route expired; gate did not move.` Stage again deliberately |
| Wrong bin | Stage PASS, then put the part in HOLD by hand | Log shows requested ≠ observed. The sensor-based ACK is not implemented yet |

## 9. Known firmware behaviors to watch

The visualizer shows each of these.

1. **The gate stays where it last moved.** After a PASS move, the flap keeps pointing at PASS. If the next specimen's staging fails or expires and you advance the part anyway, it lands in PASS. Always do step 9 (send `H` + button) before loading the next part.
2. **A new letter replaces a staged one.** If `P` is staged and anyone sends `R`, the sketch silently switches to `R` and restarts the 15 s timer. Send only one letter per specimen.
3. **Opening the serial port can reset the ESP32.** Many boards reboot when a program opens the port, which clears any staged route and moves the gate to HOLD. Wait for the ready banner before sending a letter.
4. **"AUTHORIZED MOVE" is a command report, not a position measurement.** A hobby servo has no feedback. Only your observation (or a future arrival sensor) confirms where the part went.
5. Only one program can hold the serial port. Close the Arduino Serial Monitor before running any other tool that uses the port.

## 10. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| ESP32 reboots when the servo moves | Servo powered from the ESP32, or supply too weak | Servo on external supply; add the capacitor; 2 A supply |
| Servo jitters at rest | Missing common ground or noisy supply | Check step 5 wiring; shorten servo wires |
| Servo does not move at all | Signal on the wrong pin, supply off, or ESP32Servo missing | Verify GPIO13 and the supply; reinstall ESP32Servo |
| Button never registers | Wrong pin or both wires on the same side of a tactile button | Use diagonal legs; verify GPIO32 exists on your board |
| No serial port on the laptop | Charge-only USB cable or missing USB driver (CP210x/CH340) | Use a data cable; install the driver |
| One webcam disappears | Bus power or bandwidth | Separate hub ports; powered hub; lower resolution |
| Two webcams have the same name | Identical Logitech models | Identify them by USB port and by covering one lens |

## 11. Local hardware checklist (fill in yourself)

These checks can only be done on your bench. Leave them blank until you have run them. Kiro Web does not mark them passed.

| ID | Check | Result (PASS / FAIL / NOT RUN) | Date | Notes |
|---|---|---|---|---|
| H1 | Both Logitech webcams enumerate and stay stable at the same time | | | |
| H2 | Webcam A captures the caliper display clearly without glare | | | |
| H3 | Webcam B reproduces specimen pose and lighting | | | |
| H4 | Firmware compiles for the exact board and `ESP32Servo` version (record versions) | | | |
| H5 | Servo uses external regulated power with common signal ground | | | |
| H6 | P/H/R does not move the servo before the button | | | |
| H7 | Button moves the gate once; held button does not repeat | | | |
| H8 | Timeout prevents movement | | | |
| H9 | Calibrated angles route to PASS/HOLD/REJECT without binding (record angles) | | | |
| H10 | Good, uncertain, and anomaly runs complete with saved evidence | | | |
| H11 | Optional arrival sensor correlates the actual bin, or is explicitly disabled | | | |
