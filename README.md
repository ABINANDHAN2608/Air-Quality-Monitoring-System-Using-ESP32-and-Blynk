# Air-Quality-Monitoring-System-Using-ESP32-and-Blynk



/*******************************************************
 * Air Quality Monitoring System using ESP32
 * Components:
 *  - ESP32
 *  - MQ135 Gas Sensor
 *  - DHT11 Temperature & Humidity Sensor
 *  - 16x2 I2C LCD
 *  - Blynk IoT
 *******************************************************/
```
#define BLYNK_TEMPLATE_ID "TMPLwToQUqRw"
#define BLYNK_TEMPLATE_NAME "Air Quality Monitoring"
#define BLYNK_AUTH_TOKEN "C8Y7T0Fr54QF8pdfQ5dZsdfhhSdiQBFLj8mYe"

#define BLYNK_PRINT Serial

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>
#include <LiquidCrystal_I2C.h>

//-------------------------
// LCD Configuration
//-------------------------
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Custom Degree Symbol
byte degree_symbol[8] =
{
  0b00111,
  0b00101,
  0b00111,
  0b00000,
  0b00000,
  0b00000,
  0b00000,
  0b00000
};

//-------------------------
// WiFi & Blynk Credentials
//-------------------------
char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "WiFi Username";
char pass[] = "WiFi Password";

//-------------------------
// Timer
//-------------------------
BlynkTimer timer;

//-------------------------
// MQ135 Gas Sensor
//-------------------------
int gas = 32;               // GPIO32
int sensorThreshold = 1200; // Threshold for Bad Air

//-------------------------
// DHT11 Configuration
//-------------------------
#define DHTPIN 2
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

//-------------------------------------------------
// Function to send sensor data to Blynk
//-------------------------------------------------
void sendSensor()
{
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature))
  {
    Serial.println("Failed to read from DHT11 sensor!");
    return;
  }

  int gasValue = analogRead(gas);

  // Send values to Blynk
  Blynk.virtualWrite(V0, temperature);
  Blynk.virtualWrite(V1, humidity);
  Blynk.virtualWrite(V2, gasValue);

  Serial.print("Temperature : ");
  Serial.print(temperature);
  Serial.println(" °C");

  Serial.print("Humidity : ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.print("Gas Value : ");
  Serial.println(gasValue);
}

//-------------------------------------------------
// Setup
//-------------------------------------------------
void setup()
{
  Serial.begin(115200);

  // Connect to WiFi & Blynk
  Blynk.begin(auth, ssid, pass);

  // Initialize DHT11
  dht.begin();

  // Initialize LCD
  lcd.begin();
  lcd.backlight();

  lcd.createChar(1, degree_symbol);

  lcd.setCursor(3, 0);
  lcd.print("Air Quality");

  lcd.setCursor(3, 1);
  lcd.print("Monitoring");

  delay(2000);
  lcd.clear();

  // Send data every 30 seconds
  timer.setInterval(30000L, sendSensor);
}

//-------------------------------------------------
// Main Loop
//-------------------------------------------------
void loop()
{
  Blynk.run();
  timer.run();

  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();
  int gasValue = analogRead(gas);

  //-------------------------
  // Display Temperature
  //-------------------------
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("Temperature");

  lcd.setCursor(0, 1);
  lcd.print(temperature);

  lcd.write(1);     // Degree Symbol
  lcd.print("C");

  delay(4000);

  //-------------------------
  // Display Humidity
  //-------------------------
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("Humidity");

  lcd.setCursor(0, 1);
  lcd.print(humidity);
  lcd.print("%");

  delay(4000);

  //-------------------------
  // Display Air Quality
  //-------------------------
  lcd.clear();

  if (gasValue < sensorThreshold)
  {
    lcd.setCursor(0, 0);
    lcd.print("Gas:");
    lcd.print(gasValue);

    lcd.setCursor(0, 1);
    lcd.print("Fresh Air");

    Serial.println("Fresh Air");
  }
  else
  {
    lcd.setCursor(0, 0);
    lcd.print("Gas:");
    lcd.print(gasValue);

    lcd.setCursor(0, 1);
    lcd.print("Bad Air");

    Serial.println("Bad Air");

    // Send Notification
    Blynk.logEvent("pollution_alert", "Bad Air");
  }

  Serial.print("Gas Value : ");
  Serial.println(gasValue);

  delay(4000);
}
```
