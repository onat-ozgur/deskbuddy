# 🤖 DeskBuddy — ESP32-C3 Multi-Functional Desk Companion

**DeskBuddy** is a portable, battery-powered, modular desk companion built on an **ESP32-C3 Super Mini** microcontroller and a **0.96" SSD1306 OLED display**. The system combines a state machine for mode switching, procedural animations, digital clock/widget screens, and interactive mini-game infrastructure into a single lightweight embedded firmware.

---

## ⚙️ System Architecture & Modules

DeskBuddy uses a modular architecture driven by button interactions to switch between different modes:

### 1. 🎏 Zen Koi Pond Mode
* **Mechanism:** A procedural fish simulation powered by a 12-segment inverse kinematics (IK) spine physics engine.
* **Physics & UI:** Joint angle constraints (`maxJointAngle = 0.20f`) prevent unnatural spine folding or tail collisions. Procedural Matsuba (herringbone) scale patterns and velocity-governed fin folding are calculated in real time. Tightened screen boundaries (`-8` to `136` X, `-6` to `70` Y) keep fish naturally framed on screen. Features zero UI clutter or text overlays for a distraction-free aesthetic.
* **Environment:** Includes 2 procedurally spawned floating lily pads (`LILY_COUNT = 2`) on every boot and button-triggered feeding/water displacement ripple mechanics.

### 2. ⏰ Clock & System Monitor Mode
* **Mechanism:** A minimalist desktop widget displaying digital time, device uptime, and system status metrics.

### 3. 🎮 Mini-Game Engine
* **Mechanism:** A pixel-art, reaction-driven micro-game infrastructure controlled via a single button.

---

## 🛠️ Hardware Requirements

| Component | Specification | Quantity |
| :--- | :--- | :--- |
| **MCU** | ESP32-C3 Super Mini | 1 |
| **Display** | 0.96" I2C SSD1306 OLED (128x64) | 1 |
| **Charger** | TP4056 Type-C / Micro-USB Module | 1 |
| **Power Source** | 3.7V Li-Ion / LiPo Battery | 1 |
| **Switches & Input** | Push Button (Control) & Slide Switch (Power) | 1 each |

---

## 🔌 Circuit Diagram & Wiring

The TP4056 module is used with its default current-setting resistor (1.2kΩ / 1A charging rate). The voltage coming from the TP4056 output during operation/charging is safely stepped down to 3.3V by the ESP32-C3's onboard LDO regulator.

```text
       +---------------------------------------+
       |         3.7V Li-Ion / LiPo Battery    |
       +-------------------+---------------+---+
                           |               |
                          (+)             (-)
                           |               |
                           v               v
                   +---------------+---------------+
                   |  [BAT+]             [BAT-]    |
                   |                               |
                   |         TP4056 MODULE         |
                   |                               |
                   |  [OUT+]             [OUT-]    |
                   +-------+---------------+-------+
                           |               |
                     (Power Switch)        |
                           |               |
                           v               v
                   +---------------+---------------+
                   |  [5V / VEXT]        [GND]     |
                   |                               |
                   |        ESP32-C3 SUPER MINI    |
                   |                               |
                   |  [GPIO8/SDA]    [GPIO9/SCL]   |
                   +-------+---------------+-------+
                           |               |
                           v               v
                   +---------------+---------------+
                   |  [SDA]              [SCL]     |
                   |     0.96" I2C SSD1306 OLED    |
                   |  [VCC -> 3.3V]  [GND -> GND]  |
                   +-------------------------------+
```

### Pin Connection Table

| Module / Pin | Connected To | Description |
| :--- | :--- | :--- |
| **TP4056 BAT+ / BAT-** | Battery (+) / Battery (-) | Battery charging and protection |
| **TP4056 OUT+** | Slide Switch -> ESP32-C3 **5V** | Switched main system power |
| **TP4056 OUT-** | ESP32-C3 **GND** | Common ground |
| **OLED VCC / GND** | ESP32-C3 **3.3V / GND** | Display power supply |
| **OLED SDA / SCL** | ESP32-C3 **GPIO8 / GPIO9** | I2C communication bus |
| **Push Button** | ESP32-C3 **GPIO4** & **GND** | Mode switching / User interaction (Internal Pullup) |

---

## 🧠 Software Architecture & Button Logic

The system handles user interaction through a single multi-gesture button state machine:

* **Single Click:** Performs an in-app action (e.g., feeds fish in Koi Pond mode, jumps in Mini-Game).
* **Double Click:** Cycles through modes (Koi Pond ↔ Clock ↔ Mini-Game).
* **Long Press:** Resets the current state or returns to the primary view.

---

## 📂 Directory Structure

```text
DeskBuddy/
├── include/
│   ├── config.h          # Pin definitions, display, and global config
│   ├── button_handler.h  # Button state machine header
│   ├── koi_pond.h        # Koi simulation header
│   ├── clock_widget.h    # Clock and widget header
│   └── mini_game.h       # Mini-game engine header
├── src/
│   ├── main.cpp          # Main setup, loop, and mode switching
│   ├── button_handler.cpp# Click and gesture detection
│   ├── koi_pond.cpp      # Fish IK physics and pond renderer
│   ├── clock_widget.cpp  # Clock and UI rendering
│   └── mini_game.cpp     # Game logic and rendering
├── platformio.ini        # PlatformIO environment configuration
└── README.md             # Project documentation
```

---

## 🚀 Installation & Setup

### Option A: Using PlatformIO (Recommended)

1. Open the project in VS Code with the **PlatformIO IDE** extension installed.
2. Verify `platformio.ini` configuration:

```ini
[env:esp32-c3-devkitm-1]
platform = espressif32
board = esp32-c3-devkitm-1
framework = arduino
monitor_speed = 115200
board_build.mcu = esp32c3
lib_deps = 
	adafruit/Adafruit SSD1306@^2.5.7
	adafruit/Adafruit GFX Library@^1.11.5
```

3. Connect your board via USB-C and click **Upload**.

### Option B: Using Arduino IDE

1. Add the ESP32 board manager URL in preferences:
   `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
2. Install the `esp32` package via **Tools > Board > Boards Manager**.
3. Install required libraries via **Tools > Manage Libraries**:
   * `Adafruit SSD1306`
   * `Adafruit GFX Library`
4. Select **ESP32C3 Dev Module** under **Tools > Board**.
5. Set **USB CDC On Boot** to **Enabled**.
6. Compile and upload to your board.

---
