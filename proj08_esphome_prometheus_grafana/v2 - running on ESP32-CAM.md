When migrating to production (ESP32-CAM board), I had to do some changes in the code to make it work. Because there are 2 devices connected to the same I2C BUS, I had to play with the parameter ==frequency== in order to make it works. 
- high frequency (800Mhz) was not working
- low frequency (50Mhz by default), was working, but with an error message in the LOGs ("Component display took a long time for an operation")
- the value of 400MHz was one that worked perfect for me
- there is NO problem if using one 1 device connected to the I2C bus. Only when we have more than 1 sharing the same bus, we need to pay attention on the clock parameters. 

```yaml
esphome:
  name: esp01-office
  friendly_name: esp01-office

  
esp32:
  board: esp32dev
  framework:
    type: arduino
  
# Enable logging
logger: 

# Enable Home Assistant API
api:
  encryption:
    key: "cfNZEJRvZL+GzcfrO+jH5rIfKoqNmfT6GAfO1xcYdeU="  

ota:
  password: "9d3114c3af39a2d0666b5ffecc876e05" 

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp01-Office Fallback Hotspot"
    password: "b0C3yzkQdC3n" 

captive_portal:

# Enable HTP interface
web_server:
  port: 80
  local: true
  auth:
    username: admin
    password: changeme    

# Prometheus: activate /metrics endpoint
prometheus:

# Example configuration entry for ESP32 i2c BUS
i2c:
  sda: GPIO14
  scl: GPIO15
  scan: True
  # 800Mhz does NOT work, 50MHz has error MSGs in the LOGs, 400MHz works fine
  frequency: 400kHz

# DHT12 sensor connected to I2C PINs
sensor:
  - platform: am2320
    address: 0x5c
    temperature:
      name: "esp01-office-temperature"
      id: mysensort
    humidity:
      name: "esp01-office-humidity"
      id: mysensorh
    update_interval: 60s

# Example configuration NTP
time:
  - platform: sntp
    id: sntp_time
    timezone: America/Toronto
    servers:
     - 0.pool.ntp.org
     - 1.pool.ntp.org
     - 2.pool.ntp.org

# Fonts and Images used ob OLED display
font:
  - file: 'fonts/arial.ttf'
    id: font1
    size: 31
  - file: 'fonts/digital-mono.ttf'
    id: font2
    size: 46  

image:
  - file: mdi:water-percent
    id: iconhumid
    resize: 32x32
  - file: mdi:temperature-celsius
    id: icontemp
    resize: 32x32  

# OLED display SSD1306 config
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x32"
    id: my_display
    #reset_pin: D0
    address: 0x3C
    #update_interval: 20s
    pages:
      - id: page1
        lambda: |-
          it.printf(20, 0, id(font1), "%.1f", TextAlign::TOP_LEFT, id(mysensorh).state);
          it.image(85, 0, id(iconhumid));

      - id: page2
        lambda: |-
          it.printf(20, 0, id(font1), "%.1f", TextAlign::TOP_LEFT, id(mysensort).state);
          it.image(85, 0, id(icontemp));

      - id: page3
        lambda: |-
          it.strftime(5, 0, id(font2), "%H:%M", id(sntp_time).now());

# For example cycle through pages on a timer
interval:
  - interval: 20s
    then:
      - display.page.show_next: my_display
      - component.update: my_display
```