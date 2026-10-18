# HomeDeck

**A 7" desk touchscreen for Home Assistant that shows your home at a glance and wakes up when you walk over.**
<img width="1200" height="800" alt="image" src="https://github.com/user-attachments/assets/86897348-447b-48dc-ac6a-51900d919659" />


What is HomeDeck?

HomeDeck is a 7" smart display that sits on a desk or bench and connects to Home Assistant over Wi-Fi. At a glance it shows the time, weather, calendar, energy use, and whether doors or windows are open. One tap controls lights, fans, air-con, scenes and music.

It's built around a Sunton ESP32-S3 7" touchscreen board, plus a custom add-on PCB I designed in KiCad. The add-on board provides protected USB-C power and three sensors:

LD2410C mmWave radar notices when someone is in front of the panel (even sitting still) and wakes the screen
BH1750 light sensor dims the screen automatically in a dark room
SHT40 reads room temperature and humidity and reports them back to Home Assistant

A small buzzer gives touch feedback and alerts. Everything sits in a 3D-printed enclosure I modelled in Fusion 360, tilted back 20° so it's easy to read and tap from a chair.

Why I built it

I use Home Assistant at home, but controlling anything means pulling out my phone, unlocking it and opening an app, which is often slower than walking to the light switch. I wanted something that's always on and always in the same place: glance to see what's going on, tap once to change it.

You can buy smart displays, but most are locked to one company's ecosystem, rely on the cloud, and can't be customized. HomeDeck runs locally through ESPHome and Home Assistant, so it keeps working without the internet and shows exactly what I want.

I also wanted a project that would teach me real hardware skills: drawing a schematic, routing a two-layer PCB, modelling a multi-part enclosure and writing firmware, rather than just plugging modules together.

Features
7" 800×480 capacitive touchscreen
Controls lights, fans, AC, scenes and media through Home Assistant
Shows time, weather, calendar, energy use and alerts
Wakes automatically when someone walks up (mmWave presence)
Auto-brightness from the ambient light sensor
Reports room temperature and humidity to Home Assistant
Buzzer for touch feedback, timers and doorbell alerts
Settings adjustable from Home Assistant (screen timeout, minimum brightness, sounds)
Fully local: ESPHome + LVGL, no cloud needed
One USB-C cable for power
How it works

The Sunton board's ESP32-S3 drives the display and runs ESPHome with an LVGL interface. The display uses almost every GPIO, so my add-on PCB connects through the few left on the board's expansion headers:

Signal	Part	ESP32 pin	Header
I2C SDA / SCL	SHT40 + BH1750	GPIO19 / GPIO20	P3 (shared with touch, no address conflicts)
UART RX / TX	LD2410C	GPIO44 / GPIO43	P1
Presence output	LD2410C	GPIO17	P4
Buzzer (via NPN transistor)	Piezo	GPIO18	P4
5 V power in	USB-C	n/a	P1
I2C
I2C
UART and OUT pin
PWM
Wi-Fi
SHT40
ESP32-S3 running ESPHome
BH1750
LD2410C
Buzzer
LVGL touch UI
Home Assistant
Firmware

The firmware is a single ESPHome file, firmware/homedeck.yaml, commented section by section. The full explanation with diagrams is in firmware/FIRMWARE.md.

Sensors
Sensor	What it does	How it's read	How it's used
SHT40	Room temperature + humidity	I2C 0x44, every 30 s, offset for case warmth + 2-min average	Shown on the Home page, sent to Home Assistant
BH1750	Room brightness (lux)	I2C 0x23, every 2 s, 10-s average	Auto-brightness: brightness = min + lux / 400
LD2410C	Is someone in front of the screen?	OUT pin (GPIO17) for instant wake; UART for presence + distance	Wakes the screen; it only sleeps once the radar says the room is empty

Example: the light sensor triggers auto-brightness on every new reading.

yaml
- platform: bh1750
  id: room_lux
  update_interval: 2s
  filters:
    - sliding_window_moving_average:
        window_size: 5            # 10 s average, so shadows don't cause flicker
  on_value:
    - script.execute: auto_brightness
LVGL UI

4 pages (Home, Rooms, Climate, Media) with a nav bar on LVGL's top_layer, so it's always visible.

Element	Widget	Touch → Home Assistant	Home Assistant → screen
Room tiles	Checkable button	light.toggle / fan.toggle	Tile turns yellow when HA confirms it's on
Air-con dial	arc (16–30 °C)	climate.set_temperature on release	Dial follows the set-point
Volume	slider	media_player.volume_set on release	Slider follows the volume
Clock, weather, room	label	n/a	Updated by timer / HA / SHT40

Tiles only change colour when Home Assistant confirms the new state, so the screen never shows the wrong state, and it updates if someone uses a wall switch:

yaml
- platform: homeassistant          # listens to the real light
  entity_id: light.lounge
  on_state:
    - lvgl.widget.update:
        id: btn_lounge
        state:
          checked: !lambda 'return x;'
Custom PCB

Designed in KiCad: a ~60 × 45 mm two-layer board with power traces on the front, signals on the back, ground fill on both sides and stitching vias. It passes DRC with 0 errors.

Power input: USB-C (5.1 kΩ CC resistors) → 2 A polyfuse → SMAJ5.0CA TVS diode → 10 µF + 100 nF decoupling.

<img width="751" height="582" alt="Schematic" src="https://github.com/user-attachments/assets/cf73ffb1-b691-4b8e-9d77-4d6d3f266f41" /> <img width="926" height="642" alt="PCB" src="https://github.com/user-attachments/assets/1746f53a-ca06-4d8a-ae76-c0612a7187e0" />
Enclosure

