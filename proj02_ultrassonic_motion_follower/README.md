
# Arduino Ultrassonic Motion Follower
👀🤖🚀

#arduino #sensor #ultrassonic

## Description:

This is an Arduino project - in this case a NANO - used in conjunction with 2 ultrassonic sensors SR04 and a servo motor SG90.

The ultrassonic sensor can be used to detect an object and measure the distance between the sensor and the object. This project used 2 sensors to detect an object and,  in case this object move to the right or to the left, the Arduino will move the servo motor and try to follow the object.

## List of Equipments:

- Arduino: if you don´t know what is an Arduino, you can start using these links: 

[https://www.arduino.cc/](https://www.arduino.cc/)  
[https://en.wikipedia.org/wiki/Arduino](https://en.wikipedia.org/wiki/Arduino)

- Ultrasonic Sensor HC-SR04: more details and how it works you can find here: 

[https://www.itead.cc/wiki/Ultrasonic_Ranging_Module_HC-SR04](https://www.itead.cc/wiki/Ultrasonic_Ranging_Module_HC-SR04)

- Servo Motor: a simple servo motor to be used with arduino to make the "head" movements. More details here: 

[https://www.arduino.cc/en/Reference/Servo](https://www.arduino.cc/en/Reference/Servo)


## Logic:

The 1st step is understand how the Ultrasonic Sensor works. Basically, it emits an ultrasound at 40 kHz which travels through the air and if there is an object or obstacle on its path it will bounce back to the module (echo). The echo will then tell us the distance travelled in microseconds. Considering the travel time and the speed of the sound you can calculate the distance.

Take a look at the diagram below:  

<img width="335" alt="proj02_image1" src="https://github.com/mmiller1br/mm_projects/assets/32887571/bc17cd90-3972-403a-9982-7489bc61ed34">


Now, let´s check the angle this Sensor use when transmiting the ultrasound wave  ... for best performance, the object must be between the 30 degree area, as you can check at the next diagram:

<img width="185" alt="proj02_image2" src="https://github.com/mmiller1br/mm_projects/assets/32887571/7d21ebd1-eb0e-45ae-89b5-26ca369ab1e3">


So, how to position 2 sensors to work like human eyes ? To illustrate this question, see the next diagram with 3 different examples.

- In the 1st example, they were installed in parallel - in this case, both have almost the same view angle (width), and it´s difficult to identify in which side a object is located (right or left) - DOES NOT WORK
- In the 2nd example, they were installed with a big angle between them - in this case, depending the size of this angle, we create a GAP in the middle of sensors, a place where a object is not reflecting the ultrasonic waves - DOES NOT WORK
- In the 3rd example, we have a small angle between the sensors, large enough to have at least 3 different "areas": one where the object is "seen" for both sensors (middle), one where the object is "seen" only for the left sensor (left eye) and the last where the object is reflecting only the ultrasonic waves from the right sensor (right eye).

<img width="468" alt="proj02_image3" src="https://github.com/mmiller1br/mm_projects/assets/32887571/846a2358-311a-4f5b-a5c8-4fde72b2afe9">

## Video

📽️ https://www.youtube.com/watch?v=APU4y4s7U1w

