# TIGSICH – thumb-slider remote for IPOTools SuperTIG 200Di

A passive, torch-mounted current control that replaces the IPOTools **FT-47K-CL8** foot pedal.
A **thumb slider** sets the welding current and stays where you leave it. A **push-on / push-off button** starts and
stops the arc. The concept is the same as the [6061 Slide Lever](https://www.6061.com/slidelever.htm).

> **Status: design rev 3, not yet prototyped.** The socket voltages on the machine have **not been
> measured** yet. See [Open items](#open-items--before-first-use).

![Side view](docs/mechanism.svg)

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
- **No series trim resistors.** The pedal's 0.5 kΩ minimum is simply the pot's residual resistance. The PTA datasheet
  specifies "500 Ω or 1 % max", which is exactly what the pedal shows.
- **Pins 6–7 are bridged inside the GX20 plug, not on the PCB.** This keeps the cable at 5 cores (thinner and more
  flexible on a torch).
- **The switch itself must latch.** IPOTools states the remote works **only in 2T mode**, so the machine's 4T
  latch can't be used with a momentary button.

### Supply voltage (assumed, not measured)

There is **no power pin** on this plug. The only voltages present are the pot reference (across 3–5) and the
trigger pull-up (across 1–2). Machines of this type typically put 5–15 V there, so the design
**assumes ≈12 V**. Because the design is passive, the voltage only has to stay within the parts' ratings:

| Part | Rating | Margin at 12 V |
|---|---|---|
| PTA1543 (15 mm, linear) | 100 V DC, 0.05 W | 12 V across 50 kΩ = 2.9 mW |
| TL2230 | 30 V DC, 100 mA | fine for a signal-level input |

---

## 2. Architecture

| Option | Outcome |
|---|---|
| Hall sensor + MCU + digital pot | **Rejected.** Needs power, and the plug has no supply pin. It would have to steal current from a 47 kΩ pot reference (≈0.2 mA) or use a battery. It also needs a high-voltage digital pot plus HF-start hardening. |
| Rev 1: finger lever (bell-crank) driving a slide pot | **Rejected on size.** A 35° swing needs a 25 mm drive arm to move the slider 15 mm, which put the lever top about 50 mm above the PCB. The board was 22.5 mm wide. |
| Rev 2: thumb slider + C&K PVA1 EE H4 latching button | **Rejected on height.** The PVA1 is 15 mm tall above the PCB, which set a 2 mm lid and a total height of about 20 mm. The target is under 15 mm. |
| **Rev 3: thumb slider + E-Switch TL2230 latching button (chosen)** | Passive, no linkage. **14.1 mm** maximum height (button, unlatched), 55.5 × 19 mm board. |

**How it's used:**

1. **Set the current:** slide the knob. Rear = minimum, front = maximum. It stays where you leave it, because the pot's
   own 30–250 gf friction holds it.
2. **Start the arc:** press the button once. It latches (the plunger stays 1 mm lower) and closes pins 1–2.
3. **Stop the arc:** press it again.

The machine stays in **2T mode**. The trade-off is that nothing stops the arc if you drop the torch: that takes a
deliberate press. The same footprint takes the momentary `TL2230OAF140` (hold to weld) if that's preferred.

---

## 3. Parts

| Ref | Part | Why |
|---|---|---|
| RV1 | **Bourns PTA1543-2015CIB503**: 15 mm travel, 50 kΩ linear, single gang, PC pins, **no centre detent**, 15 mm insulated (CI) lever | Short slider (30 mm body), so the unit stays short. 15 mm travel ≈ 13 A/mm on a 200 A machine (the user's choice). **Not** `-2215…`: that code has a *centre detent*. The **lever lug is cut down** to ≈ 12.5 mm above the PCB bottom, just enough to reach through the lid into the knob. |
| SW1 | **E-Switch TL2230EEF140**: DPDT, latching (EE), 140 gf, 7 × 7 mm, THT | The lowest latching switch with a proper datasheet that I found. Plunger top 12.5 mm above the PCB (11.5 mm latched). 1.0 mm to latch, 1.8 mm full travel. The datasheet shows the pinout and contact states explicitly. |
| J1 | Solder pads with strain-relief holes (`SolderWire-0.15sqmm_1x05_P4mm…_Relief`) | No connector to shake loose on a torch. The wire passes up through a 2 mm hole before reaching its pad, so it's held in place. |
| Cable | 5 × 0.14 mm² flexible (insulation OD ≤ 1.5 mm), ~4 m, GX20-7 female plug | Same length as the original pedal (4.3 m). |

**Switches I compared** (all heights above the PCB bottom):

| Switch | Size (mm) | Button top: unlatched / latched | Verdict |
|---|---|---|---|
| C&K PVA1 EE H4 | 9.8 × 9.2 | 16.6 / 15.1 | Too tall (rev 2) |
| **E-Switch TL2230EEF140** | **7 × 7** | **14.1 / 13.1** | **Chosen** |
| Alps SPEF110100 | 12.2 wide, 9 deep | ≈ 14.6 unlatched | Wider, only 14.5 V DC rated |
| Generic 5.8 mm "self-lock" switches | 5.8 × 5.8 | similar | No trustworthy datasheet |

**Switch wiring:** both poles are in parallel. The TL2230 datasheet shows **rest = 1–2 and 4–5 closed**, and
**pushed / latched = 2–3 and 5–6 closed**. So **2 + 5 → TRIG_COM** and **3 + 6 → TRIG_NO**, and the rest contacts 1 and 4
are left unconnected. The two contacts in parallel give redundancy for a low-level signal.

**Switch life is the weak point.** The TL2230 is rated for **10,000 electrical cycles**, about 5,000 welds (on + off).
That's fine for hobby use, and it's a cheap THT part that's easy to replace.

**Pot life:** the PTA is rated at **15,000 slide cycles**. With a set-and-leave slider (rather than a pedal that moves on
every weld) that lasts a long time. A conductive-plastic slide pot of the same footprint is the upgrade path.

**No filtering or ESD parts on the board.** The original pedal is a bare pot and switch, and the machine's input
circuitry was designed for that. If HF-start interference shows up, the first fix is a shielded cable with the shield
grounded at the machine end.

---

## 4. Mechanics and heights (see `docs/mechanism.svg`)

All heights are measured from the PCB bottom, using nominal datasheet values:

| Item | Height |
|---|---|
| PCB | 1.6 mm |
| Pot body top | 8.1 mm |
| TL2230 body top | 8.6 mm |
| Lid (1.5 mm), resting just above the switch body | 8.8 → **10.3 mm** |
| Pot lever lug (cut down) | ≈ 12.5 mm |
| Slider knob top | **≈ 13.3 mm** |
| TL2230 plunger top: latched / unlatched | 13.1 / **14.1 mm** (highest point) |

**The height target (< 15 mm) is met** without a separate button cap. Add the housing floor under the PCB
(≈ 1–1.5 mm) to get the total thickness on the torch.

Housing notes:

- **Button collar:** the 3 × 2.6 mm plunger sticks up through a hole in the lid. A small **raised collar** around it
  (to ≈ 12 mm) helps a gloved finger find it and prevents accidental presses. A printed cap of up to 0.8 mm can go on
  the plunger if you want a bigger contact area; keep it under 15 mm.
- **Lid slot:** the slot for the pot lever is 15 mm of travel plus the 4 mm lever, so about 19.5 × 1.5 mm.
  The **knob has a skirt** wider than the slot, so it covers the slot at every position and keeps dust out of the pot.
- **Lid fixing:** the lid screws into the pot's two **M2 threaded holes** (on top of the pot frame, 26 mm apart).
  This clamps lid, pot and PCB into one stack. The PCB has **no mounting holes**, because there's no room at 19 mm wide.
  The PCB rests in grooves or on ribs in the housing floor, which also carry the button-press force.
- **Mounting on the torch:** two strap or zip-tie slots in the housing floor. The cable exits at the rear, alongside the torch hose.

---

## 5. PCB

- **Board:** **55.5 × 19 mm**, 2 layers, 1.6 mm, rounded corners (R2), 0.4 mm tracks (signal-level currents).
  All copper is on the top layer, with no vias. There is no ground plane: the circuit has no ground.
- **Width:** the 19 mm is set by the 5-pad cable column (16 mm of pads at 4 mm pitch, plus pad size and edge clearance).
  The pot is only 11 mm wide including its tabs, and the switch 7 mm. A 3.7 mm pitch pad set would bring the board to
  about 18 mm, but only with cable insulation of 1 mm OD or less.
- **Layout, rear to front (cable → torch head):**
  1. **Cable pads (J1):** at the rear edge.
  2. **Slide pot (RV1):** its slider runs along the board centreline.
  3. **Latching button (SW1):** centred, 1.5 mm in front of the pot body.
- **Routing:** the pot and switch connections that pass alongside the pot run in lanes above and below the pot body.
- **Pot direction:** RV1 pin 1 (the rear end) connects to plug pin 3, so the **rear slider position = wiper next to
  plug pin 3**, i.e. P3–P4 ≈ 0.5 kΩ. If the machine gives *maximum* current at the rear, swap the wires on plug pins 3 and 5.
- **J1 pad order** (top to bottom) is chosen so nothing crosses. It's printed on the **bottom** silkscreen next to each pad:

| J1 pad | Silk | Net | Plug pin | Diagram colour |
|---|---|---|---|---|
| 1 | P1 | TRIG_COM | 1 | black |
| 2 | P3 | POT_A | 3 | blue |
| 3 | P4 | WIPER | 4 | brown |
| 4 | P5 | POT_B | 5 | red |
| 5 | P2 | TRIG_NO | 2 | white |

- **TL2230 footprint:** it's custom (`tigsich.pretty/SW_E-Switch_TL2230`), because KiCad's library doesn't have one.
  It was made from the datasheet PCB layout (component side): 2 rows of 3 × Ø0.8 mm holes, 2 mm pitch, rows 5 mm apart,
  pin 1 square. The pad numbers match the `Switch:SW_Push_DPDT` symbol: B = common 2/5, A = rest 1/4, C = pushed 3/6.
- **3D models:** `tigsich.3dshapes/*.wrl` are **rough block models** for fit checks only. KiCad has no PTA1543 or
  TL2230 model, and the pot model shows the lever at full length (not cut). E-Switch offers a STEP file for the TL2230;
  download the vendor STEP files before designing the housing.

**Verification:**

- **ERC:** 0 violations.
- **DRC:** 0 violations, 0 unconnected, 0 schematic-parity issues.
- **Tool:** KiCad 10.0.6.

---

## 6. Files

| Path | Content |
|---|---|
| `tigsich.kicad_pro/.kicad_sch/.kicad_pcb` | KiCad project |
| `tigsich.pretty/` | Project footprint library (TL2230) |
| `tigsich.3dshapes/` | Approximate 3D models |
| `docs/mechanism.svg` | Scaled side view with heights |
| `docs/schematic.pdf` / `.png` | Schematic exports |
| `docs/pcb_layout.png`, `docs/pcb_3d.png` | Board renders |
| `configuration.jpeg` | 7-pin plug pinout reference |

---

## Open items / before first use

1. **Measure the socket.** With the machine on and the pedal unplugged, measure the DC voltage across 3–5, across 1–2, and from
   each pin to the case. It must be **≤ 30 V** on 1–2 for the TL2230. Don't strike an arc (HF start) while probing.
2. **Check the switch.** Before soldering, check with a meter that the TL2230 closes 2–3 (and 5–6) only when latched.
   On the finished board, check that P1–P2 is open when the button is up and closed when it's latched.
3. **Check the pot direction:** the rear slider position should give minimum current. Swap plug pins 3/5 if needed.
4. **Check part dimensions** against real parts. The PTA lug length after cutting and the TL2230 body height set the
   lid and knob heights.
5. **Design the housing:** lid with slot, button hole and collar, M2 screws into the pot, knob with skirt,
   PCB grooves, strap slots, cable gland.
6. **Test first on scrap:** check minimum and maximum current, that the latch works with gloves on, and that HF start
   doesn't cause flicker. If it does, use a shielded cable.

## Sources

- [SSC C910-0725 info sheet (same 7-pin pinout)](https://ssccontrols.com/uploads/Product-Information-Sheet-C910-0725-TIG-Foot-Controls.pdf)
- [IPOTools FT-47K-CL8 foot pedal](https://ipotools.eu/product/tig-foot-pedal/)
- [Bourns PTA series datasheet](https://www.bourns.com/docs/product-datasheets/pta.pdf)
- [E-Switch TL2230 datasheet](https://configured-product-images.s3.amazonaws.com/Datasheets/TL2230.pdf)
- [E-Switch TL2230 product page](https://www.e-switch.com/product-catalog/tl2230-series-subminiature-pushbutton-switch)
- [C&K PVA series datasheet (rev 2)](https://www.mouser.com/datasheet/2/240/pva-3050984.pdf)
- [Alps SPEF series (compared)](https://tech.alpsalpine.com/e/products/category/switches/sub/03/series/spef/)
- [6061 Slide Lever (concept reference)](https://www.6061.com/slidelever.htm)
