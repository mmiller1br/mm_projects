# Arduino Weather Station
🌡️💦📊💻

#arduino #sensor #weather #esp32


## Description:

![](_assets/proj07_gif.gif)

This is an Arduino project - in this case an ESP32 - to create an indoor weather station (can be outdoor as well, if you change the sensor - see my project 01). 

The sensor reads information about Temperature and Humidity and send these values to an external Database (InfluxDB), running in a Raspberry PI Server (https://www.youtube.com/shorts/Sa97st6NF4c), once every 60 seconds. With these values in the database, you can use them to create graphs and dashboards - in my case I have these values available on my HomeAssistant Dashboard using Grafana, where I can see historical information from different areas in my house.

## List of Equipments:

- Arduino: if you don´t know what is an Arduino, you can start using these links: 
  [https://www.arduino.cc/](https://www.arduino.cc/)
  [https://en.wikipedia.org/wiki/Arduino](https://en.wikipedia.org/wiki/Arduino)​

- Temp/Humid sensor model AM2320 (better accurancy): 
  [https://www.adafruit.com/product/3721](https://www.adafruit.com/product/3721)



## Coding:

This project has some parts that will be described below:
- sensor reading
- OLED display to show the current readings
- influxdb connectivity to store all readings

1) **Sensor Reading**: this project is using the sensor AM2320. It gives more accurate results than the basic sensor DHT11. Another option would be the model AHT10, that gives even better results:

```arduino
// AM2320 Libraries
#include "Adafruit_Sensor.h"
#include "Adafruit_AM2320.h"

// Initialize sensor AM2320
Adafruit_AM2320 am2320 = Adafruit_AM2320();

...

am2320.begin();

...

t = am2320.readTemperature();
h = am2320.readHumidity();
```

3) **OLED display**: the OLED display used here was the basic 128x32 pixels using an Adafruit library SSD13306. For the scrolling feature to change between TEMP and HUMID, I had to create a function to change the display and create this effect.

```arduino
// OLED display Libraries
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

... 

// Define screen size     
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels

// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
#define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

...

void showtemp(float temp, float humid, int info) {
  int x;
    
  // TEMP big, HUMID small (scroll DOWN old display, scroll UP new display)
  if (info == 0) {
    for (x=0; x<=32; x++) {
      display.clearDisplay();
      display.setFont();
      display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
      //display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setTextSize(1);             // Normal 1:1 pixel scale
      display.setCursor(0, x);     // Start at top-left corner
      display.print("     Temp: " + String(temp,1) + " C   ");
  
      display.setFont(&FreeSans12pt7b);
      display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setCursor(28, x+31);     // Start at top-left corner
      display.print(humid, 1);
      display.print(" %");
      display.display();
      delay(30);
    }
    delay(300);
    
    for (x=32; x>=0; x--) {
      display.clearDisplay();
      display.setFont();
      display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
      //display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setTextSize(1);             // Normal 1:1 pixel scale
      display.setCursor(0, x);     // Start at top-left corner
      display.print("      Hum: " + String(humid,1) + "%     ");
  
      display.setFont(&FreeSans12pt7b);
      display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setCursor(30, x+31);     // Start at top-left corner
      display.print(temp, 1);
      display.setCursor(77, x+31);
      display.print("  C");
      display.drawCircle(84, x+17, 3, SSD1306_WHITE);
      display.display();
      delay(30);
    }
  }
  // HUMID big, TEMP small
  else {
    for (x=0; x<=32; x++) {
      display.clearDisplay();
      display.setFont();
      display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
      //display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setTextSize(1);             // Normal 1:1 pixel scale
      display.setCursor(0, x);     // Start at top-left corner
      display.print("      Hum: " + String(humid,1) + "%     ");
  
      display.setFont(&FreeSans12pt7b);
      display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setCursor(30, x+31);     // Start at top-left corner
      display.print(temp, 1);
      display.setCursor(77, x+31);
      display.print("  C");
      display.drawCircle(84, x+17, 3, SSD1306_WHITE);
      display.display();
      delay(30); 
    }
    delay(300);
    
    for (x=32; x>=0; x--) {
      display.clearDisplay();
      display.setFont();
      display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
      //display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setTextSize(1);             // Normal 1:1 pixel scale
      display.setCursor(0, x);     // Start at top-left corner
      display.print("     Temp: " + String(temp,1) + " C   ");
  
      display.setFont(&FreeSans12pt7b);
      display.setTextColor(SSD1306_WHITE); // Draw white text
      display.setCursor(28, x+31);     // Start at top-left corner
      display.print(humid, 1);
      display.print(" %");
      display.display();
      delay(30);
    }
  }
}
```

5) **InfluxDB**: finally, after reading the sensor and present the result on the OLED display,, it's time to send these values to an external InfluxDB database. To enable this, I'm using a library to config this ESP32 as a influxdb client and send these values to an InfluxDB Server that is running on a Raspberry PI.
   
   The InfluxDB is running as a docker container as part of the IOTStack (https://sensorsiot.github.io/IOTstack/), in case you want to know more about this part. Thanks @Andreas Spiess for the excelent video (https://www.youtube.com/watch?v=a6mjt8tWUws).

```
// INFLUXDB Library
#include <InfluxDbClient.h>

// InfluxDB server url, e.g. http://192.168.1.48:8086 (don't use localhost, always server name or ip address)
#define INFLUXDB_URL "http://192.168.2.250:8086"
// InfluxDB database name 
#define INFLUXDB_DB_NAME "THERMOSTAT"

// Single InfluxDB instance
InfluxDBClient client(INFLUXDB_URL, INFLUXDB_DB_NAME);

// Data point
Point sensor("stations");

...

// this is part of the LOOP function
  if ((millis() - time2) > time_db) {
    sensor.clearFields();
    sensor.addField("name", "sensor1");
    sensor.addField("temp", t);
    sensor.addField("humidity", h);
    if (!client.writePoint(sensor)) {
       Serial.print("InfluxDB write failed: ");
       Serial.println(client.getLastErrorMessage());
```

Finally, with all values on your Database, I'm using Grafana to create a Dashboard and present this information on my HomeAssistant. I'm also using the values from my InfluxDB database to create entities on HomeAssistant and present this using any card available.  Example of my dashboard here:

![](_assets/proj01_dashboard.png)


## Video

📽️ https://www.youtube.com/shorts/u7VZ8gYpaBI

