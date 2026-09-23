# TIGSICH – finger remote for IPOTools SuperTIG 200Di

A passive, torch-mounted finger control that replaces the IPOTools **FT-47K-CL8** foot pedal.
Squeezing the lever first closes the trigger (arc on), then raises the welding current as the
lever moves further, the same way the pedal behaves. Releasing it returns the current to minimum and stops the arc.
The concept is the same as the [6061 Finger Pedal](https://www.6061.com/fingerpedal.htm).

> **Status: design rev 1, not yet prototyped.** The socket voltages on the machine have **not been
> measured** yet. See [Open items](#open-items--before-first-use).

![Mechanism side view](docs/mechanism.svg)

| Schematic | PCB (top) | 3D (approximate models) |
|---|---|---|
| ![schematic](docs/schematic.png) | ![pcb](docs/pcb_layout.png) | ![3d](docs/pcb_3d.png) |

---

## 1. Machine interface

The machine has a GX20 7-pin remote socket. The pinout is the same one used by AHP/PrimeWeld pedals
(`configuration.jpeg`). It was confirmed by measuring the user's own FT-47K-CL8 pedal:

| Plug pin | Wire colour (diagram) | Function | Measured on FT-47K-CL8 |
|---|---|---|---|
| 1 | black | trigger contact | open → short when pedal pressed |
| 2 | white | trigger contact | 〃 |
| 3 | blue | pot end A | 3–4 and 4–5 swing 0.5 kΩ … 50 kΩ in opposite directions |
| 4 | brown | pot wiper | |
| 5 | red | pot end B | |
| 6–7 | – | remote-detect link | 0 Ω (bridged inside the plug) |

**Decisions based on this:**

- **Pot value 50 kΩ linear.** The pedal model name (47K) and the measurement (≈50 kΩ end to end) agree.
  The ±20 % pot tolerance doesn't matter, because the machine reads the wiper as a *ratio* of the reference voltage.
- **The 0.5 kΩ minimum is just the pot's residual resistance.** The PTA datasheet specifies
  "500 Ω or 1 % max" residual resistance, which is exactly what the pedal shows. So the series trim resistors (R1/R2)
  I first proposed were **dropped**: they would add nothing.
- **Pins 6–7 are bridged inside the GX20 plug, not on the PCB.** This keeps the cable at 5 cores (thinner and more
  flexible on a torch), and the remote is still detected even if the board is disconnected from its cable.

### Supply voltage (assumed, not measured)

There is **no power pin** on this plug. The only voltages present are the pot reference (across 3–5) and the
trigger pull-up (across 1–2). Machines of this type typically put 5–15 V there, so the design
**assumes ≈12 V**. Because the design is passive, the voltage only has to stay within the parts' ratings:

| Part | Rating | Margin at 12 V |
|---|---|---|
| PTA1543 (15 mm, linear) | 100 V DC, 0.05 W | 12 V across 50 kΩ = 2.9 mW |
| D2F-01L3 | 30 V DC, 0.1 A, gold contacts, min. load 1 mA @ 5 V | fine for a signal-level input |

If you measure more than 30 V on pins 1–2, the switch must change (see open items).

---

## 2. Architecture: passive, not Hall + MCU

Alternatives considered:

| Option | Why not chosen |
|---|---|
| Hall sensor + MCU + digital pot | Needs power, and the plug has no supply pin. It would have to steal current from a 47 kΩ pot reference (≈0.2 mA available) or use a battery. It also needs a high-voltage digital pot (MCP41HV51 / AD5290) plus HF-start hardening. More parts, more ways to fail. |
| Rotary pot at the lever pivot | A finger lever swings only about 35°, while a pot needs about 270° for its full range. Only about 12 % of the current range would be reachable without gearing. |
| **Slide pot + microswitch, driven by a lever (chosen)** | Passive, electrically identical to the original pedal, uses the pot's full range, and every part is cheap and replaceable. |

---

## 3. Parts

| Ref | Part | Why |
|---|---|---|
| RV1 | **Bourns PTA1543-2015CIB503**: 15 mm travel, 50 kΩ linear (B), single gang, PC pins, **no centre detent**, 15 mm insulated (CI) lever | Short stroke, so the lever can drive the full range. Metal frame with M2 threads on top for fixing to the housing. **Not** `-2215…`: the second `2` in that code means a *centre detent*, which would click halfway through the squeeze. |
| SW1 | **Omron D2F-01L3**: ultra-subminiature, simulated-roller lever (R1.3), gold contacts (`-01`) | Gold contacts are rated for low-level signal loads. The roller-type lever tolerates a sliding cam better than the plain hinge lever (D2F-01L) I suggested at first. 1,000,000 mechanical operations. Uses **COM + NO**, so a broken switch or linkage fails *open*, i.e. the arc stays off. NC is left unconnected. |
| J1 | Solder pads with strain-relief holes (`SolderWire-0.15sqmm_1x05_P4mm…_Relief`) | No connector to shake loose on a torch. The wire passes up through a 2 mm hole before reaching its pad, so it's held in place. |
| H1, H2 | M2 holes | Fix the front of the board to the housing. The rear is held by the pot's own M2 threads through the housing lid. |
| Cable | 5 × 0.14–0.25 mm², flexible, ~4 m, GX20-7 female plug | Same length as the original pedal (4.3 m). |

**Pot life:** the PTA is rated at **15,000 slide cycles**. That's fine for hobby use (several years), but it is the
wear part, which is why it's a plug-in THT part at an easy-to-reach position. A conductive-plastic slide pot of the same
footprint is the upgrade path if it wears out.

**No filtering or ESD parts on the board.** The original pedal is a bare pot and switch, and the machine's input
circuitry was designed for that. Adding capacitors could change how the machine's input behaves for no known benefit.
If HF-start interference shows up in testing, the first fix is a shielded cable with the shield grounded at the machine end.

---

## 4. Mechanism (see `docs/mechanism.svg`)

It's a **bell-crank with a pin-in-slot (Scotch-yoke) drive**:

1. **Lever:** the finger lever and a downward **drive arm** are one rigid part that turns on pivot **P**.
2. **Pin and shoe:** a pin at the end of the drive arm runs in a **vertical slot** in a small **shoe** pressed onto the pot lever.
3. **Motion:** when the arm swings, the pin's horizontal motion moves the slider, and its small vertical motion (1.2 mm) is absorbed by the slot.
4. **Travel:** slider travel is x = 2·r·sin(θ/2), with r = pivot-to-pin distance and θ = total swing.

| r (pivot → pin) | θ | Slider | Finger travel at 35 mm | Pivot height above PCB bottom |
|---|---|---|---|---|
| 20 mm | 45° | 15.3 mm | ≈27 mm | ≈36 mm |
| **25 mm (chosen)** | **35°** | **15.0 mm** | **≈21 mm** | **≈41 mm** |
| 30 mm | 29° | 15.0 mm | ≈18 mm | ≈46 mm |

**Why r = 25 mm and θ = 35°:** it balances finger travel against housing height. The swing is symmetric about vertical
(±17.5°), so the pot position is linear in lever angle to within 1.6 %.

**Direction:**

- **At rest:** the drive arm leans forward, so the **slider sits at the front end**.
- **At full press:** it sits at the rear end.
- **Result:** at rest the wiper is next to RV1 pin 3, which is **plug pin 5**. So at rest P4–P5 ≈ 0.5 kΩ and P3–P4 ≈ 50 kΩ.

Measure your pedal at rest. **If it's the other way round (P3–P4 low at rest), swap the wires on plug pins 3 and 5.**
Nothing on the PCB changes.

**Trigger timing:**

- **Cam sector:** a cam sector on the lever sits in the switch's plane, 9 mm beside the pot. Its underside is an arc
  **centred on the pivot** (R ≈ 29.6 mm), so once it is over the D2F roller, further rotation doesn't push the roller
  any deeper (a *dwell*). That protects the switch's small 0.5 mm overtravel.
- **Engagement:** it reaches the roller after about **2–2.5°** of swing (slider ≈1.1 mm, ≈7 % current). The arc
  therefore starts near minimum current, and on release the current falls back before the contact opens.
- **Tuning:** R is a to-be-tuned number. D2F operating-position tolerance is ±1.2 mm, so print the sector and adjust
  (or shim the board) on the first prototype.

**Stops and spring:**

- **End stops:** both are in the housing, not on the pot (its internal stop is rated 5 kgf, and a finger can exceed that).
- **Return spring:** a torsion spring at P, ≈1–2 N at the finger pad. The pot needs 30–250 gf to slide, plus 0.8 N for the switch.

**Height trade-off:** with the pot lying flat, the pivot sits ≈41 mm above the PCB bottom. If that's too tall on the
torch, the options are (a) r = 20 mm / θ = 45° (−5 mm height, +6 mm finger travel), or
(b) mount the board vertically along the side of the torch handle so the torch body takes up part of that height.

---

## 5. PCB

- **Board:** 54 × 22.5 mm, 2 layers, 1.6 mm, rounded corners (R2), 0.4 mm tracks (signal-level currents).
  One track (POT_B) runs on the bottom layer under the pot. There is no ground plane: the circuit has no ground.
- **Placement follows the mechanism:**
  - **Cable:** the pads are at the **rear** edge, where the torch hose runs.
  - **Pot:** RV1's centre (slider mid-travel) is directly under the pivot axis.
  - **Switch:** SW1 sits beside the pot, with its **roller exactly under the pivot axis**.
  - **Markings:** the pivot line and roller position are drawn on the `User.Drawings` layer for the housing design.
- **J1 pad order** (top to bottom on the board) is chosen for untangled routing and is marked on the silkscreen:

| J1 pad | Silk | Net | Plug pin | Diagram colour |
|---|---|---|---|---|
| 1 | P3 | POT_A | 3 | blue |
| 2 | P4 | WIPER | 4 | brown |
| 3 | P5 | POT_B | 5 | red |
| 4 | P1 | TRIG_COM | 1 | black |
| 5 | P2 | TRIG_NO | 2 | white |

- **D2F footprint:** it's custom (`tigsich.pretty/SW_Omron_D2F-01L3`), because KiCad's library doesn't have one.
  It was made from the Omron datasheet: 3 × Ø1.2 mm holes at 5.08 mm pitch, body 12.8 × 5.8 mm. The pad numbers follow the
  `Switch:SW_SPDT` symbol: **2 = COM (left), 1 = NO (middle, square pad), 3 = NC (right)**.
- **3D models:** `tigsich.3dshapes/*.wrl` are **rough block models** for fit checks only. KiCad has no PTA1543 or
  D2F model. Download the vendor STEP files before designing the housing.

**Verification:**

- **ERC:** 0 violations.
- **DRC:** 0 violations, 0 unconnected, 0 schematic-parity issues.
- **Tool:** KiCad 10.0.6.

---

## 6. Files

| Path | Content |
|---|---|
| `tigsich.kicad_pro/.kicad_sch/.kicad_pcb` | KiCad project |
| `tigsich.pretty/` | Project footprint library (D2F) |
| `tigsich.3dshapes/` | Approximate 3D models |
| `docs/mechanism.svg` | Scaled side view and lever-swing timing chart |
| `docs/schematic.pdf` / `.png` | Schematic exports |
| `docs/pcb_layout.png`, `docs/pcb_3d.png` | Board renders |
| `configuration.jpeg` | 7-pin plug pinout reference |

---

## Open items / before first use

1. **Measure the socket.** With the machine on and the pedal unplugged, measure the DC voltage across 3–5, across 1–2, and from
   each pin to the case. It must be **≤ 30 V** on 1–2 for the D2F-01. Don't strike an arc (HF start) while probing.
2. **Check the pot direction** against the pedal at rest (section 4), and swap plug pins 3/5 if needed.
3. **Check part dimensions** against real parts: PTA lever height (sets the pin height and the pivot height),
   and the D2F roller free and operating positions (set the cam radius).
4. **Design the housing:** lever and drive arm, shoe, cam sector, torsion spring, end stops, strap slots, and a cable gland.
5. **Test first on scrap:** check minimum and maximum current, that the trigger releases reliably, and that HF start
   doesn't cause flicker. If it does, use a shielded cable.

## Sources

- [SSC C910-0725 info sheet (same 7-pin pinout)](https://ssccontrols.com/uploads/Product-Information-Sheet-C910-0725-TIG-Foot-Controls.pdf)
- [IPOTools FT-47K-CL8 foot pedal](https://ipotools.eu/product/tig-foot-pedal/)
- [Bourns PTA series datasheet](https://www.bourns.com/docs/product-datasheets/pta.pdf)
- [Omron D2F datasheet](https://omronfs.omron.com/en_US/ecb/products/pdf/en-d2f.pdf)
- [6061 Finger Pedal (concept reference)](https://www.6061.com/fingerpedal.htm)
