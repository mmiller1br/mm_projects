# ESPHOME + Prometheus + Grafana
🌡️💦📊💻

#esphome #sensor #weather #esp32 #prometheus #grafana


## Description:

This is a variant of my project 07. The difference is the ESPHOME software used on ESP32 development board, that makes the integration with HomeAssistant much easier. 

On my previous project, in order to export all the IoT data to an external database, I was using Influxdb. And, to create the dashboards, Grafana, of course. Unfortunately, with ESPHOME there is not an easy way to integrate with influxdb. I've tried HTTP requests to call influxdb APIs, but that was not working very well. Plan B in action, let's replace influxdb to Prometheus: enable prometheus on ESPHOME, read all the values using a prometheus server, and use Grafana to create all the graphs.

Final result, OLED display shows temperature, humidity and clock (NTP synchronized), changing every 20s. Data is available through Prometheus integration. Grafana read data from Prometheus and display the graphs. Also, these data is available on HomeAssistant for any kind of automation. 

## ESPHOME code

```yaml
esphome:
  name: my-esp-test
  friendly_name: my-esp-test

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "1234567890asdfghjkl0987654321"

ota:
  password: "0486fcfb29e109c580be1efce7f7440f"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "My-Esp-Test Fallback Hotspot"
    password: "bHefeIyfgIjC"

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

# DHT11 sensor connected to pin GPIO18
sensor:
  - platform: dht
    pin: GPIO18
    temperature:
      name: "ESP-DHT11 Temperature"
      id: mysensort
    humidity:
      name: "ESP-DHT11 Humidity"
      id: mysensorh
    update_interval: 20s

# Example configuration NTP
time:
  - platform: sntp
    id: sntp_time
    timezone: America/Toronto
    servers:
     - 0.pool.ntp.org
     - 1.pool.ntp.org
     - 2.pool.ntp.org

# Example configuration entry
i2c:
  sda: GPIO21
  scl: GPIO22
  frequency: 800kHz

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

## Prometheus Integration

With the code above, data is available using the following URL: http://esphome_ip_address/metrics

```
#TYPE esphome_sensor_value GAUGE
#TYPE esphome_sensor_failed GAUGE
esphome_sensor_failed{id="esp-dht11_temperature",name="ESP-DHT11 Temperature"} 0
esphome_sensor_value{id="esp-dht11_temperature",name="ESP-DHT11 Temperature",unit="°C"} 23.4
esphome_sensor_failed{id="esp-dht11_humidity",name="ESP-DHT11 Humidity"} 0
esphome_sensor_value{id="esp-dht11_humidity",name="ESP-DHT11 Humidity",unit="%"} 74
```

And here is the Prometheus Server config to read these information/values from ESPHOME. This is the content on the file config.yml on Prometheus server:

```
scrape_configs:
  - job_name: "esphome"
    basic_auth:
      username: admin
      password: changeme
    static_configs:
      - targets:
        - 192.168.5.60
```

Target is the IP address of your ESPHOME. In my case, because the web interface has username/password enabled, we need to use the same parameters. After that, data from ESPHOME is available on your Prometheus Server, and can be used as data source for your Grafana graphs.

