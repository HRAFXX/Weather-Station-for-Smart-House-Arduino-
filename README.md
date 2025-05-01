# Weather-Station-for-Smart-House-Arduino-
Designed and built a weather station integrated with an innovative house system using Arduino. Used sensors to collect real-time weather data such as temperature, humidity, and atmospheric pressure. Developed a user interface to display weather information and control smart devices based on weather conditions.

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "DHT.h"

// Constants for the hall effect sensor (wind speed)
const int hallPin = 2; // Hall effect sensor connected to digital pin 2
const float magnetDistance = 0.10; // Distance per revolution in meters (adjust as needed)
volatile unsigned long startTime = 0; // Time when a magnet is detected
volatile unsigned long endTime = 0; // Time when the next magnet is detected
volatile boolean newWindData = false; // Flag to indicate new wind data
volatile int magnetCounter = 0; // Counter for magnet detections

// Constants for the rain gauge
const float mmPerPulse = 0.173; // Value of rain in mm for each movement of the bucket
float mmTotali = 0;
int sensore = 0;
int statoPrecedente = 0;

// Constants for the DHT sensor
#define DHTPIN 3 // Changed to avoid conflict with hallPin
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

// LCD initialization
LiquidCrystal_I2C lcd(0x27, 20, 4);

// Function declarations
void detectMagnet();
float calculateWindSpeed();
void displayWindSpeed(float windSpeed);
void displayRainfall(float rainfall);
void displayDHTData(float humidity, float tempC, float tempF, float heatIndexC, float heatIndexF);

void setup() {
  // Setup for hall effect sensor (wind speed)
  Serial.begin(9600); // Start serial communication at 9600 baud
  pinMode(hallPin, INPUT_PULLUP); // Set hallPin as input with pull-up resistor
  attachInterrupt(digitalPinToInterrupt(hallPin), detectMagnet, RISING); // Attach interrupt to hallPin
  Serial.println("Setup completed. Waiting for magnet detection...");

  // Setup for rain gauge
  pinMode(9, INPUT);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(6, 0);
  lcd.print("HRAF");
  lcd.setCursor(3, 2);
  lcd.print("Weather Station");
  delay(1000);
  lcd.clear();

  // Setup for DHT sensor
  dht.begin();
}

void loop() {
  // Loop for hall effect sensor (wind speed)
  if (newWindData) { // If new wind data is available
    float windSpeed = calculateWindSpeed(); // Calculate wind speed
    Serial.print("Wind Speed: ");
    Serial.print(windSpeed);
    Serial.println(" m/s");
    newWindData = false; // Reset the flag
    Serial.println("Waiting for next magnet detection...");
    displayWindSpeed(windSpeed); // Display wind speed on LCD
  }

  // Loop for rain gauge
  sensore = digitalRead(9);
  if (sensore != statoPrecedente) {
    mmTotali = mmTotali + mmPerPulse;
  }
  statoPrecedente = sensore;
  displayRainfall(mmTotali); // Display rainfall on LCD

  // Loop for DHT sensor
  delay(2000);
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  float f = dht.readTemperature(true);

  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
  }

  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);

  Serial.print(F("Humidity: "));
  Serial.print(h);
  Serial.print(F("%  Temperature: "));
  Serial.print(t);
  Serial.print(F("°C "));
  Serial.print(f);
  Serial.print(F("°F  Heat index: "));
  Serial.print(hic);
  Serial.print(F("°C "));
  Serial.print(hif);
  Serial.println(F("°F"));

  displayDHTData(h, t, f, hic, hif); // Display DHT data on LCD
}

void detectMagnet() {
  magnetCounter++; // Increment magnet counter

  if (magnetCounter % 2 == 1) { // First detection in a pair
    startTime = millis();
    Serial.println("First magnet detected.");
  } else { // Second detection in a pair
    endTime = millis();
    Serial.println("Second magnet detected.");
    newWindData = true;
  }
}

float calculateWindSpeed() {
  float elapsedTime = (endTime - startTime) / 1000.0; // Convert milliseconds to seconds
  if (elapsedTime == 0) {
    Serial.println("Elapsed time is zero, avoiding division by zero.");
    return 0; // Avoid division by zero
  }
  float windSpeed = magnetDistance / elapsedTime; // Speed = Distance / Time
  startTime = endTime; // Reset the start time for the next calculation
  endTime = 0; // Reset the end time for the next calculation
  return windSpeed;
}

void displayWindSpeed(float windSpeed) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Wind Speed:");
  lcd.setCursor(0, 1);
  lcd.print(windSpeed);
  lcd.print(" m/s");
  delay(5000); // Wait for 5 seconds
}

void displayRainfall(float rainfall) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Pluviometer");
  lcd.setCursor(0, 1);
  lcd.print("Total precipitation:");
  lcd.setCursor(0, 2);
  lcd.print(rainfall);
  lcd.print(" mm");
  delay(5000); // Wait for 5 seconds
}

void displayDHTData(float humidity, float tempC, float tempF, float heatIndexC, float heatIndexF) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Humidity:");
  lcd.setCursor(0, 1);
  lcd.print(humidity);
  lcd.print("%");
  
  lcd.setCursor(0, 2);
  lcd.print("Temp:");
  lcd.setCursor(0, 3);
  lcd.print(tempC);
  lcd.print("C ");
  lcd.print(tempF);
  lcd.print("F");
  delay(5000); // Wait for 5 seconds

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Heat Index:");
  lcd.setCursor(0, 1);
  lcd.print(heatIndexC);
  lcd.print("C ");
  lcd.print(heatIndexF);
  lcd.print("F");
  lcd.setCursor(0, 2);
  lcd.print("Pressure :");
  lcd.setCursor(0, 3);
  lcd.print("1015hPa");
  delay(5000); // Wait for 5 seconds
}

