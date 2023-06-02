
```c++
//Carrega a biblioteca do sensor ultrassonico  
#include <Ultrasonic.h>

//Carrega a biblioteca do motor SERVO  
#include <Servo.h>  
   
//Define os pinos para o trigger e echo RIGHT  
#define Rpino_trigger 4  
#define Rpino_echo 5

//Define os pinos para o trigger e echo LEFT  
#define Lpino_trigger 7  
#define Lpino_echo 8

//Define e inicializa o servo e a variÃ¡vel de Ã¢ngulo  
Servo servo;  
int angle = 90;  
   
//Inicializa o sensor nos pinos definidos acima  
Ultrasonic ultrasonicR(Rpino_trigger, Rpino_echo);  
Ultrasonic ultrasonicL(Lpino_trigger, Lpino_echo);  
   
void setup()  
{  
  //Define pino de controle do SERVO  
  servo.attach(10);  
  servo.write(angle);

  Serial.begin(9600);  
  Serial.println("Lendo dados do sensor...");  
}  
   
void loop()  
{  
  //Le as informacoes do sensor, em cm e pol  
  float cmMsecR, inMsecR, cmMsecL, inMsecL;  
  long microsecR = ultrasonicR.timing();  
  long microsecL = ultrasonicL.timing();  
  cmMsecR = ultrasonicR.convert(microsecR, Ultrasonic::CM);  
  inMsecR = ultrasonicR.convert(microsecR, Ultrasonic::IN);  
  cmMsecL = ultrasonicL.convert(microsecL, Ultrasonic::CM);  
  inMsecL = ultrasonicL.convert(microsecL, Ultrasonic::IN);  
  //Exibe informacoes no serial monitor  
  //Serial.print("Distancia R: ");  
  //Serial.print(cmMsecR);  
  //Serial.print("cm, Distancia L: ");  
  //Serial.print(cmMsecL);  
  //Serial.print(",   angle: ");  
  //Serial.println(angle);  
    
  // Serial.print(" - Distancia em polegadas: ");  
  // Serial.println(inMsecR);  
  // delay(500);

  // sÃ³ ativa quando existe algum objeto a uma distÃ¢ncia < 50cm, em qualquer olho  
    if ((cmMsecR < 50) || (cmMsecL < 50)) {  
    
     // objeto mais a esquerda, vira para a esquerda  
     if ((cmMsecR > 50) && (cmMsecL < 50)) {  
        if (angle < 180) {  
           angle = angle + 5;  
           servo.write(angle);  
        }  
     }  
          // objeto mais a direita, vira para a direita  
     if ((cmMsecR < 50) && (cmMsecL > 50)) {  
        if (angle > 0) {  
          angle = angle - 5;  
          servo.write(angle);  
        }  
     }  
  }  
  delay(30);  
}
```

