# PhysicalQualityRelay

PhysicalQualityRelay combines the audited QualityRelay 17 SAFE-MESH research package with a buildable, manually advanced physical inspection demonstration.

The recommended demonstration is:

`LOAD → CALIPER + WEBCAM A → VISION + WEBCAM B → DECISION → HUMAN CONFIRMATION → SERVO DIVERTER → PASS / HOLD / REJECT`

Start here:

- [Project overview and software setup](README_START_HERE.md)
- [Exact physical build, wiring, equipment, printing, and test guide](docs/engineering/MANUAL_TRACK_SERVO_SORTER_BUILD_GUIDE.md)
- [Kiro Web handoff](KIRO_START_PROMPT.md)
- [Physical demo requirements](.kiro/specs/manual-track-demo/requirements.md)
- [ESP32 manual-sorter firmware](firmware/manual_sorter/esp32_manual_sorter.ino)
- [Printable test specimens](examples/cad/stl/)

The transport is manual. One ESP32 is used only for the final servo diverter, so a powered conveyor, motor driver, and multiple ESP32 boards are not required for the first demonstration. This is a research prototype, not an industrial safety system or certified inspection product.
