
# Garage Door Controller + Weather Station
🚪🌡️📊
#arduino #influxdb #sensor #grafana #iot

## Description:

This is an Arduino project - in this case a ESP8266 - used to control the garage door using Alexa. This Arduino is connected to the WiFi and it uses the Library [FauxmoESP](https://bitbucket.org/xoseperez/fauxmoesp/get/master.zip) to connect to Alexa. 

It has a magnetic door sensor to check if the door is OPEN or CLOSED and avoid trying to open is the door is already opened, or close it if the door is already closed.

To make better use of this Arduino, I'm including 2 extra sensors (temperature and humidity), one indoor and one outdoor. All data captured from these sensors will be sent to an external InfluxDB sensor running on a Raspberry PI server ([IOT-Stack](https://sensorsiot.github.io/IOTstack/)) and, with Grafana, we can create beautiful dashboards.

### Hardware:
- Arduino ESP866 Wemos D1-R1
- Indoor Temp/Humid Sensor AHT10
- Outdoor Temp/Humid Sensor AM2301
- Servo Motor SG90
- Magnetic Door Sensor

### Software:
- InfluxDB to store all data from my temperature sensors
- Grafana to create dashboards with the information stored on InfluxDB
(*these are docker containers running on my local Raspberry PI mini-server*)

### Picture:

To make it a clean solution, I've used my 3D printer to design 2 parts for this project: 
- a case to accomodate the Arduino + connections to all sensors 
- a case to attach the servo motor to the switch used to open/close the garage door

All wires used to connect the Arduino ports to the connectors are made with silicone, which make the final result much better.

![image](https://github.com/mmiller1br/mm_projects/assets/32887571/7003a9f9-9f67-45a9-b04f-37178a98c973)

And this is how the servo motor works when you ask Alexa to OPEN or CLOSE your garage. It's my version of [SwitchBot Bot](https://ca.switch-bot.com/collections/all/products/switchbot-bot), but homemade ;)

![proj01_button](https://github.com/mmiller1br/mm_projects/assets/32887571/24ac0c09-0d6c-4017-9777-d695ac67bc46)

## Code comments:

1. define your WiFi credentials and create a function to initialize the WiFi module and connect to your home wifi:
```arduino
#define WIFI_SSID "wifi-name"
#define WIFI_PASS "wifi-password"
...
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
```

2. write your data from the temperature/humidity sensors to your InfluxDB database:
```arduino
     // write INFLUXDB 1/minute
     if (millis() - timer > 60000) {      
        // AM2301
        humid_outdoor = dht.readHumidity();
        temp_outdoor  = dht.readTemperature();   

        sensor.clearFields();
        sensor.addField("name", "sensor3");
        sensor.addField("temp", temp_outdoor);
        sensor.addField("humidity", humid_outdoor);

        // AHT10
        aht.getEvent(&humid_indoor, &temp_indoor);

        timer = millis();

        sensor.clearFields();
        sensor.addField("name", "sensor2");
        sensor.addField("temp", temp_indoor.temperature);
        sensor.addField("humidity", humid_indoor.relative_humidity);

```

3. Define the name to be used on Alexa for this device:
```arduino
// Define the name as this device will be recognized and used in ALEXA
#define garage "garage"
```

4. For every time Alexa send a command to the Arduino, there is a check to see if the function "push_button" needs to be called or not. This is done ny the variable "change" - it's set to TRUE everytime that the push button needs to be exeuted. That is:
	- state LOW means command "close the door"
	- state HIGH means command "open the door"
	- read_door LOW means "door is OPEN"
	- read_door HIGH means "door is CLOSED"

	So the logic is: if door is CLOSED and command is OPEN IT, or door is OPEN and command is CLOSE IT, we need to set the FLAG "change" to TRUE.
	
```arduino
     // Add virtual devices (device name = "garage")
     fauxmo.addDevice(garage);

     fauxmo.onSetState([](unsigned char device_id, const char *device_name, bool state, unsigned char value) {

         Serial.printf("[MAIN] Device #%d (%s) state: %s value: %d\n", device_id, device_name, state ? "ON" : "OFF", value);  
          
         if (strcmp(device_name, garage)==0) {
             digitalWrite(LED_BUILTIN, state ? LOW : HIGH);
             read_door = digitalRead(DOOR_SENSOR);
             
             if (((state == LOW) && (read_door == LOW)) || ((state == HIGH) && (read_door == HIGH))) {
                change = true;
             }
         }


```

5. Function PUSH_BUTTON (using the servo motor to push that button)
```arduino
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

## Grafana dashboard:

![image](https://github.com/mmiller1br/mm_projects/assets/32887571/6cbec1b1-0bc0-4fe0-9d77-db08e9bd2ad7)

This dashboard can be used directlly on Grafana, or in my case, on HomeAssistant, where I have visibility about Temp/Humid on differentes areas in my house. SENSOR1 is information from my basement, SENSOR2 and SENSOR3 are info from this project, with information from my garage (indoor = AHT10) and outdoor (AM2301), respectively.
