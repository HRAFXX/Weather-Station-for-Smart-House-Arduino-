# 🌤️ Weather Station for Smart House - Arduino Edition

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform: Arduino](https://img.shields.io/badge/Platform-Arduino-blue.svg)](https://www.arduino.cc/)
[![Language: C++](https://img.shields.io/badge/Language-C%2B%2B-orange.svg)]()

## 📡 About This Project

Welcome to your very own DIY Smart House Weather Station! This project uses an Arduino Uno and a DHT11 sensor to monitor **temperature** and **humidity**. Plus, it displays everything on a slick color TFT screen so your home gets its own meteorology lab.

Whether you're looking to smarten up your house or just love playing with sensors, you're in the right place. Let’s go full weather wizard! 🌪️

<p align="center">
  <img src="images/weather-station-demo.jpg" alt="Weather Station Display" width="400"/>
</p>

---

## 🌟 Features

- **Dual Readings**: Reads both **temperature** and **humidity** in real time.
- **Colorful Display**: Live weather info on a bright TFT screen (with emojis for extra flair).
- **Plug-and-Play**: Easy to assemble, beginner-friendly code.
- **Expandable**: Add more sensors, WiFi modules, or even cloud logging!

---

## 🧰 Tech Stack

- **🔌 Arduino Uno R3** – The brain of the system.
- **🌡️ DHT11 Sensor** – Measures temperature and humidity.
- **📺 ILI9341 TFT Display** – Shows data in style (can be replaced with an OLED).
- **⚡ C++ / Arduino IDE** – Programming environment.

---

## 🛠️ Hardware Required

| Component              | Quantity |
|------------------------|----------|
| Arduino Uno R3         | 1        |
| DHT11 Temp/Humidity Sensor | 1    |
| ILI9341 TFT Display (or similar) | 1 |
| Breadboard             | 1        |
| Jumper Wires           | Several  |
| USB Cable for Arduino  | 1        |
| 4.7kΩ Resistor         | 1        |

---

## 🧪 How to Build It

### 1. **Wiring the Circuit**

- **DHT11 Sensor**
  - VCC → 5V
  - GND → GND
  - DATA → Digital Pin 2
  - 4.7kΩ resistor between VCC and DATA

- **TFT Display (ILI9341 Example Pins)**
  - CS → Pin 10  
  - DC → Pin 9  
  - RST → Pin 8  
  - SCLK → Pin 13  
  - MOSI → Pin 11  
  - VCC → 5V  
  - GND → GND

> Make sure to match your display's wiring if it differs.

---

## 💻 Installation & Upload

1. **Install the Arduino IDE** – [Download it here](https://www.arduino.cc/en/software)
2. **Install Libraries** via Library Manager:
   - `DHT sensor library` by Adafruit
   - `Adafruit Unified Sensor`
   - `Adafruit GFX`
   - `Adafruit ILI9341`
3. **Connect your Arduino** via USB
4. **Open the Sketch** and upload the code below 👇

---

## 📜 Full Arduino Code

```cpp
#include <DHT.h>
#include <Adafruit_GFX.h>
#include <Adafruit_ILI9341.h>

// ==== Pin Config ====
#define DHTPIN 2
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

// TFT Display Pins (update if needed)
#define TFT_CS    10
#define TFT_DC     9
#define TFT_RST    8

Adafruit_ILI9341 tft = Adafruit_ILI9341(TFT_CS, TFT_DC, TFT_RST);

void setup() {
  Serial.begin(9600);
  dht.begin();
  tft.begin();
  tft.setRotation(1);
  tft.fillScreen(ILI9341_BLACK);
  tft.setTextSize(2);
  tft.setTextColor(ILI9341_WHITE);
}

void loop() {
  float temp = dht.readTemperature();
  float hum = dht.readHumidity();

  if (isnan(temp) || isnan(hum)) {
    Serial.println("❌ Failed to read from DHT sensor!");
    return;
  }

  Serial.print("🌡️ Temp: ");
  Serial.print(temp);
  Serial.print(" °C | 💧 Humidity: ");
  Serial.print(hum);
  Serial.println(" %");

  // Draw on TFT screen
  tft.fillScreen(ILI9341_BLACK);
  tft.setCursor(10, 30);
  tft.print("🌡 Temp: ");
  tft.print(temp);
  tft.println(" C");

  tft.setCursor(10, 60);
  tft.print("💧 Hum: ");
  tft.print(hum);
  tft.println(" %");

  delay(3000); // Update every 3 sec
}
