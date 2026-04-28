# Tomy i-SOBOT: Technical reference

The Tomy i-SOBOT is a 17-servo, 19-IC consumer humanoid robot released in Japan in October 2007 at ¥29,800 and in the United States shortly thereafter at approximately US$300. It earned a Guinness World Record as the smallest mass-produced humanoid robot at the time of release, won METI's Robot of the Year 2008, and appeared on TIME's "Best Inventions of 2007" list.[^1] [^2] [^3] Despite that pedigree, Tomy never released a PC SDK, service documentation, or spare parts program, and never publicly identified the main microcontroller, voice IC, or per-servo controller ICs. The IR remote protocol and internal servo bus were subsequently decoded by hobbyists between 2007 and 2012. This document consolidates manufacturer specifications, the canonical Robot Watch teardown, and the surviving hobbyist record into a single technical reference for the Michiyuki project.

---

## Physical construction and 17-DOF skeleton

The i-SOBOT stands **165 mm tall, 96 mm wide, 67 mm deep, and weighs 350 g**.[^4] Its molded plastic shell is built around five field-replaceable modules — torso plus four limbs — with metal output shafts riding in metal bearings. Tomy's official Japanese designation is **"Omnibot 17μ i-SOBOT,"** where "μ" refers to the in-house micro-servos.

The robot has **17 powered degrees of freedom, one per servo**: 1 DOF in the head (yaw), 3 per arm (shoulder roll, shoulder pitch, elbow), and 5 per leg.[^5] The two fingers on each hand are adjustable mechanically but are **not powered**; marketing language about "23 points of articulation" includes these passive joints. Servo alignment is set by hand using a **1.5 mm hex wrench** included in the box, matching triangle index marks on the servo horn to the frame.

A single gyro provides pitch-axis balance feedback. There is no accelerometer, no edge sensor, and no bump sensor. Two head LEDs (one green eye, one blue side light) signal mood and status.

---

## Electronics: a 19-IC stack with unidentified main processors

Tomy advertised **19 ICs**: one main CPU, one voice-recognition CPU, and seventeen servo-embedded controllers — one inside each servo.[^5] The canonical Robot Watch teardown by Takayoshi Ishii (November 2007) found that **neither main processor could be identified by part number** from the markings visible on the ICs.[^5] Ishii speculated, based on package geometry, that the smaller IC was the host CPU and the larger was a Sensory Inc. RSC-4x-series voice-recognition chip, but neither identification has been confirmed. As of 2026, no firmware dump, ROM image, or datasheet for any of these three IC types has surfaced publicly.

The main board has been mapped in detail by Robot Watch.[^5] A two-board stack carries the suspected voice IC on a daughter board. An unpopulated **5-pin 2 mm-pitch header on the rear of the main PCB** is almost certainly a JTAG or in-circuit debug port; its pinout has not been documented. A switching regulator generates the **3.3 V logic rail**; a separate audio amplifier feeds a speaker mounted directly under the battery compartment. An **LMV358 dual rail-to-rail op-amp** sits next to the gyro to amplify its analog output, with five test points (T1–T5) clustered nearby. The board is RoHS-compliant for European sale.

Power comes from **three series AAA NiMH cells (3.6 V nominal)**, shipped as Sanyo eneloop cells (1.2 V × 3, 750 mAh) with a Tomy charger brick.[^4] Tomy explicitly disallows alkaline cells: oscilloscope traces show servo logic running at 3.3 V regulated, but instantaneous current demand from 17 simultaneously held servos drops the raw rail to approximately 3 V under load, which alkalines cannot sustain reliably. Runtime is approximately **60 minutes on eneloops** and roughly **40 minutes on alkalines**.[^6]

---

## Sensors

The sensor suite is minimal. Tomy's marketing copy references two gyros for balance, but the Robot Watch teardown found **a single Murata GYROSTAR piezoelectric vibrating-element gyro** mounted on a perpendicular daughter board, oriented to measure pitch.[^5] The visible "GYROSTAR" branding and the LMV358 reference circuit strongly suggest the **Murata ENC-03 family** (3 V supply, ±300 °/s), but no part number is printed on the component, and this identification is inferential.[^7]

A single **microphone** feeds the dedicated voice-recognition IC, supporting a fixed **10-word vocabulary** (English on US units, Japanese on Japanese units). The vocabulary is speaker-independent. Tomy's own engineers noted late in development that the head occasionally blocks the microphone; voice recognition is the weakest behavioral mode in owner reviews.[^8]