Modelled in Fusion 360 as four 3D-printed parts:

Front bezel holds the screen behind a 2 mm lip, with a sensor bay behind the top strip for the LD2410C and BH1750 (light window above the screen)
Back shell has vent slots, plus a cradle so the SHT40 reads room air through its own vents instead of screen heat
Base (190 × 120 × 24 mm) holds the display at 20°, with space for the PCB inside
Bottom plate mounts the add-on PCB on standoffs and has rubber feet underneath
Exploded view

Shows how the screen, sensors, add-on PCB, printed parts and screws fit together. The full parts list, step-by-step assembly and cable wiring table are in cad/ASSEMBLY.md.

Show Image

Show Image

Renders
<img width="1187" height="650" alt="Enclosure" src="https://github.com/user-attachments/assets/22c62446-e304-473a-a2d2-8882ae5dc1ef" /> <img width="1029" height="582" alt="Enclosure" src="https://github.com/user-attachments/assets/7f7fa2cd-6b7c-4f8f-86b6-ad2c9f27993c" /> <img width="919" height="752" alt="Enclosure" src="https://github.com/user-attachments/assets/35ecc35f-866f-48b0-a4f9-901a6cc68d96" /> <img width="1094" height="670" alt="Enclosure" src="https://github.com/user-attachments/assets/2a7e2eb2-e8d9-4431-ae22-f776afd30ea0" />

Screen-related sizes are estimates and will be checked against the real Sunton board before the final print.

Bill of materials

All parts from AliExpress except the custom PCB, which comes from JLCPCB. Prices are approximate AUD and change with sales, exchange rates and shipping, so check the listing before ordering. Many small parts only come in packs, so you'll have spares.

Electronics
#	Part	Qty	Approx. AUD
1	Sunton ESP32-8048S070C (7" 800×480, ESP32-S3, capacitive touch)	1	$70.00
2	HLK-LD2410C 24 GHz mmWave presence sensor	1	$6.00
3	SHT40 temperature/humidity breakout module	1	$4.50
4	BH1750 (GY-302) light sensor module	1	$3.00
5	Passive piezo buzzer 12 mm (pack of 10)	1 pack	$3.00
6	MMBT3904 NPN transistor, SOT-23 (pack of 100)	1 pack	$2.50
7	USB-C 6-pin power-only socket (pack of 10)	1 pack	$3.00
8	2 A polyfuse, 1812 (pack of 20)	1 pack	$3.00
9	SMAJ5.0CA TVS diode (pack of 50)	1 pack	$3.00
10	0805 resistor + capacitor sample kit	1 kit	$8.00
11	1.25 mm 4-pin connectors + pre-crimped cables	1 set	$5.00
12	2.54 mm female header strips	1 pack	$2.50
13	Female–female jumper wires, 20–30 cm (sensors → PCB)	1 pack	$3.00
14	Custom PCB, 5 pcs + shipping (JLCPCB)	1 order	$18.00
	Electronics subtotal		$134.50
Enclosure
#	Part	Qty	Approx. AUD
15	PETG filament (~300 g of a 1 kg roll)	1	$8.00
16	M3 brass heat-set inserts (need 12, pack of 50)	1 pack	$7.00
17	M3 screw assortment (4× M3×16, 8× M3×6)	1 kit	$8.00
18	Self-adhesive rubber feet	1 pack	$3.00
	Enclosure subtotal		$26.00
Power
#	Part	Qty	Approx. AUD
19	USB-C power supply, 5 V / 3 A	1	$10.00
20	USB-C cable, 1 m	1	$4.00
	Power subtotal		$14.00
Total
	AUD
Electronics	$134.50
Enclosure	$26.00
Power	$14.00
Estimated total	≈ $174.50
Repository layout
homedeck/
├── README.md
├── JOURNAL.md               ← build log, day by day
├── firmware/
│   ├── homedeck.yaml        ← ESPHome firmware (fully commented)
│   └── FIRMWARE.md          ← how the sensors + UI work
├── pcb/                     ← KiCad project + Gerbers
└── cad/
    ├── ASSEMBLY.md          ← parts list, assembly steps, wiring
    ├── *.step / *.stl       ← printable parts + assemblies
    
The first review asked for better firmware documentation and an exploded view. What changed:

Firmware: commented every section of homedeck.yaml and wrote FIRMWARE.md, explaining each sensor (what it does, how it's read, how it's used) and how every LVGL element is wired to Home Assistant, with code snippets and diagrams
Exploded view: built a full assembly in Fusion 360 with every component and screw, plus ASSEMBLY.md
Design fix found along the way: the exploded view showed the radar and light sensor had nowhere to mount, so I added a sensor bay behind the top bezel (bezel 116 → 128 mm) and an SHT40 cradle with its own vents in the back shell
Status
 Concept, parts selection and block diagram
 Schematic + PCB (KiCad), DRC clean, Gerbers exported
 Enclosure CAD + exploded view (Fusion 360)
 ESPHome firmware + documentation
 Order parts + PCB
 Assemble and test
 Print the enclosure, final assembly and demo video
Acknowledgements
Sunton pinout info from ESP3D and Blakadder's templates
Built with KiCad, Fusion 360, ESPHome, LVGL and Home Assistant
AI assistance (Claude) was used for planning, guidance, diagrams, the firmware and documentation, and a reference enclosure model
