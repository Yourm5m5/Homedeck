# HomeDeck Enclosure: Assembly Guide

The numbers below match the labels in the exploded view.

## Parts

| # | Part | Type | Qty | Notes |
|---|---|---|---|---|
| 1 | Front bezel | Printed | 1 | Holds the screen; sensor bay behind the top strip |
| 2 | Sunton ESP32-8048S070C | Bought | 1 | 7" display + ESP32-S3 |
| 3 | LD2410C mmWave radar | Bought | 1 | Sits in the sensor bay, sees through 2 mm of plastic |
| 4 | BH1750 (GY-302) light sensor | Bought | 1 | Sits in the sensor bay, behind the 10 × 5 mm window |
| 5 | SHT40 breakout | Bought | 1 | Sits in a cradle behind its own vents in the back shell |
| 6 | Back shell | Printed | 1 | Vents, SHT40 cradle, cable exit at the bottom |
| 7 | M3 × 16 socket-head screws | Bought | 4 | Back shell → bezel |
| 8 | Weighted base | Printed | 1 | Tilted 20° slot for the display, cavity for the PCB |
| 9 | HomeDeck add-on PCB | Made (JLCPCB) | 1 | USB-C power in + sensor/buzzer sockets |
| 10 | M3 × 6 screws | Bought | 4 | PCB → bottom-plate posts |
| 11 | Bottom plate | Printed | 1 | PCB standoffs, rubber-foot recesses |
| 12 | M3 × 6 screws | Bought | 4 | Bottom plate → base |
| — | M3 brass heat-set inserts | Bought | 12 | 4 in bezel bosses, 4 in plate posts, 4 in base bosses |
| — | Female–female jumper wires, ~25 cm | Bought | ~12 | Sensors → add-on PCB sockets |
| — | 1.25 mm JST cables | Bought | 3 | Add-on PCB ↔ Sunton P1 / P3 / P4 (usually included with the board) |
| — | Rubber feet | Bought | 4 | Under the bottom plate |

## How it fits together

```
        DISPLAY STACK (front → back)                BASE STACK (top → bottom)
        ─────────────────────────────               ─────────────────────────
        1  Front bezel                              8  Weighted base
        3  LD2410C ┐ in the sensor bay              10 4× M3×6 ──┐
        4  BH1750  ┘ behind the top strip           9  Add-on PCB ┘ screws onto posts
        2  Sunton display module                    11 Bottom plate
        5  SHT40 (in back-shell cradle)             12 4× M3×6 (up into the base)
        6  Back shell
        7  4× M3×16 (through shell into bezel)

        The display stack drops into the base's 20° slot.
        Cables run: back-shell cable exit → slot floor tunnel → base cavity → add-on PCB.
```

## Assembly steps

**Before you start:** press the 12 heat-set inserts in with a soldering iron (about 220 °C for PLA, 245 °C for PETG): 4 in the bezel corner bosses, 4 in the bottom-plate posts and 4 in the base's plate bosses.

1. **Sensors into the bezel.** Lay the front bezel face-down. Press the **LD2410C (3)** and **BH1750 (4)** into their cradles in the sensor bay, components facing the front. The BH1750's sensor chip should line up with the small window. A dab of hot glue on the cradle edge holds them. Plug jumper wires onto their pins, leading out through the gap at the bottom of each cradle.
2. **Display in.** Lay the **Sunton module (2)** screen-down into the bezel pocket. Connect the three 1.25 mm cables to its P1, P3 and P4 headers.
3. **SHT40 into the shell.** Press the **SHT40 (5)** into its cradle inside the **back shell (6)**, sensor facing the vents, and plug in its jumper wires.
4. **Close the display.** Route all cables (sensor jumpers + 3 JST cables) through the cable exit at the bottom of the back shell. Fit the shell onto the bezel and fix it with **4 × M3×16 (7)**.
5. **PCB onto the plate.** Screw the **add-on PCB (9)** onto the **bottom plate (11)** posts with **4 × M3×6 (10)**, USB-C edge towards the plate's back edge.
6. **Wire up.** Feed the cables from the display through the slot-floor tunnel into the base cavity, then plug them in:

   | Cable from | Plugs into on add-on PCB |
   |---|---|
   | Sunton P1 (UART + 5 V) | J2 |
   | Sunton P3 (I2C) | J3 |
   | Sunton P4 (buzzer, LD OUT, 3.3 V) | J4 |
   | SHT40 jumpers | J5 |
   | BH1750 jumpers | J6 |
   | LD2410C jumpers | J7 |

7. **Close the base.** Drop the display stack into the **base (8)** slot. Fit the bottom plate into the base cavity, USB-C lined up with the back hole, and fix it with **4 × M3×6 (12)** from underneath. Stick on the rubber feet.
8. **Power up** with a USB-C 5 V / 2 A supply.

## Why components sit where they do

| Component | Location | Reason |
|---|---|---|
| LD2410C radar | Top bezel, facing forward | Needs a clear view of whoever is in front of the screen; works through plastic, so no hole needed |
| BH1750 light | Top bezel, behind a window | Must see the room's light, not the backlight, so it sits above the screen |
| SHT40 temp/humidity | Back shell, at its own vents, away from the screen | The LCD backlight warms the case, so the sensor reads room air through the vents |
| Add-on PCB | In the base | Keeps weight low (stops tipping) and puts USB-C at the back for a neat cable |

## Files

| File | What |
|---|---|
| `homedeck_front_bezel / back_shell / base / bottom_plate .step/.stl` | Printable parts |
| `homedeck_exploded_assembly.step` | Coloured exploded assembly (open in Fusion 360) |
| `homedeck_assembled_assembly.step` | Coloured assembled model |
| `images/homedeck-exploded.png` | Labelled exploded view |
| `images/homedeck-exploded-rear.png` | Exploded view from behind |
| `images/homedeck-assembled.png` | Assembled render |
| `images/homedeck-drawing.png` | Dimensioned drawing |
| `enclosure.py`, `exploded.py` | Parametric source (CadQuery). Edit the numbers at the top of `enclosure.py` and re-run |

> Display dimensions are placeholders until the real Sunton board is measured. Everything is parametric, so the exploded view and drawing update automatically.
