```c++
// include the LiquidCrystal library code:  
#include <LiquidCrystal.h>

// include the Ultrasonic sensor library code:  
#include <Ultrasonic.h>

// initialize the library by associating any needed LCD interface pin  
// with the arduino pin number it is connected to  
const int rs = 12, en = 11, d4 = 5, d5 = 4, d6 = 3, d7 = 2;  
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

int sec;  
int rest = 2; // number of free slots  
int slot1led1 = 14, slot1led2 = 15 , space1 = 0;  
int slot2led1 = 16, slot2led2 = 17 , space2 = 0;

// Initialize ultrasonic pins - SLOT1  
#define pino_trigger1 6  
#define pino_echo1 7  
// Initialize ultrasonic pins - SLOT2  
#define pino_trigger2 8  
#define pino_echo2 9

Ultrasonic ultrasonic1(pino_trigger1, pino_echo1);  
Ultrasonic ultrasonic2(pino_trigger2, pino_echo2);

  
void setup() {  
  // set up the LCD's number of columns and rows:  
  lcd.begin(16, 2);  
  // Print a message to the LCD.  
  lcd.print("Free Spaces");  
    
  pinMode(slot1led1, OUTPUT);  
  pinMode(slot1led2, OUTPUT);  
  pinMode(slot2led1, OUTPUT);  
  pinMode(slot2led2, OUTPUT);  
  digitalWrite(slot1led1, HIGH); // green  
  digitalWrite(slot1led2, LOW);  // red  
  digitalWrite(slot2led1, HIGH); // green  
  digitalWrite(slot2led2, LOW);  // red  
}

void loop() {  
  // set the cursor to column 0, line 1  
  // (note: line 1 is the second row, since counting begins with 0):  
  lcd.setCursor(0, 1);  
  // print the number of seconds since reset:  
  // sec = millis() / 1000;

  lcd.print(rest);  
  lcd.print(" / 2 ... ");  
  // lcd.print(sec);

  // Read the info from the sensor, in CM and INCHES - SLOT1  
  float cmMsec1, inMsec1;  
  long microsec1 = ultrasonic1.timing();  
  cmMsec1 = ultrasonic1.convert(microsec1, Ultrasonic::CM);  
  inMsec1 = ultrasonic1.convert(microsec1, Ultrasonic::IN);   
  // Serial.print("Distancia em cm: ");  
  // Serial.print(cmMsec);  
  lcd.print(cmMsec1);

  // Read the info from the sensor, in CM and INCHES - SLOT2   
  float cmMsec2, inMsec2;  
  long microsec2 = ultrasonic2.timing();  
  cmMsec2 = ultrasonic2.convert(microsec2, Ultrasonic::CM);  
  inMsec2 = ultrasonic2.convert(microsec2, Ultrasonic::IN);   
  // Serial.print("Distancia em cm: ");  
  // Serial.print(cmMsec);  
  // lcd.print(cmMsec2);

  delay(500);

  if (cmMsec1 > 7) {  
  digitalWrite(slot1led1, HIGH);  
  digitalWrite(slot1led2, LOW);  
  if (space1 == 1) {  
     space1 = 0;  
     rest++;       
  }  
  }  
  if (cmMsec1 < 7) {  
  digitalWrite(slot1led1, LOW);  
  digitalWrite(slot1led2, HIGH);  
  if (space1 == 0) {  
     space1 = 1;  
     rest--;       
  }  
  }

  if (cmMsec2 > 7) {  
  digitalWrite(slot2led1, HIGH);  
  digitalWrite(slot2led2, LOW);  
  if (space2 == 1) {  
     space2 = 0;  
     rest++;       
  }  
  }  
  if (cmMsec2 < 7) {  
  digitalWrite(slot2led1, LOW);  
  digitalWrite(slot2led2, HIGH);  
  if (space2 == 0) {  
     space2 = 1;  
     rest--;       
  }  
  }

}
```