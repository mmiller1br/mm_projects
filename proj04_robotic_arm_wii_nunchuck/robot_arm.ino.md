
```c++
  
/*  
 * Arduino Robotic ARM Control --   
 */

#include <Wire.h>  
#include "nunchuck_funcs.h"

byte accx,accy,zbut,cbut, joyx, joyy;

  
void setup()  
{  
    Serial.begin(9600);  
    nunchuck_setpowerpins();  
    nunchuck_init(); // send the initilization handshake

    // Serial.print("WiiChuckDemo ready\n");

    // Pinos para controle para UP e DOWN  
    pinMode(11, OUTPUT);   
    pinMode(12, OUTPUT); 

    // Pinos para controle para RIGHT e LEFT  
    pinMode(9, OUTPUT);   
    pinMode(10, OUTPUT); 

    // GARRAS / GRIPPER  
    pinMode(2, OUTPUT);  
    pinMode(3, OUTPUT);  
}

void loop()  
{  
        nunchuck_get_data();

        // Data from Gyroscope   
        accx  = nunchuck_accelx(); // ranges from approx 70 - 182  
        accy  = nunchuck_accely(); // ranges from approx 65 - 173  
        // DData from Buttoms 
        zbut = nunchuck_zbutton();  
        cbut = nunchuck_cbutton();   
        // Data from Joystick
        joyx = nunchuck_joyx();  
        joyy = nunchuck_joyy();

        // Serial.print("accx: "); Serial.print((byte)accx,DEC);  
        // Serial.print("\taccy: "); Serial.print((byte)accy,DEC);  
        // Serial.print("\tzbut: "); Serial.print((byte)zbut,DEC);  
        // Serial.print("\tcbut: "); Serial.println((byte)cbut,DEC);  
        // Serial.print("\tJoy.X: "); Serial.print((byte)joyx, DEC);  
        // Serial.print("\tJoy.Y: "); Serial.println((byte)joyy, DEC);  
          
        // DOWN movement  
        if ((accy < 115) || (joyy > 150)) {  
           // digitalWrite(12, HIGH);  
           digitalWrite(12,1);  
           digitalWrite(11,0);             
        }  
        // UP moviment  
        if ((accy > 150) || (joyy < 90)) {  
           // digitalWrite(11, HIGH);  
           digitalWrite(12,0);  
           digitalWrite(11,1);  
        }  
        if ((accy < 150) && (accy > 115) && (joyy > 120) && (joyy < 150)) {  
           // digitalWrite(12, LOW);  
           // digitalWrite(11, LOW);  
           digitalWrite(12,0);  
           digitalWrite(11,0);  
        }

        // LEFT movement  
        if ((accx < 100) || (joyx < 90)) {    
           digitalWrite(9,0);  
           digitalWrite(10,1);  
        }  
        // RIGHT movement 
        if ((accx > 160) || (joyx > 150)) {  
           digitalWrite(9,1);  
           digitalWrite(10,0);  
        }  
        if ((accx < 160) && (accx > 100) && (joyx > 120) && (joyx < 150)) {  
           digitalWrite(9,0);  
           digitalWrite(10,0);  
        }

        // CLOSE the FINGERs 
        if (zbut == 1) {  
           digitalWrite(2,1);  
           digitalWrite(3,0);  
        }  
        if (cbut == 1) {  
           digitalWrite(2,0);  
           digitalWrite(3,1);  
        }  
        if ((zbut == 0) && (cbut == 0)) {  
           digitalWrite(2,0);  
           digitalWrite(3,0);  
        }

    delay(100);  
}
```