The **IR receiver is in the right shoulder**, accepting approximately 38 kHz modulated commands at up to ~1.4 m range over a ~30° cone. A **rear A/B channel switch** allows two i-SOBOTs to share a room without crosstalk. Direct sunlight overwhelms the receiver.

---

## Actuators: custom 17-servo train

The actuators are the engineering centerpiece. Each of the 17 servos contains a **6 mm × 14 mm coreless DC motor**, a **five-stage gear train with a measured ~1:826 theoretical reduction** (~1:578 at 70% efficiency), a position-feedback potentiometer on a D-cut shaft, a **slip clutch in the fourth-stage gear** to absorb impacts, and an embedded micro-CPU that decodes serial commands.[^5] Housings are pinned rather than screwed, and use metal output shafts with metal upper-case bearings. Tomy never published torque or speed specifications; Ishii's subjective estimate of approximately **90 °/s** is the only number in the public record.[^5]

The internal communication protocol was reverse-engineered by Robot Watch and by Osaka University researcher "hide."[^5] Servos are **daisy-chained on a per-limb serial bus running 2400 bps, 8N1, 3.3 V logic**, with each servo hard-coded to a position in an 8-byte packet:

```
[0xFF] [0x05] [pos1] [pos2] [pos3] [pos4] [pos5] [checksum]
```

**Position bytes use 128 as center and 0 as free/relax** (no torque). The checksum is an unsigned 8-bit sum with overflow ignored. Throughput is approximately 30 commands per second per bus. The protocol carries **position only — there is no speed parameter** — so motion speed is entirely dictated by the gear ratio and the servo's internal P-loop.

---

## IR remote protocol

The external IR protocol is the best-documented part of the system, independently decoded in 2009 by two researchers whose findings are consistent.[^9] [^10]

The physical layer uses a **38 kHz carrier, 2550 µs start burst, and pulse-distance encoding**: approximately 630 µs ON inter-bit, with 380 µs OFF representing a 0 and 850 µs OFF representing a 1, and a 205 ms inter-frame gap.[^10]

The logical layer is a **6-bit header** (1 channel bit + 2 type bits + 3 checksum bits) plus either a **16-bit or 24-bit payload**, for total messages of 22 or 30 bits.[^10] Type `01` produces 2-byte discrete-action messages; type `00` produces 3-byte messages that directly drive arm servo positions, hinting at a finer-grained control path that the stock remote underuses. The checksum is the byte-wise sum of header plus payload, folded by adding 3-bit groups, low 3 bits taken.[^10]

Krompiec's exhaustive sweep of the command space discovered approximately thirty undocumented opcodes the factory remote never sends, including animal sound effects, hidden dances (SWAN LAKE, MOONWALK, DISCO), narrative clips, self-test routines, a developer audio sync mode, and a FART command at `0x40`.[^10] Moody's published `Isobot.h` library exposes approximately 140 named macros covering the documented commands.[^9]

---

## Remote control unit

The bundled remote is a small handheld with a **monochrome LCD (~16×32 dots)**, two analog joysticks, a numeric/letter keypad (digits 1–4, letters P/K/G/L/R), three program-memory keys (M1/M2/M3), a MODE button, and an A/B channel switch on the back.[^4] Battery configuration is **3 × AA in the original Japanese package**; some US-market documentation cites 4 × AAA, which appears to be a genuine regional variation.

The remote supports four modes: **RC** (real-time joystick), **Program** (3 banks × 80 steps = 240 stored actions), **Special Action** (~180 canned motions, ~18 special combos), and **Voice Control**. Programmed sequences survive battery removal. Tomy never released a PC SDK, motion editor, or USB bridge; all choreography is performed on the LCD.

---

## Known failure modes

After nearly two decades of owner reports, the failure pattern is well established.

**Servo gear stripping** in the lower legs and ankles is the dominant hardware failure, often triggered by a single fall; shoulder servos are a close second. The original gear grease also dries to a gum over time, producing the characteristic "click" and stiffness on arm lift.[^11] [^12] The community-validated fix is to disassemble the servo and re-grease with white lithium grease. This is complicated by the press-pin housing design rather than screws. Because the servos are non-standard custom Tomy parts, **off-the-shelf replacements do not fit**.

**Calibration drift** is the second universal complaint: servos drift out of alignment against the triangle index marks, producing an unstable stance and inability to recover from prone. Several owners report that perfectly aligned units balance worse than slightly mis-aligned ones, suggesting the open-loop balance model assumes some neutral offset.[^12]

