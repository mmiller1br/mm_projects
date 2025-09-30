# ESPHome Weather Station 2 - LVGL
🌡️💦📊💻

#esphome #sensor #weather #esp32 #lvgl


## Description:

![proj10_gif](https://github.com/user-attachments/assets/24b7a2e0-2d93-4482-a29a-028591690566)


This is an upgrade of my original project with much more information and a much better display.  

It has 4 pages to present information about internal temperature and humidity, external temperature and humidity, a clock (getting info from NTP Server) and also weather forecast information from HomeAssistant integration [Meteorologisk institutt](https://www.home-assistant.io/integrations/met/).

## List of Equipments:

- ESP32 VROOM-32U with connector for external antenna
- Temp/Humid sensor model AM2320
- round display 1.28on model C9A01A

## Coding:

This project has some parts that will be described below:
- sensor reading (AM2320)
- color display 1.28in 240x240 resolution 
- LVGL library and pages

1) **Sensor Reading**: this project is using the sensor AM2320. It gives more accurate results than the basic sensor DHT11. 

```yaml
# Configuration entry for ESP32 i2c BUS
i2c:
  sda: GPIO23
  scl: GPIO22
  scan: false
  
# AM2320 sensor connected to I2C PINs
sensor:
  - platform: am2320
    address: 0x5c
    temperature:
      name: "esp07-office-temperature"
      id: mysensort
    humidity:
      name: "esp07-office-humidity"
      id: mysensorh
    update_interval: 30s
```

3) **OLED display**: the OLED display used here is the main difference comparing with the original project. The round display GC9A01 has a much better resolution and using it in conjunction with the LVGL library, it gives you a ton of options to design the way you want to display information on it.

For this project, i'm using 4 different pages, that are presented in a loop, where each page is displayed for 20s, and then goes to the next one. The pages are:

- **local temperature and humidity**: this is information from the local sensor attacjed to the ESP32 where this code is running - in my case, my office.
  
![proj10-img1](https://github.com/user-attachments/assets/64ae220f-9976-4231-be4e-3a0edd073bc1)


- **clock**: this is information coming from NTP server and it has hour, date and day of the week.

![proj10-img2](https://github.com/user-attachments/assets/2b10bb5e-ae7e-4119-901c-0b23c4922e75)


- **outdoor temperature**: this information come from an entity on HomeAssistant. I have another ESP32 with an external sensor attached to it, sending this information HA. We can read this information and display as it is directed connected to this esp32.

![proj10-img3](https://github.com/user-attachments/assets/dc0d5410-d3eb-4664-8917-b9693da075dc)


- **weather forecast**: this is also information from another entity on HomeAssistant. In this case this info comes from an integration responsible for reading this from internet and update the entity. All information here is dynamically update (weather condition, icon, MAX temperature and MIN for the current date).

![proj10-img4](https://github.com/user-attachments/assets/d04d6409-e721-4618-8f8a-59d757c4409f)


You can see all the code for this project on the file proj10_weather_station2.yml

Cheers.
