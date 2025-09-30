# Network Rack Environment Monitor
💻

#network #sensor #esp32 #lvgl #esphome #snmp



![[Imagem do Rack1.jpg]]

### Introduction:

This idea came from one of my favourite Youtube channel, the Craft Computing (https://www.youtube.com/watch?v=bbxU_ZJRsPQ&t=777s). His words on this project: "*Axe Effect is an SNMP-enabled temperature sensor, designed to be simple and secure to deploy to your network environment, as well as delivering more accurate temperature data than most other solutions on the market today. It’s powered by a Raspberry Pi Pico-W RP2040, with a temperature sensor that is accurate out of the box to +/- 0.3C, and has a 0.01C fidelity, delivering smooth charting to your SNMP monitoring software.*"

I decided to implement this using an ESP32 board running ESPHOME. Once SNMP component is not available for this board, I decided to use one of the external components available (https://github.com/aquaticus/esphome-snmp), fork this repository, and modify it for my needs. Tks GROK for all your help ! :)

With this solution, now I have information about TEMPERATURE and HUMIDITY on my Rack, as well as power consumption. For Temp/Humid I'm using a **AHT20** sensor connected to the ESP32 board, and for Power Consumption, this data comes from a **Tuya Wifi Energy Monitor** sensor. And now I can read these data using 3 different forms:
- directed on the OLED display on my Environment Monitor module
- on HomeAssistant, displaying these info as numbers or graphs
- and on any NMS software (Network Management System), via SNMP protocol. In my case, i'm using LibreNMS runing in a docker container

### 3 Different Ways to Monitor

- via the 1RU Module with OLED display: i designed and 3D printed a 1xRU panel to my 10in rack with this OLED display - ESP32 board and sensors are all installed in the back of the panel
![[rack-monitor-ezgif.com-video-to-gif-converter.gif]]


- via HomeAssistant Dashboard (we have real-tie and historical information from my sensors)
![[Pasted image 20250930141828.png]]


- via LibreNMS (SNMP protocol network management tool). I configured my LibreNMS to read the monitor these values from my rack using SNMP. Here are the graphs.
![[Pasted image 20250930121353.png]]


### ESPHome SNMP component:

here is a link to my repo with this component, in case you want to use it in your projects:
https://github.com/mmiller1br/esphome-snmp


### Diagram:
![[Pasted image 20250915163346.png]]

