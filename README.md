# Claude Pad 🎹

> A custom, 3D-printed companion macropad for **Claude Code** — featuring a built-in OLED screen with animated expressions, a capacitive touch sensor, haptic buzzer feedback, and BLE HID keyboard output over NimBLE.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔵 BLE HID Keyboard | Connects wirelessly to macOS / Windows / Linux as a native keyboard |
| 🖥️ 128×64 OLED Display | Animated face reacts to every keypress — idle, plan, mic, escape, rocket, sleep |
| 🔊 Buzzer Feedback | Distinct tones for each action via PWM-driven passive buzzer |
| 🤏 Capacitive Touch | Touch the top of the pad to instantly open Claude (Cmd+Shift+)) |
| 😴 Auto-Sleep | Display dims to a sleeping animation after 60 s of inactivity |
| 🔄 Wake Animation | Stretching/yawning animation triggered on first button press after sleep |
| ♻️ Chord Keys | Multi-button combos unlock extra shortcuts (navigate history, dictation, etc.) |

---

## 📦 Firmware Versions

Two production-ready sketches are included:

### `ble_keyboard_3btn.ino` — **3-Button Layout** (minimal build)
Designed for a compact physical layout with the three core Claude Code actions plus navigation:

| Button | GPIO | Action |
|---|---|---|
| **PLAN** | 0 | `Shift + Tab` × 2 — opens Claude's plan mode |
| **ESC** | 1 | `Escape` — cancels current Claude action |
| **ACCEPT** | 2 | `Enter` — accepts Claude's suggestion |
| **MIC** | 3 | Hold → `Space` (hold-to-repeat) — voice dictation trigger |
| **DOWN** | 4 | `↓` Arrow Down |
| **UP** | 21 | `↑` Arrow Up |

> The "3-button" label refers to the **three primary Claude actions** (Plan / ESC / Accept). The Up/Down are navigation additions that still fit in a compact form factor.

---

### `claude-6btn-keyboard.ino` — **6-Button Layout** (full build, recommended)
Full 6-key layout — identical core logic with all 6 buttons fully mapped for a premium experience:

| Button | GPIO | Action |
|---|---|---|
| **PLAN** | 0 | `Shift + Tab` × 2 — opens plan mode |
| **ESC** | 1 | `Escape` — cancels |
| **ACCEPT** | 2 | `Enter` — accepts |
| **MIC** | 3 | Hold → `Space` (hold-to-repeat) |
| **DOWN** | 4 | `↓` Arrow Down |
| **UP** | 21 | `↑` Arrow Up |

#### Chord shortcuts (both versions)
| Combo | Action |
|---|---|
| MIC + ACCEPT | `Cmd + C` (copy) |
| ACCEPT + UP | `Cmd + [` (navigate backward) |
| ACCEPT + DOWN | `Cmd + ]` (navigate forward) |
| MIC + DOWN | `F5` (Mac dictation trigger) |
| Touch sensor (tap) | `Cmd + Shift + )` — open Claude window |

---

## 🔌 Hardware & Wiring

### Bill of Materials

| Component | Notes |
|---|---|
| **ESP32-S3 Super Mini** | Main MCU — BLE + USB-C |
| **SSD1306 OLED 128×64** | I2C, 0.96" or 1.3" |
| **Passive Buzzer** | PWM-driven via `ledc` |
| **Capacitive Touch Sensor** | TTP223 or similar, active-HIGH |
| **6× Tactile Push Buttons** | Momentary, normally-open, pulled HIGH internally |
| **3D-Printed Case** | See `/case` (STL files — coming soon) |

---

### Pin Connections

#### OLED Display (I2C)

| OLED Pin | ESP32-S3 GPIO |
|---|---|
| SDA | **GPIO 8** |
| SCL | **GPIO 9** |
| VCC | 3.3 V |
| GND | GND |

#### Buttons (active LOW, internal pull-up)

| Button Label | ESP32-S3 GPIO |
|---|---|
| PLAN | **GPIO 0** |
| ESC | **GPIO 1** |
| ACCEPT | **GPIO 2** |
| MIC | **GPIO 3** |
| DOWN | **GPIO 4** |
| UP | **GPIO 21** |

