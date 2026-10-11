# HomePanel Journal

A 7" wall-mounted Home Assistant touchscreen that shows my home at a glance and wakes when I walk up.

---

## Day 1 – 11 Oct 2026
**Time spent:** ~6 hours

### What I did
- Brainstormed project ideas for Hack Club Forge 
- Picked a **Home Assistant touchscreen panel** because it can both control my home *and* show info. Example:
  <img width="1200" height="800" alt="image" src="https://github.com/user-attachments/assets/2bfd98c3-86c5-40f5-9f36-f890e5abd4cb" />


- First planned a 5" screen, then upgraded to **7"** for more room for controls. 7" is the biggest an ESP32-S3 can drive smoothly
- Chose the **Sunton ESP32-8048S070C** (7" 800×480, capacitive touch) as the main board
- Planned a custom add-on PCB with:
  - LD2410C mmWave sensor → wakes screen when someone walks up
  - SHT40 → temperature & humidity
  - BH1750 → auto screen brightness
  - Buzzer → touch feedback & alerts
  - USB-C power input with protection
- Made a full parts list (estimated total ~$120–185 AUD)
- Created the project on Forge and set up this repo

### Decisions
| Decision | Why |
|---|---|
| ESP32-S3 over Raspberry Pi | More of a real hardware project, lower power, instant boot |
| 7" over 5" | Fits a 3×3 grid of controls + clock/weather |
| ESPHome + LVGL | Connects natively to Home Assistant, no MQTT needed |

### Problems / things to check
- The display uses most of the ESP32's pins, so I need to check the Sunton pinout for free GPIOs for I2C + UART
- Need exact screen dimensions before designing the enclosure

### Next steps
- [ ] Find the Sunton board pinout
- [ ] Draw a block diagram of how everything connects
- [ ] Start the schematic in KiCad

### What I did after
- Drew the full schematic in KiCad: USB-C power input with polyfuse + TVS protection,
  connectors to the Sunton board, SHT40/BH1750/LD2410C sensor sockets, buzzer driver
- Assigned footprints (0805 passives, SOT-23 transistor, PicoBlade connectors)
- Laid out a ~60×45mm 2-layer PCB with 4 M3 mounting holes
- Routed power on the front layer (0.8mm) and signals on the back (0.3mm)
- Added GND fills on both layers + stitching vias
- Fixed all DRC errors → **0 errors**
- Exported Gerbers and got a JLCPCB quote

### Problems I solved
- Shield pad and a ground island weren't connected → added stitching vias
- Traces too wide for USB-C pads → used 0.4mm near the connector

### Images
<img width="751" height="582" alt="image" src="https://github.com/user-attachments/assets/affa4fef-e9bd-4bb2-88fc-ff88256ee825" />
<img width="926" height="642" alt="image" src="https://github.com/user-attachments/assets/6ac5a209-c20e-4dc8-af7a-8a767a69103a" />


### Next steps
- [ ] Design the enclosure
- [ ] Start the ESPHome firmware

## Day 2 – 12 Oct 2026: Enclosure CAD (Fusion 360)
**Time spent:** 10 hours (All-nighter. I was bored)

### What I did
- Designed the full HomeDeck desk-stand enclosure in Fusion 360
- Started from a concept sketch + a dimensioned reference drawing (made with AI help),
  then modelled every part myself in Fusion
- Set up **user parameters** for the screen size, so I can update everything
  once I measure the real Sunton board
- Modelled 4 parts:
  1. **Front bezel**: screen pocket with a 2 mm lip, 155 × 88 viewing window,
     26 × 6 sensor window for the light + mmWave sensors, M3 screw bosses
  2. **Back shell**: vent slots so the SHT40 reads room air instead of screen heat,
     cable exit at the bottom, counterbored screw holes
  3. **Base**: 190 × 120 × 24 mm with a slot that holds the display at a **20° tilt**,
     hollow underneath for the add-on PCB, cable tunnel, USB-C cutout at the back
  4. **Bottom plate**: 4 standoffs for the add-on PCB, screws into the base, rubber-foot recesses
- Put it all together in an assembly to check everything fits

### Decisions
| Decision | Why |
|---|---|
| Desk stand instead of wall mount | Easier to set up, no wall drilling, can move it around |
| 20° tilt | Comfortable to read and tap from a desk |
| PCB in the base | Keeps weight low so it doesn't tip when tapping the screen |
| Heat-set inserts for all screws | Stronger than screwing into plastic, can open it many times |
| Parametric design | Real screen sizes are still unknown, so this makes changes quick |

### Problems / what I learned
- [e.g. the tilted slot was tricky. Had to sketch on an angled plane]
- [e.g. learned how to use Shell, Fillet and Joints in Fusion]
- [anything that broke or you had to redo]

### Images
<img width="919" height="752" alt="image" src="https://github.com/user-attachments/assets/5c7cca06-68ca-4e6c-9f07-6cdbe8f743c0" />
<img width="1187" height="650" alt="image" src="https://github.com/user-attachments/assets/6abb21cc-d88c-4f9c-b70d-3d7821796fa4" />
<img width="1029" height="582" alt="image" src="https://github.com/user-attachments/assets/e065960d-c555-4947-9e3d-76a86c7e45ab" />
<img width="1094" height="670" alt="image" src="https://github.com/user-attachments/assets/cf1e5bd7-0ed8-48b5-9a09-4c50d9c59a1a" />


### Still to check
- Screen sizes are placeholders: confirm against the real Sunton board
- Line up the base's USB-C hole with J1 on the add-on PCB

### Next steps
- [ ] Start the ESPHome firmware + dashboard UI
- [ ] Finish the Forge submission write-up
