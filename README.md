# Michiyuki

**Michiyuki** is an open source preservation, restoration, and reverse engineering project for the **Tomy Omnibot 17μ i-SOBOT** — the world's smallest mass-produced humanoid robot at the time of its release in 2007, and one of the most sophisticated consumer robot toys ever manufactured.

The i-SOBOT is no longer manufactured. Tomy never released a PC SDK, service documentation, or spare parts program. The custom micro-servos that animate its 17 degrees of freedom are proprietary, wear out, and have no off-the-shelf replacement. Michiyuki exists to change that.

This README intentionally avoids implementation specifics. Hardware findings, firmware reverse engineering, and parts documentation are recorded in their respective directories.

---

## Project Vision

The goal of Michiyuki is to make the i-SOBOT **indefinitely serviceable** — not merely repairable with donor parts from other failing units, but supported by a complete, open library of documented internals, reproducible components, and understood protocols.

This means:

- A complete physical description and parts catalogue of the original hardware
- Full documentation of the IR remote control protocol and internal servo bus
- Replacement servo designs compatible with the original mounting geometry
- A parts archive sufficient to restore a non-functioning unit to operation
- An extension path that preserves original behavior as a baseline while enabling new capabilities
- Replacement PCB designs that restore full functionality where original boards are damaged or unrepairable

---

## Design Philosophy

Michiyuki is guided by the following principles:

- **Documentation before modification** — no hardware is altered before its original state is recorded
- **Understanding before replacement** — components are characterized before substitutes are sought
- **Original behavior preserved as a baseline, not a ceiling** — the stock i-SOBOT experience is the reference point, not the limit
- **Open by default** — all findings, measurements, and designs are published under open licenses so the community can build on them without restriction
- **Buildability** — replacement parts and processes should be executable by a motivated individual with hobbyist tools, not specialist manufacturing equipment

---

## Repository Structure
```
michiyuki/ 
├── CONTRIBUTING.md          # Contribution guidelines 
├── LICENSE-DOCUMENTATION    # CC BY 4.0 (documentation) 
├── LICENSE-FIRMWARE         # GPL-3.0 (firmware and software) 
├── LICENSE-HARDWARE         # CERN-OHL-S-2.0 (hardware designs) 
├── README.md                # This file 
├── docs/                    # Physical descriptions, teardown notes, project documentation 
├── hardware/                # Servo designs, PCB documentation, schematics 
├── firmware/                # IR protocol documentation, servo bus protocol, microcontroller work 
├── remote/                  # Remote control documentation and protocol decoding 
├── parts/                   # Replacement parts sourcing, 3D models, fabrication guides 
└── references/              # Datasheets, teardown photographs, preserved community findings
```

---

## Current Status

The project is in active early development. Two physical specimens are in hand:

- **Robot 1** — complete with remote, fully functional; retained as a reference unit
- **Robot 2** — without remote, already disassembled; primary dissection subject

Work completed to date:

- Initial disassembly of Robot 2

Active priorities:

- Physical description documentation and component cataloguing
- Internal photography and PCB characterization
- Servo bus and IR protocol documentation

---

## Contributing

The project currently needs people with experience in:

- Electronics reverse engineering and PCB documentation
- Micro-servo design and miniature mechanical engineering
- IR protocol capture and analysis
- Embedded systems and microcontroller development (Arduino, RP2040, or similar)
- 3D scanning and CAD modelling for parts reproduction
- Precision mechanical measurement and dimensional drawing
- PCB design (KiCad or equivalent)

If you own an i-SOBOT and are willing to contribute photographs, measurements, or disassembly notes, that is equally valuable — particularly from units with different regional variants or failure modes.

Please open an issue or discussion before proposing major architectural changes so that design intent can be preserved.

---

## Prior Art and Related Work

The i-SOBOT community kept these robots alive for nearly two decades on scattered forum threads and personal blogs. Work this project builds on:

- [Robot Watch teardown by Takayoshi Ishii (2007)](https://robot.watch.impress.co.jp/cda/column/2007/11/08/731.html) — the canonical internal teardown; PCB photography, IC identification, and servo bus analysis
- [Michal Krompiec's IR protocol decode (2009)](http://minkbot.blogspot.com/2009/08/isobot-infrared-remote-protocol-hack.html) — full bit-field structure, checksum algorithm, and hidden command discovery
- [OTL/i-sobot Arduino sketch](https://github.com/OTL/i-sobot) — Arduino implementation of the IR command library
- [i-sobothacking blog](http://i-sobothacking.blogspot.com/) — direct servo bus tapping and headless control experiments
- [Arduino Forum: i-Sobot hacked or Pro Mini shield](https://forum.arduino.cc/t/i-sobot-hacked-or-pro-mini-shield/8836) — Pro Mini backpack design and IR command library
- [RoboSavvy i-SOBOT forum threads](https://forum.robosavvy.com/) — community failure mode documentation and repair discussion

---

## License

Michiyuki uses a split license model:

- **Hardware designs** are licensed under the **CERN Open Hardware License Version 2 – Strongly Reciprocal (CERN-OHL-S-2.0)** (see `LICENSE-HARDWARE`)
- **Firmware and software** are licensed under the **GNU General Public License v3.0 (GPL-3.0)** (see `LICENSE-FIRMWARE`)
- **Documentation** is licensed under **Creative Commons Attribution 4.0 International (CC BY 4.0)** (see `LICENSE-DOCUMENTATION`)

Unless otherwise noted, files are licensed according to the category they fall under.

---

Michiyuki is not affiliated with or endorsed by Tomy or Takara Tomy. "Tomy," "Takara Tomy," "Omnibot," and "i-SOBOT" are trademarks of Takara Tomy Co., Ltd.