> Connect each button between the GPIO pin and **GND**. Internal `INPUT_PULLUP` resistors are enabled — no external resistors needed.

#### Buzzer / Speaker

| Buzzer Pin | ESP32-S3 GPIO |
|---|---|
| + (positive) | **GPIO 7** |
| − (negative) | GND |

#### Capacitive Touch Sensor (TTP223 or equivalent)

| Sensor Pin | ESP32-S3 GPIO |
|---|---|
| SIG / OUT | **GPIO 20** |
| VCC | 3.3 V |
| GND | GND |

> The firmware uses `INPUT_PULLDOWN` and expects an active-HIGH signal from the sensor.

#### Onboard LED

| Function | GPIO |
|---|---|
| Status LED | **GPIO 47** (active LOW) |

> LED blinks while BLE is advertising / disconnected; stays solid when connected.

---

## 🛠️ Setup & Flash

### Requirements

- [Arduino IDE](https://www.arduino.cc/en/software) ≥ 2.x **or** VS Code + Arduino extension
- **ESP32 board package** by Espressif (≥ 3.x recommended, ≥ 2.x also supported via `#if` guards in code)
- Install via Board Manager URL: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`

### Libraries

Install via **Library Manager** (Sketch → Include Library → Manage Libraries):

| Library | Version |
|---|---|
| [NimBLE-Arduino](https://github.com/h2zero/NimBLE-Arduino) | ≥ 1.4 |
| [U8g2](https://github.com/olikraus/u8g2) | ≥ 2.35 |

### Flash Steps

1. Select board: **ESP32S3 Dev Module** (or "ESP32-S3 Super Mini")
2. Set **USB CDC On Boot** → `Enabled`
3. Set **PSRAM** → `Disabled`
4. Open `claude-6btn-keyboard.ino` (6-button, recommended) or `ble_keyboard_3btn.ino` (minimal)
5. Upload via USB-C
6. Open Serial Monitor at **115200 baud** to watch BLE advertising status
7. Pair from your computer's Bluetooth settings — device name: **"Claude Keyboard"**

---

## 🎨 OLED Animations

The display runs at 50 FPS and shows contextual animated faces:

| State | Trigger |
|---|---|
| 😐 `IDLE` | No button held |
| 🔼 `LOOK_UP` | UP button held |
| 🔽 `LOOK_DOWN` | DOWN button held |
| 🚀 `ROCKET` | ACCEPT pressed (Enter sent) |
| 🎙️ `MIC` | MIC button held |
| 📋 `PLAN` | PLAN button held |
| 💣 `ESC` | ESC pressed |
| 😪 `YAWN` | Transition to sleep |
| 💤 `SLEEP` | Idle > 60 seconds |
| 🤸 `WAKEUP` | First press after sleep / touch sensor tap |

---

## 📁 Repository Structure

```
claude-pad/
├── ble_keyboard_3btn.ino        # 3-button minimal firmware
├── claude-6btn-keyboard.ino     # 6-button full firmware (recommended)
├── 3d-models/
│   └── claude_keyboard.3mf      # 3MF print file for the enclosure
└── README.md
```

### Printing the Case

Open `3d-models/claude_keyboard.3mf` in [Bambu Studio](https://bambulab.com/en/download/studio), PrusaSlicer, or any slicer that supports `.3mf`.

Suggested settings:
- Layer height: **0.2 mm**
- Infill: **15–20%** (walls carry the load, not infill)
- Supports: **None required** (designed to print flat-side-down)
- Material: **PLA** or **PETG**

---

## 🤝 Contributing

Pull requests are welcome! Ideas for improvement:

- [x] 3MF file for 3D-printed case (`3d-models/claude_keyboard.3mf`)
- [ ] Windows shortcut support (currently tuned for macOS)
- [ ] Configurable keymap via Serial / BLE
- [ ] Battery + USB charging support

---

## 📜 License

MIT License — free to use, modify, and share. If you build one, share a photo! 🙌

---

## 🙏 Credits

Built with ❤️ using:
- [NimBLE-Arduino](https://github.com/h2zero/NimBLE-Arduino) — lightweight BLE stack for ESP32
- [U8g2](https://github.com/olikraus/u8g2) — universal 8-bit graphics library
- Espressif ESP32 Arduino core
