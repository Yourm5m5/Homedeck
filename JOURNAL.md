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
- [ ] Start the ESPHome firmware + dashboard UI - I will start the code once I have the PCB 

## Day 3 – 13/10/2026: Firmware (ESPHome + LVGL)
Time spent: ~5 hours

What I did
- Wrote the HomeDeck firmware as one ESPHome YAML file
- Set up the Sunton board's 7" RGB display (800x480) and GT911 touchscreen
- Built 4 dashboard pages with LVGL and a nav bar at the bottom:
  - Home: big clock, date, outdoor weather, room temp and humidity
  - Rooms: tap tiles to toggle lights and the fan; tiles turn yellow when on
    and stay in sync with Home Assistant
  - Climate: drag a dial to set the air-con temperature
  - Media: now playing, prev / play-pause / next, volume slider
- Added my add-on PCB sensors:
  - SHT40: room temperature and humidity, sent to Home Assistant
  - BH1750: auto-brightness (dim at night, bright in daylight)
  - LD2410C: screen wakes when someone walks up, turns off 45 s after they leave
  - Buzzer: click on button presses, plus a "play_tone" action for doorbell alerts

Decisions
- ESPHome instead of Arduino: it connects to Home Assistant natively and the
  LVGL UI can be written in YAML
- Used the LD2410C's OUT pin (GPIO17) to wake the screen because it's instant;
  the UART data still goes to Home Assistant as a presence sensor
- Turned off serial logging because the LD2410C uses UART0 (GPIO43/44), so
  updates happen over Wi-Fi (OTA) after the first flash

Problems / what I learned
- The display needs exact RGB timing values (porches, pulse widths, pixel clock)
- Learned how LVGL pages, styles and flex layouts work
- Had to allow the device to perform Home Assistant actions before the buttons worked

<img width="891" height="951" alt="image" src="https://github.com/user-attachments/assets/1a22bd5d-e5cb-40cf-a8e2-63839750e590" />


Still to test (once parts arrive)
- Display timings on the real board
- LD2410C sensitivity and the 45 s sleep timer
- Auto-brightness range

## Day 4 – 18/10/2026: Review fixes: firmware docs + exploded view
**Time spent:** ~7 hours

### Reviewer feedback
HomeDeck was sent back with two things to fix:
1. The firmware needed better documentation, especially how the **sensors** work
   with the code and how the **LVGL UI** is set up
2. The enclosure needed an **exploded view** showing how the screen, add-on PCB,
   sensors and other parts fit together

---

### Part 1: Firmware documentation

**What I did**
- Commented every section of `homedeck.yaml`: pin map, I2C addresses, why each
  setting is used, and how each UI page works
- Wrote `firmware/FIRMWARE.md`: a full guide with diagrams covering the sensors,
  the screen sleep/wake logic and how the UI talks to Home Assistant
- Improvements while documenting: smoothing filters on the sensors, settings I
  can change from Home Assistant (screen timeout, minimum brightness, button
  sounds), volume slider sync, and the first tap on a sleeping screen no longer
  presses a button by accident

#### Sensor 1: SHT40 (temperature + humidity)
**What it does:** measures the room's temperature and humidity, shown on the
Home page and sent to Home Assistant.

**How it's read:** over I2C (address 0x44) every 30 s. It sits behind its own
vents in the back shell so the screen's heat doesn't affect it.

```yaml
- platform: sht4x
  update_interval: 30s
  heater_max_duty: 0.0          # built-in heater off, it would raise the reading
  temperature:
    id: room_temp
    filters:
      - offset: -1.5            # corrects warmth from inside the case
      - sliding_window_moving_average:
          window_size: 4        # smooths out small jumps
    on_value:
      - script.execute: update_room_label   # updates the Home page text
```

**How it's used:** every new reading writes "Room 22.4°C · 48%" on the Home page.

#### Sensor 2: BH1750 (ambient light)
**What it does:** measures how bright the room is so the screen dims at night
and brightens in daylight.

**How it's read:** I2C (address 0x23) every 2 s, averaged over 5 readings so a
passing shadow doesn't make the screen flicker.

