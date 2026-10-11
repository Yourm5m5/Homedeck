# HomePanel Journal

A 7" wall-mounted Home Assistant touchscreen that shows my home at a glance and wakes when I walk up.

---

## Day 1 – 11 Oct 2026
**Time spent:** ~2 hours

### What I did
- Brainstormed project ideas for Hack Club Forge 
- Picked a **Home Assistant touchscreen panel** because it can both control my home *and* show info
- e.g. ![Uploading image.png…]()

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