The **"dead after storage" syndrome** is the most catastrophic failure mode: a unit boots to 0 V at the main board even with fresh batteries. RoboSavvy users traced this to a **small two-band fusible resistor (red/black/gold) behind the battery compartment** that opens permanently; its rated value has never been definitively documented.[^11] The remote has its own characteristic failure: it powers up, runs approximately 10 seconds, blanks the LCD, then dies — a pattern consistent with a failing electrolytic capacitor or regulator.

---

## Replacement parts

There are **no aftermarket OEM parts, no Tomy spares, no iFixit guide, and no community 3D-printed replacement designs** on Thingiverse, Printables, MyMiniFactory, or Cults3D as of early 2026. The micro-servo gears are too small for FDM printing at usable tolerances, and the plastic frame rarely fails mechanically. The universal community advice is to **acquire a "for parts/not working" donor unit** — typically US$30–80 versus US$150–300 for working units — and harvest gears, servos, and boards.[^11] [^12] Battery cells are generic AAA NiMH; the Tomy charger is the only proprietary power accessory.

---

## Community extension projects

Despite the absence of a Tomy SDK, hobbyists built a meaningful extension layer.

**Knuckles904's "Pro Mini backpack"** mounts a 3.3 V Arduino Pro Mini directly on the robot's back, drawing from the i-SOBOT's own NiMH pack and combining an IR receiver, IR LED, and sonar input to enable autonomous obstacle avoidance using the canned IR motion library.[^13]

**Krompiec's "boobs-job" hack** replaced the AAA pack with a 3.7 V Li-Ion camera battery and tapped the IR receiver line directly on the PCB to inject commands without needing an external IR LED.[^10]

The **i-sobothacking.blogspot.com "headless hack"** tapped the inter-servo serial bus and drove individual servos with a microcontroller, proving the bus is accessible to anyone willing to solder.[^14]

**RobotShop's "Isobot Robot Version 1.5"** replaced a dead main board with a Basic Stamp BS2 issuing 8-byte serial frames directly.[^15] The **OTL/i-sobot GitHub repo** packages an Arduino sketch derived from this work, with a level-converter note for 5 V boards.[^16]

Synthiam community discussions concluded the **EZ-B v4 is too large to fit inside the shell, IoTiny is borderline, and Arduino Pico is the only board that fits comfortably** — explaining why no single-board computer conversion has been completed.[^17]

---

## Variants and product lineage

The product line is narrower than community rumors suggest. The **Japan release** wears blue and white with Japanese voice firmware; the **US/international release** is black and grey with English voice firmware. A later **Black Version (TKT30389)** sold in Japan with Japanese firmware.[^18] There is no confirmed "i-SOBOT EX" and no documented hardware revision of the main unit. The **i-SOBOT CAM Version**, announced in January 2007 at ¥41,790 with a Wi-Fi camera in a swiveling head and an intended October 2007 release, was quietly cancelled and never shipped.[^19]

The closest successors are **Robo-Q** (February 2009, ¥3,500, 3.4 cm — a non-programmable walking biped) and **i-SODOG** (spring 2013, ¥31,500, 15 servos, smartphone-controllable).[^20] The broader lineage runs back through the Omnibot (1984), Omnibot 2000 (1984), and Takara's Walkie Bits (2005).[^21]

---

## Historical context

The i-SOBOT was the **first major flagship product of the merged Takara Tomy** (merger effective 1 March 2006). Chief engineer **Yosuke Yoneda** had begun the project at Tomy before the merger. The Japan retail launch was **24–25 October 2007 at ¥29,800 (ex-tax)**; the US debut followed at approximately **US$300**.[^3] [^4]

| Award / record | Date | Notes |
|---|---|---|
| Guinness "smallest humanoid in mass production" | 2006 (listed in 2007 GWR book) | — |
| TIME "Best Inventions of 2007" | November 2007 | Robots category |
| METI Robot of the Year (Japan) | 18 December 2008 | Grand Prize |

Tomy's announced sales target was 300,000 units worldwide; actual unit sales were never disclosed. By late 2009, RobotShop and Hammacher were clearing stock at US$95.99–99, marking the practical end of the US market.[^22]

---

## Open questions

The following technical gaps remain unresolved as of the start of the Michiyuki project:

