
```c++
//
// ESP8266 PINs used on this project:
//
// outdoor temp sensor AM2301: pin D7
// indoor temp sensor AHT10:   pin D14 and D15 (I2C communication)
// door sensor (magnetic):     pin D13
// servo motor (push button):  pin D8
//

#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <Servo.h>
#include "DHT.h"
#include <InfluxDbClient.h>
#include <Adafruit_AHT10.h>
 
#define WIFI_SSID "wifi-name"
#define WIFI_PASS "wifi-password"

// ALEXA support for ESP8266 board
#include "fauxmoESP.h"
fauxmoESP fauxmo;

// Servo Motor Initialization
Servo servo;
#define SERVO_PIN D8
int angle = 90;

// AM2301 outdoor TEMP and HUMID sensor
#define DHTPIN D7       // modify to the pin we connected
#define DHTTYPE DHT21   // AM2301
DHT dht(DHTPIN, DHTTYPE);

// AHT10 indoor TEMP and HUMID sensor
// it uses I2C communication, pins SDA/SCK - WEMOS pins D12 and D13
Adafruit_AHT10 aht;

// InfluxDB server url, e.g. http://192.168.1.48:8086 (don't use localhost, always server name or ip address)
#define INFLUXDB_URL "http://192.168.2.48:8086"
// InfluxDB database name 
#define INFLUXDB_DB_NAME "THERMOSTAT"
// Single InfluxDB instance
InfluxDBClient client(INFLUXDB_URL, INFLUXDB_DB_NAME);

// Data point
Point sensor("stations");

float temp_outdoor, humid_outdoor;
sensors_event_t temp_indoor, humid_indoor;

#define DOOR_SENSOR D5
int read_door;
bool change = false;

#define SERIAL_BAUDRATE 115200

// Define the name as this device will be recognized and used in ALEXA
#define garage "garage"

// Wifi
void wifiSetup() {
     // Set WIFI module to STA mode
     WiFi.mode(WIFI_STA);

     // Connect
     Serial.printf("[WIFI] Connecting to %s ", WIFI_SSID);
     WiFi.begin(WIFI_SSID, WIFI_PASS);

     // Wait
     while (WiFi.status() != WL_CONNECTED) {
         Serial.print(".");
         delay(100);
     }
     Serial.println();

     // Connected!
     Serial.printf("[WIFI] STATION Mode, SSID: %s, IP address: %s\n", WiFi.SSID().c_str(), WiFi.localIP().toString().c_str());
}

void setup() {

     // Init serial port and clean garbage
     Serial.begin(SERIAL_BAUDRATE);
     Serial.println();
     Serial.println();

     // LEDs
     pinMode(LED_BUILTIN, OUTPUT);
     digitalWrite(LED_BUILTIN, LOW);

     // DOOR Sensor initialization
     pinMode(DOOR_SENSOR, INPUT_PULLUP);

     // DHT initialization
     dht.begin();

     // AHT initialization
     aht.begin();

     // Wifi initialization
     wifiSetup();
     
     // SERVO initialization
     servo.attach(SERVO_PIN);
     servo.write(angle);

     // InfluxDB Server initialization
     if (client.validateConnection()) {
        Serial.print("Connected to InfluxDB: ");
        Serial.println(client.getServerUrl());
     } else {
        Serial.print("InfluxDB connection failed: ");
        Serial.println(client.getLastErrorMessage());
     }

     // By default, fauxmoESP creates it's own webserver on the defined port
     // The TCP port must be 80 for gen3 devices (default is 1901)
     // This has to be done before the call to enable()
     fauxmo.createServer(true); // NOT needed, this is the default value
     fauxmo.setPort(80);        // This is required for gen3 devices

     // You have to call enable(true) once you have a WiFi connection
     // You can enable or disable the library at any moment
     // Disabling it will prevent the devices from being discovered and switched
     fauxmo.enable(true);

     // You can use different ways to invoke alexa to modify the devices state:
     // "Alexa, turn yellow lamp on"
     // "Alexa, turn on yellow lamp
     // "Alexa, set yellow lamp to fifty" (50 means 50% of brightness, note, this example does not use this functionality)

     // Add virtual devices (device name = "garage")
     fauxmo.addDevice(garage);

     fauxmo.onSetState([](unsigned char device_id, const char *device_name, bool state, unsigned char value) {

         // Callback when a command from Alexa is received.
         // You can use device_id or device_name to choose the element to perform an action onto (relay, LED,...)
         // State is a boolean (ON/OFF) and value a number from 0 to 255 (if you say "set kitchen light to 50%" you will receive a 128 here).
         // Just remember not to delay too much here, this is a callback, exit as soon as possible.
         // If you have to do something more involved here set a flag and process it in your main loop.

         Serial.printf("[MAIN] Device #%d (%s) state: %s value: %d\n", device_id, device_name, state ? "ON" : "OFF", value);  
          
         // Checking for device_id is simpler if you are certain about the order they are loaded and it does not change.
         // Otherwise comparing the device_name is safer.

         if (strcmp(device_name, garage)==0) {
             digitalWrite(LED_BUILTIN, state ? LOW : HIGH);
             read_door = digitalRead(DOOR_SENSOR);
             Serial.print("Door Sensor: ");
             Serial.println(read_door);
             Serial.print("STATE: ");
             Serial.println(state);
             
             if (((state == LOW) && (read_door == LOW)) || ((state == HIGH) && (read_door == HIGH))) {
                change = true;
             }
         }

     });

}

void loop() {

     // fauxmoESP uses an async TCP server but a sync UDP server
     // Therefore, we have to manually poll for UDP packets
     fauxmo.handle();

     // This is a sample code to output free heap every 5 seconds
     // This is a cheap way to detect memory leaks
     static unsigned long last = millis();
     static unsigned long timer = millis();
     
     if (millis() - last > 5000) {
        last = millis();
        Serial.printf("[MAIN] Free heap: %d bytes\n", ESP.getFreeHeap());

        read_door = digitalRead(DOOR_SENSOR);
        if (read_door == HIGH) {
           Serial.println("Door CLOSED");
        }
        else {
           Serial.println("Door OPEN");
        }
     }

     // write INFLUXDB 1/minute
     if (millis() - timer > 60000) {      
        // AM2301
        humid_outdoor = dht.readHumidity();
        temp_outdoor  = dht.readTemperature();
        Serial.println("INFLUXDB push ...");
        Serial.print("T_outdoor: ");
        Serial.print(temp_outdoor);
        Serial.print(", H_outdoor= ");
        Serial.println(humid_outdoor);   

        sensor.clearFields();
        sensor.addField("name", "sensor3");
        sensor.addField("temp", temp_outdoor);
        sensor.addField("humidity", humid_outdoor);
        // send info to DB
        client.writePoint(sensor);

        // AHT10
        aht.getEvent(&humid_indoor, &temp_indoor);// populate temp and humidity objects with fresh data
        Serial.print("T_Indoor: "); 
        Serial.print(temp_indoor.temperature); 
        Serial.print(" degrees C");
        Serial.print(", H_Indoor: "); 
        Serial.print(humid_indoor.relative_humidity); 
        Serial.println("%");
        timer = millis();

        sensor.clearFields();
        sensor.addField("name", "sensor2");
        sensor.addField("temp", temp_indoor.temperature);
        sensor.addField("humidity", humid_indoor.relative_humidity);
        // send info to DB
        client.writePoint(sensor);
     }

     // If your device state is changed by any other means (MQTT, physical button,...)
     // you can instruct the library to report the new state to Alexa on next request:
     // fauxmo.setState(ID_YELLOW, true, 255);

     if (change == true) {
        Serial.print("push_button");
        push_button();
        change = false;
     }     
     //if (digitalRead(DOOR_SENSOR) == HIGH) {
     //   Serial.print("door OPEN");
     //}
     //else {
     //   Serial.print("door CLOSED");
     //}
     delay(100);
}

void push_button() {         
   for(angle = 90; angle < 160; angle+=10)    
   {                                  
      servo.write(angle);       
      delay(10);         
   }
   delay(100);
   for(angle = 160; angle > 90; angle-=5) 
   {
      servo.write(angle);
      //delay(10);
   }
}  
```