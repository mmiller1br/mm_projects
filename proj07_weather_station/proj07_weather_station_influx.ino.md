
```arduino
// AM2320 Libraries
#include "Adafruit_Sensor.h"
#include "Adafruit_AM2320.h"

// OLED display Libraries
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// Font to be used
#include <Fonts/FreeSerifBold24pt7b.h>
#include <Fonts/FreeSans12pt7b.h>

// Define screen size     
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels

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


// General Libraries
#include <SPI.h>
#include <Wire.h>

// Initialize the OLED display using Arduino Wire:
#define SDA_PIN 14// GPIO21 -> SDA
#define SCL_PIN 15// GPIO22 -> SCL

// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
#define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// WiFi Credentials
#define WIFI_SSID "your-wifi-ssid"
#define WIFI_PASS "your-wifi-password"

// Initialize sensor AM2320
Adafruit_AM2320 am2320 = Adafruit_AM2320();


#define LOGO_HEIGHT   16
#define LOGO_WIDTH    16
static const unsigned char PROGMEM logo_bmp[] =
{ B00000000, B11000000,
  B00000001, B11000000,
  B00000001, B11000000,
  B00000011, B11100000,
  B11110011, B11100000,
  B11111110, B11111000,
  B01111110, B11111111,
  B00110011, B10011111,
  B00011111, B11111100,
  B00001101, B01110000,
  B00011011, B10100000,
  B00111111, B11100000,
  B00111111, B11110000,
  B01111100, B11110000,
  B01110000, B01110000,
  B00000000, B00110000 };

static const unsigned char PROGMEM image_data_thermometer[] = {
    0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xe1, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xce, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xde, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfc, 0xda, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xf0, 0xda, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfc, 0xda, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xf0, 0x04, 0x00, 0xc3, 0xe2, 0x00, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xe0, 0x00, 0x00, 0xc3, 0xc0, 0x00, 0x7f, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xf0, 0xda, 0x7f, 0xe2, 0x32, 0x1e, 0xc1, 0xc3, 0x8c, 0x3f, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xe6, 0x33, 0x1b, 0xe1, 0x83, 0x8c, 0x3f, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfc, 0xda, 0x7f, 0xfe, 0x3f, 0x03, 0xe0, 0x83, 0x8c, 0x3f, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xfe, 0x3f, 0x03, 0xe0, 0x03, 0x80, 0x7f, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xf0, 0xda, 0x7f, 0xfe, 0x3f, 0x13, 0xc8, 0x03, 0x80, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xfe, 0x3f, 0x1b, 0x4c, 0x43, 0x8f, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xfe, 0x1e, 0x1e, 0x4c, 0x43, 0x8f, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xda, 0x7f, 0xf8, 0x0e, 0x00, 0x04, 0x81, 0x07, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xbb, 0x3f, 0xfc, 0x1c, 0x00, 0xc6, 0xc3, 0x07, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0x73, 0x9f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfe, 0xe1, 0xdf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfe, 0xde, 0xdf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfe, 0xde, 0xef, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfe, 0xde, 0xcf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xfe, 0xcc, 0xdf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0x61, 0xdf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0x3f, 0xbf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0x9e, 0x7f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xe0, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
    0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff
};

float t, h;                // temperature and humidity
int d = 0;                 // 0 means temperature first, 1 means humidity first
int time_display = 30000;  // timer to refresh display = 30s
int time_db      = 60000;  // timer to send info to INFLUXDB = 60s
unsigned long time1, time2;   


void setup() {
  int i; // counter
  time1 = millis();
  
  // Initialize software PINs SDA and SCL
  Wire.begin(SDA_PIN,SCL_PIN);

  Serial.begin(9600);
  while (!Serial) {
    delay(10); // hang out until serial port opens
  }

  // SSD1306_SWITCHCAPVCC = generate display voltage from 3.3V internally
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Address 0x3C for 128x32
    Serial.println(F("SSD1306 allocation failed"));
    for(;;); // Don't proceed, loop forever
  }

  //Serial.println("Adafruit AM2320 Basic Test");
  am2320.begin();

  // STEP1: COUNTDOWN
  // Countdown ... 3, 2, 1 ...
  for (i=3; i>0; i--) {
    display.clearDisplay();
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(10,15);  
    display.print(i);
    display.display();
    delay(1000);
  }

  // STEP2: LOGO
  drawbitmap();  
  // Invert and restore display, pausing in-between
  for (i=0; i<3; i++) {
    display.invertDisplay(true);
    delay(500);
    display.invertDisplay(false);
    delay(500);
  }

  // Initialize WiFi
  wifiSetup();  

  // Show display
  showtemp(t,h, 0);
}

void loop() {
  t = am2320.readTemperature();
  h = am2320.readHumidity();
  
  //Serial.print("Temp: "); Serial.println(t);
  //Serial.print("Hum: "); Serial.println(h);

  if ((millis() - time1) >= time_display) {
    showtemp(t, h, d);
    if (d==0) {
      d = 1;
    }
    else {
      d = 0;
    }
    time1 = millis();
  }

  if ((millis() - time2) > time_db) {
    sensor.clearFields();
    sensor.addField("name", "sensor1");
    sensor.addField("temp", t);
    sensor.addField("humidity", h);
    if (!client.writePoint(sensor)) {
       Serial.print("InfluxDB write failed: ");
       Serial.println(client.getLastErrorMessage());
    }
    time2 = millis();
    // how to check the influxdb:
    // influx
    // use THERMOSTAT
    // select * from stations where "name" = 'sensor1' ORDER BY time DESC limit 10
  }
  delay(500);
}

//-----------------------------------------------------------------------------
// Wifi
void wifiSetup() {
     int i = 0;
     int j = 15;
     // Set WIFI module to STA mode
     WiFi.mode(WIFI_STA);

     // Connect
     Serial.printf("[WIFI] Connecting to %s ", WIFI_SSID);


     WiFi.begin(WIFI_SSID, WIFI_PASS);

     display.clearDisplay();
     // Wait
     while (WiFi.status() != WL_CONNECTED) {
         Serial.print(".");
         display.setTextSize(1);             // Normal 1:1 pixel scale
         display.setTextColor(SSD1306_WHITE);        // Draw white text
         display.setCursor(0,0);             // Start at top-left corner
         display.print("Connecting to ");
         display.print(WIFI_SSID);
         display.drawPixel(i, j, SSD1306_WHITE);
         display.display();
         delay(100);
         i = i + 1;
         if (i == 127) {
           i = 0;
           j = j+3;
         }
     }
     Serial.println();

     // Connected!
     Serial.printf("[WIFI] STATION Mode, SSID: %s, IP address: %s\n", WiFi.SSID().c_str(), WiFi.localIP().toString().c_str());

     display.clearDisplay();
     display.setCursor(0,0);             
     display.print("Connected !");
     display.setCursor(0, 15);
     display.print("IP = ");
     display.print(WiFi.localIP().toString().c_str());
     display.display();
     delay(2000);
}
//-----------------------------------------------------------------------------


void drawbitmap(void) {
  display.clearDisplay();

  //display.drawBitmap(
  //  (display.width()  - LOGO_WIDTH ) / 2,
  //  (display.height() - LOGO_HEIGHT) / 2,
  //  logo_bmp, LOGO_WIDTH, LOGO_HEIGHT, 1);
  display.drawBitmap(0,0,image_data_thermometer, 128, 32, 1);  
  display.display();
  delay(1000);
}

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