- **Main CPU part number** — markings not readable from published teardown photography; unpopulated 5-pin debug header on the main PCB is the most likely route to identification
- **Voice IC part number** — package geometry suggests Sensory RSC-4x family, but this is inferential and unconfirmed
- **Per-servo controller IC part number** — never identified in any published teardown
- **Servo torque and speed specifications** — Tomy never published these; Ishii's ~90 °/s is the only figure in the public record
- **Fusible resistor value** — the component responsible for the "dead after storage" failure has never been measured or documented
- **Firmware** — no dump exists; the unpopulated debug header is the most likely access point

---

## References

[^1]: Guinness World Records — Smallest mass-produced humanoid robot (i-SOBOT). Listed in the 2007 GWR book; confirmed by Tomy press materials. <https://www.guinnessworldrecords.com/>

[^2]: Pink Tentacle — i-SOBOT named "2008 Robot of the Year." (December 2008). <https://pinktentacle.com/2008/12/i-sobot-named-2008-robot-of-the-year/>

[^3]: Engadget — Takara Tomy's Omnibot2007 i-SOBOT, "the world's smallest robot." (January 2007). <https://www.engadget.com/2007-01-23-takara-tomys-omnibot2007-i-sobot-the-worlds-smallest-robot.html>

[^4]: Robot Magazine — Tomy i-SOBOT in BotMag Fall 2007. (Fall 2007). <https://www.yumpu.com/en/document/view/6675354/tomy-i-sobot-in-botmag-fall-2007pdf>

[^5]: Robot Watch — Takara Tomy "Omnibot 17μ i-SOBOT" teardown and analysis report. Takayoshi Ishii. (8 November 2007). <https://robot.watch.impress.co.jp/cda/column/2007/11/08/731.html>

[^6]: RoboSavvy Forum — Servo sound and trembling thread. <https://forum.robosavvy.com/viewtopic.php?t=3277>

[^7]: Open Impulse — Murata ENC-03M Single Axis Gyro Sensor Datasheet. <https://www.openimpulse.com/blog/wp-content/uploads/wpsc/downloadables/ENC-03M-Single-Axis-Gyro-Sensor-Datasheet.pdf>

[^8]: Robot Reviews — Tomy's i-SOBOT review thread. <http://www.robotreviews.com/chat/viewtopic.php?t=7799>

[^9]: Arduino Forum — i-Sobot hacked or Pro Mini shield (Knuckles904 / Miles Moody IR library). <https://forum.arduino.cc/t/i-sobot-hacked-or-pro-mini-shield/8836>

[^10]: minkbot.blogspot.com — iSobot Infrared Remote Protocol Hack. Michal Krompiec. (August 2009). <http://minkbot.blogspot.com/2009/08/isobot-infrared-remote-protocol-hack.html>

[^11]: RoboSavvy Forum — My Isobot is dead. <https://forum.robosavvy.com/viewtopic.php?f=12&t=3680>

[^12]: RoboSavvy Forum — i-Sobot arm problem. <https://robosavvy.com/forum/viewtopic.php?t=8158>

[^13]: Make: — Arduino controlled I-Sobot. <https://makezine.com/article/technology/arduino/arduino-controlled-i-sobot/>

[^14]: i-sobot hacking blog. <http://i-sobothacking.blogspot.com/>

[^15]: RobotShop Community — Isobot Robot Version 1.5. <https://community.robotshop.com/forum/t/isobot-robot-version-1-5/1122>

[^16]: GitHub — OTL/i-sobot: i-sobot Arduino sketch. <https://github.com/OTL/i-sobot>

[^17]: Synthiam Community — Isobot/Robi robot thread. <https://synthiam.com/Community/Questions/Isobot-Robi-robot-20507>

[^18]: HLJ.com — Omnibot i-Sobot Black Version (TKT30389). <https://www.hlj.com/omnibot-i-sobot-black-version-tkt30389>

[^19]: New Atlas — i-SOBOT: the smallest humanoid robot in production. <https://newatlas.com/i-sobot-the-smallest-humanoid-robot-in-production/8264/>

[^20]: Engadget — i-SODOG robot unveiled. (June 2012). <https://www.engadget.com/2012-06-17-takara-tomy-isodog-robot-unveiled.html>

[^21]: Wikipedia — Omnibot. <https://en.wikipedia.org/wiki/Omnibot>

[^22]: RobotShop Community — Buy I-Sobot, World's Smallest Humanoid Robot, For Only $95.99. <https://community.robotshop.com/blog/show/buy-i-sobot-worlds-smallest-humanoid-robot-for-only-95-99>
