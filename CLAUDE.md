# CLAUDE.md — Michiyuki

This file tells Claude Code what it needs to know to work on this repository effectively.

---

## What this project is

**Michiyuki** is an open source preservation, restoration, and reverse engineering project for the **Tomy i-SOBOT** — a 17-DOF biped humanoid toy robot, historically notable as the world's smallest mass-produced biped humanoid robot at the time of its release.

The project name comes from *michiyuki* (道行き): "going along the road," a Noh/Kabuki term for stylized travel scenes, and a style of coat whose square neckline mirrors the robot's boxy form.

**Goals:**
- Document the original hardware thoroughly before knowledge is lost
- Maintain and restore functional units
- Reverse engineer internals to understand and extend the robot's functions
- Source or create replacement parts
- Ultimately produce replacement PCB designs for damaged or unrepairable boards

**Design philosophy:** documentation before modification; understanding before replacement; original behavior preserved as a baseline, not a ceiling.

---

## Repository layout

```
docs/          Human-readable documentation (hardware, protocols, assembly)
hardware/      Schematics, PCB designs, board photos, component maps
firmware/      Disassembly, annotations, any reconstructed firmware
remote/        IR remote documentation and protocol work
parts/         Replacement part sourcing notes, 3D models, specs
references/    Prior art, teardowns, external sources (read-only mirror content)
```

---

## Licenses

| Scope    | License        |
|----------|----------------|
| Docs     | CC-BY-4.0      |
| Firmware | GPL-3.0        |
| Hardware | CERN-OHL-S-2.0 |

Apply the correct license header when creating new files. Do not mix licenses across scopes.

---

## Known hardware facts

These are confirmed unless contradicted by physical inspection:

- **DOF:** 17
- **ICs:** 19 total — 1 main CPU, 1 voice IC, 17 per-servo controllers. **No IC part numbers have been publicly identified.** Do not assert or infer part numbers without physical evidence.
- **Servo bus:** 2400 bps 8N1 serial, 8-byte packets
- **IR protocol:** 38 kHz carrier; fully decoded by Krompiec (2009) — see `references/`
- **Gyro:** Single Murata GYROSTAR (pitch axis only)
- **Power:** 3× AAA NiMH
- **Servos:** Custom proprietary micro-servos; no known off-the-shelf replacement exists. This is a central preservation challenge.

---

## Canonical references

| Source | Location | Status |
|--------|----------|--------|
| Robot Watch teardown (Takayoshi Ishii, 2007) | `references/` | Primary hardware reference; treat as confirmed |
| Krompiec minkbot IR decode (2009) | `references/` | Primary IR protocol reference; treat as confirmed |
| OTL Arduino sketch | `references/` | Supporting code reference |
| i-sobothacking blog | `references/` | Supporting reference |

When making claims that can be grounded in these sources, cite them explicitly.

---

## Physical specimens

| Unit | State | Notes |
|------|-------|-------|
| Robot 1 | Complete with remote, **functioning** | **Do not disassemble, modify, or risk operational state without explicit user confirmation** |
| Robot 2 | No remote, already disassembled | Primary dissection subject — safe to work with |

---

## Epistemic rules — read carefully

These apply to all contributions, human or AI:

1. **Distinguish confirmed facts from inferences.** Use language like "confirmed," "inferred," "unverified," or "unknown" explicitly. Do not fill gaps with plausible-sounding information.
2. **IC part numbers are unknown.** The main CPU, voice IC, and per-servo controller ICs have never been publicly identified. Do not speculate about their identity without physical evidence.
3. **Flag uncertainty.** If something isn't supported by a source in `references/` or by physical inspection of Robot 2, say so.
4. **Ground claims in sources.** When a reference document supports a claim, cite it. When it doesn't, say so.

---

## Commit conventions

- Scope commits to a single task or closely related set of changes
- Commit message format: `<type>(<scope>): <short description>`
  - Types: `docs`, `hardware`, `firmware`, `remote`, `parts`, `refs`, `chore`
  - Example: `docs(servo-bus): document 8-byte packet structure`
- Documentation commits are encouraged — partial progress is worth preserving
- Ask the user whether findings should be committed before moving on to the next task

---

## What not to do

- Do not assert IC part numbers without physical evidence
- Do not modify or create risks for Robot 1 without explicit user confirmation
- Do not carry over work from a previous session unprompted — wait for the user to establish scope
- Do not use placeholder or speculative content in `references/` — that directory mirrors external prior art