```yaml
- platform: bh1750
  id: room_lux
  update_interval: 2s
  filters:
    - sliding_window_moving_average:
        window_size: 5
  on_value:
    - script.execute: auto_brightness
```

**How it's used:** the `auto_brightness` script turns lux into a backlight level:

```cpp
// brightness = minimum + lux / 400, kept between minimum and 100 %
float lo = id(min_brightness).state / 100.0f;   // e.g. 0.15
float b  = lo + id(room_lux).state / 400.0f;
id(target_brightness) = std::max(lo, std::min(1.0f, b));
```

| Room | Lux | Screen |
|---|---|---|
| Dark | 0 | 15 % |
| Lamp | 100 | 40 % |
| Daylight | 400+ | 100 % |

#### Sensor 3: LD2410C (mmWave presence radar)
**What it does:** detects when someone is in front of HomeDeck, even sitting
still, so the screen wakes when you walk up and turns off when you leave.

**How it's read:** two ways from the same sensor:
- **OUT pin (GPIO17):** a simple HIGH/LOW wire, instant, used to wake the screen
- **UART (256000 baud):** detailed data (presence + distance) for Home Assistant

```yaml
- platform: gpio
  id: presence_out
  pin: GPIO17
  on_press:
    - script.execute: wake_screen   # someone arrived → screen on
  on_release:
    - script.execute: idle_timer    # they left → start the sleep countdown
```

**How it's used:** the screen only turns off if the radar confirms nobody is
there, so it doesn't go dark while someone is reading it:

```yaml
- id: idle_timer
  mode: restart                     # any new activity restarts the countdown
  then:
    - delay: !lambda 'return id(screen_timeout).state * 1000;'
    - if:
        condition:
          binary_sensor.is_off: presence_out   # only sleep if the room is empty
        then:
          - light.turn_off: backlight
          - lvgl.pause:                         # stop redrawing to save CPU
```

#### How the LVGL UI is set up
**Layout:** 4 pages (Home, Rooms, Climate, Media) plus a nav bar along the bottom.
The nav bar is on LVGL's `top_layer`, so it shows on every page but is only
written once:

```yaml
top_layer:
  widgets:
    - obj:                          # bar along the bottom
        align: BOTTOM_MID
        layout:
          type: flex                # buttons space themselves out evenly
          flex_flow: row
        widgets:
          - button:
              on_click:
                - lvgl.page.show: page_rooms   # switch page
              widgets:
                - label: { text: "Rooms" }
```

**Buttons (Room tiles):** each tile is a *checkable* button. Checked = light on
(tile turns yellow). Tapping it asks Home Assistant to toggle the light:

```yaml
- button:
    id: btn_lounge
    checkable: true
    checked:
      bg_color: 0xF6B73C            # yellow when the light is on
    on_click:
      - homeassistant.action:
          action: light.toggle
          data:
            entity_id: light.lounge
```

The tile only changes colour when Home Assistant **confirms** the light's state,
so the screen never shows the wrong state, and it also updates if someone uses
the wall switch:

```yaml
- platform: homeassistant          # listens to the real light in HA
  entity_id: light.lounge
  on_state:
    - lvgl.widget.update:
        id: btn_lounge
        state:
          checked: !lambda 'return x;'   # x = true if the light is on
```

**Arc (air-con dial) and slider (volume):** the number updates live while you
drag (`on_value`), but the command is only sent once when you let go
(`on_release`), so dragging from 20 °C to 25 °C sends 1 command, not 6:

```yaml
- arc:
    id: arc_ac
    min_value: 16
    max_value: 30
    on_value:                       # while dragging → update the number
      - lvgl.label.update:
          id: lbl_ac_set
          text: { format: "%d°C", args: ['x'] }
    on_release:                     # let go → send to Home Assistant once
      - homeassistant.action:
          action: climate.set_temperature
          data:
            entity_id: climate.living_room
            temperature: !lambda 'return x;'
```

**Labels (clock):** updated every 10 seconds by a timer:

```yaml
interval:
  - interval: 10s
    then:
      - lvgl.label.update:
          id: lbl_time
          text: !lambda 'return id(ha_time).now().strftime("%H:%M");'
```

