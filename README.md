# 🤖 Desk Buddy (Desk Companion)

An interactive, animated desktop companion powered by an **Arduino Nano** and a **0.96" SSD1306 OLED display**. Desk Buddy keeps you company with dynamic expressions, helps you stay productive with a Pomodoro timer, gives you motivational quotes, tracks time, and even helps you make decisions!

---

## ✨ Features

- **🙂 Dynamic Face Moods:** Features multiple animated expressions (Normal, Happy, Sleepy, Surprised, Love/Heart eyes, Wink) that automatically shift or can be toggled manually.
- **👋 Personalized Greetings:** Greets you upon boot and pops up periodic friendly reminders.
- **💬 Rotating Quotes:** Displays inspirational and witty quotes, automatically rotating or cycled manually.
- **🍅 Pomodoro Timer:** 25-minute focus session followed by a 5-minute break to boost productivity.
- **⏱️ Stopwatch:** Built-in timer with start, pause, and auto-reset functionality.
- **🔮 Magic Decider:** Ask a question, long-press the button, and receive a random decision!

---

## 🛠️ Hardware Requirements

| Component | Description |
|---|---|
| **Microcontroller** | Arduino Nano (or Uno / Pro Mini / compatible ATmega328P) |
| **Display** | 0.96" I2C SSD1306 128x64 OLED Display |
| **Input** | 1x Momentary Push Button |
| **Wiring** | Breadboard & Jumper wires (or custom enclosure/PCB) |

---

## 🔌 Wiring & Pinout

### 1. OLED Display (I2C)
| OLED Pin | Arduino Nano Pin | Note |
|---|---|---|
| **VCC** | 5V (or 3.3V) | Power |
| **GND** | GND | Ground |
| **SDA** | **A4** | I2C Data |
| **SCL** | **A5** | I2C Clock |

### 2. Push Button
| Button Leg | Arduino Nano Pin | Note |
|---|---|---|
| Leg 1 | **D2** | Input with internal pull-up enabled |
| Leg 2 | **GND** | Ground |

---

## 🎮 Controls

The single push button supports **Short Press** (switch screens) and **Long Press** (trigger action):

| Screen | Short Press (< 600ms) | Long Press (> 600ms) |
|---|---|---|
| **Face** | Go to Quotes | Change expression / mood manually |
| **Quotes** | Go to Pomodoro | Next quote immediately |
| **Pomodoro** | Go to Stopwatch | Start / Pause timer |
| **Stopwatch** | Go to Decider | Start / Pause timer |
| **Decider** | Return to Face | Pick a random answer |

*(Note: Tap button during the startup greeting to skip directly to the face).*

---

## 📦 Software & Libraries

1. Download and install the [Arduino IDE](https://www.arduino.cc/en/software).
2. Open the Arduino IDE Library Manager (`Tools` > `Manage Libraries...` or `Ctrl + Shift + I`).
3. Search for and install **`U8g2`** by Oliver.
4. Open `_desk_buddy_code/_desk_buddy_code.ino`.
5. Select your board (`Arduino Nano`) and processor (`ATmega328P` or `ATmega328P (Old Bootloader)`).
6. Select your COM port and click **Upload**.
