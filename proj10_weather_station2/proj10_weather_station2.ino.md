
```yaml
esphome:
  name: esp07-office2
  friendly_name: esp07-office2

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "OCvDIrIZNXnJV41234567890"

ota:
  - platform: esphome
    password: "f9a11baab4e91234567890"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp07-Office2 Fallback Hotspot"
    password: "1234567890"

captive_portal:

# Enable HTTP interface
web_server:
  port: 80
  local: true
  auth:
    username: admin
    password: password
  
# Prometheus: activate /metrics endpoint
prometheus:
  
# Configuration NTP
time:
  - platform: sntp
    id: sntp_time
    timezone: America/Toronto
    servers:
     - 0.pool.ntp.org
     - 1.pool.ntp.org
     - 2.pool.ntp.org

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
  
  # Reading information from HomeAssistant Entitites  
  - platform: homeassistant
    id: ha_outdoor_temp
    entity_id: sensor.esp02_garage_esp02_outdoor_temperature
  
  - platform: homeassistant
    id: ha_weather
    entity_id: weather.home
    
text_sensor:
  - platform: homeassistant
    id: ha_weather_text
    entity_id: sensor.weather_text
  
  - platform: homeassistant
    id: ha_weather_icon
    entity_id: sensor.weather_icon
  
  - platform: homeassistant
    id: ha_weather_temp_cur
    entity_id: sensor.weather_cur_temp
  
  - platform: homeassistant
    id: ha_weather_temp_min
    entity_id: input_number.weather_temp_min
  
  - platform: homeassistant
    id: ha_weather_temp_max
    entity_id: input_number.weather_temp_max

font:
  - file: "fonts/materialdesignicons-webfont.ttf"
    id: my_mdi
    size: 55
    bpp: 4
    glyphs: [
      "\U000F0F55", # home-thermometer-outline
      "\U000F18D7", # sun-thermometer-outline
      "\U000F1A71", # snowflake-thermometer
      "\U000F058E", # water-percent
      ]
  - file: "fonts/materialdesignicons-webfont.ttf"
    id: my_mdi_weather
    size: 80
    bpp: 4
    glyphs: [
      "\U000F0594", # clear-night
      "\U000F0590", # cloudy
      "\U000F0F2F", # exceptional
      "\U000F0591", # fog
      "\U000F0592", # hail
      "\U000F0593", # lightning
      "\U000F067E", # lightning-rainy
      "\U000F0595", # partlycloudy
      "\U000F0F31", # partlycloudynight
      "\U000F0596", # pouring
      "\U000F0597", # rainy
      "\U000F0598", # snowy
      "\U000F067F", # snowy-rainy
      "\U000F0599", # sunny
      "\U000F059D", # windy
      "\U000F059E", # windy-variant
      "\U000F14E4", # sunny-off
      ]
  - file: "fonts/materialdesignicons-webfont.ttf"
    id: my_mdi_temp
    size: 30
    bpp: 4
    glyphs: [
      "\U000F072F", # arrow-DOWN
      "\U000F0738", # arrow-UP
      "\U000F0E0A", # water-outline
      ]
  - file: 'fonts/arial.ttf'
    id: font1
    size: 70
  - file: 'fonts/arial.ttf'
    id: font2
    size: 20
  - file: 'fonts/arial.ttf'
    id: font4
    size: 40
  - file: 'fonts/london-font.ttf'
    id: font3
    size: 70
  
color:
  - id: my_red
    red: 100%
  - id: my_green
    green: 100%
  - id: my_blue
    blue: 100%
  - id: my_blue2
    hex: 167BBD
  - id: my_yellow
    hex: FFFF00
  - id: my_orange
    hex: FFA500
  - id: my_white
    hex: FFFFFF
  - id: my_gray
    hex: 555555
  
image:
  - file: mdi:water-percent
    id: iconhumid
    resize: 30x30
    type: rgb
  - file: mdi:thermometer
    id: icontemp
    resize: 30x30
    type: rgb
  - file: mdi:home-thermometer-outline
    id: icontemphome
    resize: 30x30
    type: rgb


# SPI BUS config
spi:
  clk_pin: GPIO32
  mosi_pin: GPIO33

# Example minimal configuration entry
display:
  - platform: ili9xxx
    id: my_display
    model: GC9A01A
    dc_pin: GPIO27
    reset_pin: GPIO25
    cs_pin: GPIO26
    invert_colors: true
    auto_clear_enabled: false
    #rotation: 90
    transform:
      swap_xy: true
      #mirror_x: true
    update_interval: never

output:
  - platform: ledc
    pin: GPIO12
    id: gpio_12
light:
  - platform: monochromatic 
    id: my_display_backlight
    output: gpio_12
    name: esp07-backlight
    restore_mode: RESTORE_AND_ON
    on_turn_on: 
      then:
        - light.turn_on: 
            id: my_display_backlight
            brightness: 50%

lvgl:
  buffer_size: 25%
  log_level: INFO
  color_depth: 16
  bg_color: black
  border_width: 0
  outline_width: 0
  shadow_width: 0
  align: CENTER

  displays:
    - my_display
    
  pages:
    - id: page1_TEMP
      widgets:
        - arc:
            height: 95%
            width: 95%
            arc_width: 14
            arc_color: dimgray
            border_width: 0
            bg_opa: TRANSP
            align: CENTER
            id: arc_id1
            pad_all: 0
            value: 40
            min_value: 0
            max_value: 40
            adjustable: true
            indicator:
              arc_color: orangered
              arc_width: 14
            knob:
              bg_color: orangered
              width: 28
            widgets:
              - label:
                  id: temperature_text
                  text_font: font1
                  text_color: snow
                  text: "-.-"
                  align: CENTER
              - label:
                  text: "\U000F0F55"
                  align: CENTER
                  text_font: my_mdi
                  text_color: darkorange
                  y: 70
              - label:
                  text_font: my_mdi_temp
                  text_color: aqua
                  align: CENTER
                  text: "\U000F0E0A"
                  x: -15
                  y: -55
              - label:
                  id: humidity_text
                  text_font: font2
                  text_color: ghostwhite
                  align: CENTER
                  text: "-.-"
                  x: 15
                  y: -55

    - id: page3_CLOCK
      widgets:
        - obj:
            height: 240
            width: 240
            align: CENTER
            bg_color: black
            border_width: 0
            pad_all: 4
            widgets:
              - meter:
                  height: 100%
                  width: 100%
                  border_width: 0
                  bg_opa: TRANSP
                  align: CENTER
                  scales:
                    - range_from: 0
                      range_to: 360
                      angle_range: 360 # sets the total angle to 180 = starts mid left and ends mid right
                      rotation: 270
                      indicators:
                        - arc: # first half of the scale background
                            color: mediumblue
                            r_mod: 12 # radius difference from the scale default radius
                            width: 6
                            start_value: 20
                            end_value: 160
                        - arc: # second half of the scale background
                            color: deeppink
                            r_mod: 12
                            width: 6
                            start_value: 200
                            end_value: 340
              - obj: # to cover the middle part of meter indicator line
                  height: 146
                  width: 146
                  radius: 73
                  align: CENTER
                  border_width: 0
                  bg_color: black
                  pad_all: 0

              - label:
                  id: my_date
                  text_font: montserrat_16
                  text_color: snow
                  text: " TEST "
                  align: CENTER
                  y: -60

              - label:
                  id: my_weekday
                  text_font: montserrat_18
                  text_color: black
                  bg_opa: 1
                  bg_color: lime
                  align: CENTER
                  y: 50
                  text: " DAY "

              - label:
                  id: my_clock
                  text_font: font3
                  text_color: snow
                  text: "-.-"
                  align: CENTER


    - id: page4_OUTDOOR_TEMP
      widgets:
        - arc:
            height: 95%
            width: 95%
            arc_width: 14
            arc_color: dimgray
            border_width: 0
            bg_opa: TRANSP
            align: CENTER
            id: arc_id3
            pad_all: 0
            value: 60
            min_value: -20
            max_value: 50
            adjustable: true
            indicator:
              arc_color: limegreen
              arc_width: 14
            knob:
              bg_color: limegreen
              width: 28
            widgets:
              - label:
                  id: temperature_out_text
                  text_font: font1
                  text_color: snow
                  text: "-.-"
                  align: CENTER
              - label:
                  text: "\U000F1A71"
                  align: CENTER
                  text_font: my_mdi
                  text_color: gold
                  y: 70
              - label:
                  text_font: font2
                  text_color: ghostwhite
                  align: CENTER
                  #bg_opa: 1
                  #bg_color: ghostwhite
                  text: " outdoor "
                  y: -55


    - id: page5_WEATHER
      widgets:
        - obj:
            height: 240
            width: 240
            align: CENTER
            bg_color: black
            border_width: 0
            pad_all: 4
            widgets:
              - meter:
                  height: 100%
                  width: 100%
                  border_width: 0
                  bg_opa: TRANSP
                  align: CENTER
                  scales:
                    - range_from: 0
                      range_to: 360
                      angle_range: 360 # sets the total angle to 180 = starts mid left and ends mid right
                      rotation: 180
                      indicators:
                        - arc: # first half of the scale background
                            color: orangered
                            r_mod: 12 # radius difference from the scale default radius
                            width: 4
                            start_value: 10
                            end_value: 170
                        - arc: # second half of the scale background
                            color: royalblue
                            r_mod: 12
                            width: 4
                            start_value: 190
                            end_value: 350
              - obj: # to cover the middle part of meter indicator line
                  height: 146
                  width: 146
                  radius: 73
                  align: CENTER
                  border_width: 0
                  bg_color: black
                  pad_all: 0

              - label:
                  id: my_weather_text
                  text: " "
                  align: CENTER
                  text_font: font2
                  text_color: white
                  #bg_opa: 1
                  #bg_color: dimgray
                  #x: -55
                  y: -60

              - label:
                  id: my_weather_icon
                  text: "\U000F14E4"
                  align: CENTER
                  text_font: my_mdi_weather
                  text_color: white
                  x: -55
                  y: 15

              - label:
                  id: my_weather_temp_cur
                  text: ha_weather_cur_temp
                  align: LEFT_MID
                  text_font: font4
                  text_color: yellow
                  x: 115
                  y: -15

              - label:
                  text: "\U000F072F" # arrow DOWN = MIN
                  align: LEFT_MID
                  text_font: my_mdi_temp
                  text_color: deepskyblue
                  x: 120
                  y: 25
              - label:
                  id: my_weather_temp_min
                  text: ha_weather_temp_min
                  align: LEFT_MID
                  text_font: font2
                  text_color: white
                  x: 155
                  y: 25

              - label:
                  text: "\U000F0738" # arrow UP = MAX
                  align: LEFT_MID
                  text_font: my_mdi_temp
                  text_color: deepskyblue
                  x: 120
                  y: 60
              - label:
                  id: my_weather_temp_max
                  text: ha_weather_temp_max
                  align: LEFT_MID
                  text_font: font2
                  text_color: white
                  x: 155
                  y: 60

interval:
  - interval: 20s
    then:
      - lvgl.page.next:

      # REFRESH PAGE1 - THERMOSTAT INDOOR
      - lvgl.arc.update:   # update ARC for temperature
          id: arc_id1
          value: !lambda return id(mysensort).state;
      - lvgl.label.update:  # update TEXT for temperature
          id: temperature_text
          text:
            format: "%.1f°"
            args: 
              - id(mysensort).state
      - lvgl.label.update:  # update TEXT for humidity
          id: humidity_text
          text:
            format: "%.0f%%"
            args: 
              - id(mysensorh).state

      # REFRESH PAGE3 - THERMOSTAT OUTDOOR    
      - lvgl.arc.update:   # update ARC for temperature OUTDOOR
          id: arc_id3
          value: !lambda return id(ha_outdoor_temp).state;
      - lvgl.label.update:  # update TEXT for temeprature OUTDOOR
          id: temperature_out_text
          text:
            format: "%.1f°"
            args: 
              - id(ha_outdoor_temp).state              

      # REFRESH PAGE4 - CLOCK
      - lvgl.label.update:  # update TEXT for CLOCK            
          id: my_clock
          text: 
            time_format: "%H:%M"
            time: !lambda return (id(sntp_time).now());
      - lvgl.label.update:
          id: my_date
          text:
            format: "%s %2d"
            args:
              - '(new const char *[12]{"JAN", "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "DEC"})[id(sntp_time).now().month-1]'
              - 'id(sntp_time).now().day_of_month'
      - lvgl.label.update:
          id: my_weekday
          text:
            format: "%s"
            args:
              - '(new const char *[7]{" SUN ", " MON ", " TUE ", " WED ", " THU ", " FRI ", " SAT "})[id(sntp_time).now().day_of_week-1]'

      # REFRESH PAGE5 - WEATHER STATION
      - lvgl.label.update:
          id: my_weather_text
          text: !lambda 'return id(ha_weather_text).state;'
      - lvgl.label.update:
          id: my_weather_icon
          text: !lambda 'return id(ha_weather_icon).state;'
      - lvgl.label.update:
          id: my_weather_temp_cur
          text: !lambda 'return id(ha_weather_temp_cur).state + "°";'
      - lvgl.label.update:
          id: my_weather_temp_min
          text: !lambda 'return id(ha_weather_temp_min).state;'      
      - lvgl.label.update:
          id: my_weather_temp_max
          text: !lambda 'return id(ha_weather_temp_max).state;'


# Restart button - option to restart esphome remotely
button:
  - platform: restart
    name: "esp07-office2 Restart"home:
  name: esp07-office2
  friendly_name: esp07-office2
  
```