#### How it all fits together
```
Sensors ──I2C / UART / GPIO──▶ ESPHome ──▶ Home Assistant (logs, automations)
                                  │
                                  ├──▶ LVGL labels (room temp, brightness)
                                  └──▶ wake / sleep logic
Touch on screen ──▶ LVGL button / arc / slider ──▶ Home Assistant action
Home Assistant state ──▶ LVGL widget update (tile colour, dial, slider)
```

---

### Part 2: Exploded view (Fusion 360)

**What I did**
- Modelled simple versions of the parts that had no CAD model (see Problems)
- Inserted everything into one Fusion design and positioned the parts with
  **joints** (display into the bezel, sensors into their cradles, PCB onto
  the bottom-plate posts, screws into their holes)
- Ran **Inspect → Interference** to check nothing overlapped
- Used the **Animation workspace → Explode** to pull the parts apart, tidied
  their positions, turned on trail lines and captured the image

**How it fits together**
```
DISPLAY STACK (front → back)            BASE STACK (top → bottom)
1  Front bezel                          8  Weighted base
3  LD2410C ┐ sensor bay behind          10 4× M3×6 screws
4  BH1750  ┘ the top bezel              9  Add-on PCB
2  Sunton display module                11 Bottom plate
5  SHT40 (back shell cradle)            12 4× M3×6 screws
6  Back shell
7  4× M3×16 screws
```
Cables run from the back shell's cable exit, through the tunnel in the base
slot, into the base cavity where they plug into the add-on PCB.

Also wrote `cad/ASSEMBLY.md` with a numbered parts list, step-by-step assembly
and a table of which cable plugs into which socket.

<img width="722" height="596" alt="image" src="https://github.com/user-attachments/assets/1f1f9d6b-40e4-40a6-81f4-722384bd0cdf" />
<img width="704" height="529" alt="image" src="https://github.com/user-attachments/assets/088708ca-a316-4f43-9f56-e7bcdf5b32e7" />
<img width="869" height="499" alt="image" src="https://github.com/user-attachments/assets/08472d9b-a660-45bc-a7f2-d938e6f57184" />
<img width="444" height="495" alt="image" src="https://github.com/user-attachments/assets/cc386595-c1fe-4ea0-9ce7-b8babd9cb91b" />

---

### Problems

**1. No CAD models for most components**
- To make the exploded view I needed 3D models of every part inside HomeDeck,
  but most of them didn't have one available online
- **Fix:** I measured each part [from the seller's drawings
  and datasheets] and modeled simple versions myself in Fusion 360: the outline,
  thickness, mounting holes and the important features (screen area on the
  display, sensor chip on the BH1750). I kept them accurate where it matters for
  fit: overall size, thickness, hole positions and connector locations.

**2. The sensors had nowhere to mount**
- When I put the assembly together, I found the LD2410C and BH1750 had no
  space inside the enclosure: the top of the front bezel was solid plastic
- **Fix:** added a **sensor bay** behind the top bezel with cradles that hold
  both sensors against the front wall. The BH1750 lines up with a small
  10 × 5 mm window, and the radar sees through the 2 mm plastic. This made the
  bezel taller (116 → 128 mm).
- Also added a cradle for the **SHT40** in the back shell with its own vents,
  so it reads room air instead of heat from the screen
- The sensors connect to the add-on PCB's sockets (J5, J6, J7) with jumper wires

### What I learned
- An exploded view is worth making for more than the reviewers: it found a
  real design problem (the sensors had nowhere to go) before I printed anything
- How to use joints and the Animation workspace in Fusion 360
- Lots of cheap modules from AliExpress don't have official CAD models, but a
  simple, accurately sized model is enough to check fit
- Writing documentation made me find things to improve in my own firmware

### Files added / updated
- `firmware/homedeck.yaml`: fully commented
- `firmware/FIRMWARE.md`: firmware guide

### Next steps
- [ ] Resubmit HomeDeck for review
- [ ] Check my simple component models against the real parts when they arrive
- [ ] Order the PCB and parts

*Note: AI (Claude) helped with the firmware, documentation and reference CAD files.*
