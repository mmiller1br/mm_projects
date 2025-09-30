```yaml
esphome:
  name: esp08-rack-monitor
  friendly_name: esp08-rack-monitor

esp32:
  board: esp32dev
  framework:
    type: arduino
    

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "V2Vnwub1+212345556666f9R7u9A77J9P3WfnWxcxcxc="

ota:
  - platform: esphome
    password: "f2e1001223345567ea3e1fae3275"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp08-Rack-Monitor"
    password: "changeme"

captive_portal:

# Enable HTTP interface
web_server:
  port: 80
  local: true
  auth:
    username: admin
    password: changeme    

# Retrieve SNMP component from github
external_components:
  - source: github://mmiller1br/esphome-snmp
   
# Homeassistant ENTITY for Power Monitor
text_sensor:
  - platform: homeassistant
    id: ha_power_monitor_power
    entity_id: sensor.rack1_power_monitor_power
  - platform: homeassistant
    id: ha_power_monitor_amp
    entity_id: sensor.rack1_power_monitor_current

# Enable SNMP component
snmp:
  contact: mmiller
  location: LAB
  read_community: "public123"
  #write_community: "my_write_string"
  temperature: aht20_temp
  humidity: aht20_humidity


font:
  - file: "fonts/materialdesignicons-webfont.ttf"
    id: my_mdi_icon
    size: 55
    bpp: 4
    glyphs: [
      "\U000F0F55", # home-thermometer-outline
      "\U000F18D7", # sun-thermometer-outline
      "\U000F050F", # thermometer
      "\U000F1A71", # snowflake-thermometer
      "\U000F058E", # water-percent
      "\U000F0E0A", # water-outline
      "\U000F0200", # ethernet
      "\U000F1903", # home-lightning-power
      ]
  - file: "fonts/materialdesignicons-webfont.ttf"
    id: my_mdi_icon2
    size: 30
    bpp: 4
    glyphs: [
      "\U000F0200", # ethernet
      "\U000F0E0A", # water-outline
      "\U000F050F", # thermometer
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

i2c:
  sda: GPIO25
  scl: GPIO26
  scan: true
  id: bus_a

sensor:
  # - platform: bmp280_i2c
  #   address: 0x77
  #   update_interval: 60s
  #   temperature:
  #     name: "BMP temp"
  #     oversampling: 2x
  #   pressure:
  #     name: "BMP Pressure"

  - platform: aht10
    variant: AHT20
    address: 0x38
    update_interval: 60s
    temperature:
      name: "esp08-rack-temperature"
      id: aht20_temp
      accuracy_decimals: 1
      filters:
      - filter_out: 0.0
      - median:
          window_size: 3
          send_every: 3
          send_first_at: 1
    humidity:
      name: "esp08-rack-humidity"
      id: aht20_humidity
      accuracy_decimals: 1
      filters:
      - filter_out: 0.0
      - median:
          window_size: 3
          send_every: 3
          send_first_at: 1


# SPI BUS config
spi:
  clk_pin: GPIO12
  mosi_pin: GPIO14

# Round Display config
display:
  - platform: ili9xxx
    id: my_display
    model: GC9A01A
    dc_pin: GPIO13
    reset_pin: GPIO32
    cs_pin: GPIO33
    invert_colors: true
    auto_clear_enabled: false
    update_interval: never
    transform:
      swap_xy: true
      mirror_x: true
      mirror_y: true

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
                  text_color: silver
                  text: "-.-"
                  align: CENTER
              - label:
                  text: "\U000F0200"
                  align: CENTER
                  text_font: my_mdi_icon
                  text_color: darkorange
                  y: 70
              - label:
                  text_font: my_mdi_icon2
                  text_color: dodgerblue
                  align: CENTER
                  text: "\U000F0E0A"
                  x: -20
                  y: -55
              - label:
                  id: humidity_text
                  text_font: font2
                  text_color: ghostwhite
                  align: CENTER
                  text: "-.-"
                  x: 15
                  y: -55

    - id: page2_HUMID
      widgets:
        - arc:
            height: 95%
            width: 95%
            arc_width: 14
            arc_color: dimgray
            border_width: 0
            bg_opa: TRANSP
            align: CENTER
            id: arc_id2
            pad_all: 0
            value: 40
            min_value: 0
            max_value: 100
            adjustable: true
            indicator:
              arc_color: royalblue
              arc_width: 14
            knob:
              bg_color: royalblue
              width: 28
            widgets:
              - label:
                  id: humidity2_text
                  text_font: font1
                  text_color: silver
                  align: CENTER
                  text: "-.-"
              - label:
                  text_font: my_mdi_icon
                  text_color: midnightblue
                  align: CENTER
                  text: "\U000F0200"
                  y: 70
              - label:
                  id: temperature2_text
                  text_font: font2
                  text_color: ghostwhite
                  text: "-.-"
                  align: CENTER
                  x: -10
                  y: -55
              - label:
                  text_font: my_mdi_icon2
                  text_color: crimson
                  align: CENTER
                  text: "\U000F050F"
                  x: 25
                  y: -55

    - id: page3_POWER
      widgets:
        - label:
            id: power_text
            text_font: font1
            text_color: silver
            align: CENTER
            text: "-.-"
        - label:
            text_font: my_mdi_icon
            text_color: green
            align: CENTER
            text: "\U000F1903"
            y: 70
        - label:
            id: amp_text
            text_font: font2
            text_color: ghostwhite
            text: "-.-"
            align: CENTER
            #x: -10
            y: -55

    - id: page4_BLANK
      skip: true
      widgets:


interval:
  - interval: 10s
    then:
      - if:
          condition:
            lambda: 'return id(esp08_show_display) == true;'
          then:
            - lvgl.page.next:
                animation: FADE_IN
                time: 1000ms

            # REFRESH PAGE1 - TEMPERATURE
            - lvgl.arc.update:   # update ARC for temperature
                id: arc_id1
                value: !lambda return id(aht20_temp).state;
            - lvgl.label.update:  # update TEXT for temperature
                id: temperature_text
                text:
                  format: "%.1f°"
                  args: 
                    - id(aht20_temp).state
            - lvgl.label.update:  # update TEXT for humidity
                id: humidity_text
                text:
                  format: "%.0f%%"
                  args: 
                    - id(aht20_humidity).state

            # REFRESH PAGE2 - HUMIDITY
            - lvgl.arc.update:   # update ARC for humidity
                id: arc_id2
                value: !lambda return id(aht20_humidity).state;
            - lvgl.label.update:  # update TEXT for temperature
                id: temperature2_text
                text:
                  format: "%.1f°"
                  args: 
                    - id(aht20_temp).state
            - lvgl.label.update:  # update TEXT for humidity
                id: humidity2_text
                text:
                  format: "%.0f%%"
                  args: 
                    - id(aht20_humidity).state

            # REFRESH PAGE3 - POWER
            - lvgl.label.update:  # update TEXT for POWER monitor
                id: power_text
                text: !lambda 'return id(ha_power_monitor_power).state + "W";'
            - lvgl.label.update:  # update TEXT for AMP monitor
                id: amp_text
                # text: !lambda 'return id(ha_power_monitor_amp).state + " A";'
                text: !lambda |-
                  char buf[16];
                  snprintf(buf, sizeof(buf), "%.1f A", atof(id(ha_power_monitor_amp).state.c_str()));
                  return std::string(buf);

          else:
            - lvgl.page.show:
                id: page4_BLANK


# Restart button - option to restart esphome remotely
button:
  - platform: restart
    name: "esp08-restart-button"


#esp32_touch:
binary_sensor:
  - platform: gpio
    id: esp08_touch_button
    name: "esp08-touch-button"
    pin: GPIO23
    on_press:
      then:
        - lambda: |-
            id(esp08_show_display) = !id(esp08_show_display); 
            ESP_LOGD("main", "Boolean flag SHOW_DISPLAY is now: %s", id(esp08_show_display) ? "true" : "false");

        - if:
            condition:
              lambda: |-
                return id(esp08_show_display);
            then:
              - lvgl.page.show:
                  id: page1_TEMP
            else:
              - lvgl.page.show:
                  id: page4_BLANK

globals:
  - id: esp08_show_display
    type: bool
    initial_value: 'true'
    restore_value: yes

```