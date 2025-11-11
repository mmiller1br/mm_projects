## ESPHome Voice Assistant
Better than Alexa ! 
Integration with LLMs, local or in the cloud.
This is my version of a Voice Assistant Satellite.
Runs on an ESP32-S3 hardware + ESPHome. Can control everything on your HomeAssistant. And it works perfectly !

It's still a PROTOTYPE, so lots of room for improvement. Below is everything I've used to create this voice assistant. Happy to receive comments to improve it.

<img width="520" height="435" alt="image" src="https://github.com/user-attachments/assets/9ce04d25-1caa-479f-b41e-228b614ef413" />


## Diagram

This is a diagram I've used for this project:

<img width="523" height="354" alt="image" src="https://github.com/user-attachments/assets/a03bbe66-8523-4691-8c01-ad1e9cf5656e" />

https://app.cirkitdesigner.com/project/23d87c44-6578-4a26-a7a3-a77ac740e9cb

## List of components:

I've tried to use everything with low cost and easy to find. No need for PCB (it's a prototype), but designing a PCB for this project and improving the microphone and speakers would be a great upgrade. Here is the list:
- ESP32-S3 Development Board **N16R8** 
- Omnidirectional Microphone Module I2S Interface INMP441
- Sound Module MAX98357A I2S 3W Class D Amplifier
- 2PCS 3525/2535 Speaker 4Ohm 3W
- 1pcs WS2812 RGB LED Ring 16 LEDs
- 3pcs TTP223 Digital Sensor Module
- Rotating Magnetic Data Cable 540 Degrees Usb Fast Charge cable

## ESPHome code

```yaml
esphome:
  name: esp03-voice-assistant
  friendly_name: esp03-voice-assistant
  platformio_options:
    board_build.flash_mode: dio
  on_boot:
    - light.turn_on:
       id: led_ww
       blue: 100%
       brightness: 60%
       effect: Walking Ring

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf

    sdkconfig_options:
      CONFIG_ESP32S3_DEFAULT_CPU_FREQ_240: "y"
      CONFIG_ESP32S3_DATA_CACHE_64KB: "y"
      CONFIG_ESP32S3_DATA_CACHE_LINE_64B: "y"
      CONFIG_AUDIO_BOARD_CUSTOM: "y"

psram:
  mode: octal # Please change this to quad for N8R2 and octal for N16R8
  speed: 80MHz

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "cZ+3FrRNFhCV81234567890/UndyUtVQ="
  on_client_connected:
        then:
          - delay: 50ms
          - light.turn_off: led_ww
          - micro_wake_word.start:
  on_client_disconnected:
       then:
          - voice_assistant.stop: 

ota:
  - platform: esphome
    password: "6a8379a412345678907eca"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Voice-Assistant Fallback Hotspot"
    password: "password"

  power_save_mode: none
  enable_on_boot: True
  fast_connect: On
  output_power: 8.5

captive_portal:
    
# Enable HTTP interface
web_server:
  port: 80
  local: true
  auth:
    username: admin
    password: password

light:
  - platform: esp32_rmt_led_strip
    id: led_ww
    rgb_order: GRB
    pin: GPIO7
    num_leds: 16
    chipset: ws2812
    name: "My Ring"
    effects:
      - pulse:
          name: "Fast Pulse"
          transition_length: 0.5s
          update_interval: 0.5s
          min_brightness: 0%
          max_brightness: 60%

      - addressable_lambda:
          name: "Walking Ring"
          update_interval: 100ms
          lambda: |-
            static int pos = 0;
            it.all() = Color(0, 0, 0);
            it[pos] = Color(255, 255, 255);
            pos = (pos + 1) % it.size();

      - addressable_lambda:
          name: "Color Wipe"
          update_interval: 50ms
          lambda: |-
            static int pos = 0;
            it[pos] = Color(0, 0, 255);
            pos++;
            if (pos >= it.size()) {
              pos = 0;
              it.all() = Color(0, 0, 0);
            }    

      - addressable_lambda:
          name: "Volume Meter"
          update_interval: 500ms
          lambda: |-
            // Total number of LEDs
            int total = it.size();

            // Get current volume value (0.0 to 1.0)
            float vol = id(volume).state;

            // Calculate how many LEDs should be lit
            int active = int(round(vol * total));

            // Clear all LEDs first
            it.all() = Color(0, 0, 0);

            // Light up LEDs according to volume
            for (int i = 0; i < active; i++) {
              it[i] = Color(0, 0, 255);  // Blue color — change as desired
            }

i2s_audio:
  - id: i2s_in
    i2s_lrclk_pin: GPIO16 # WS
    i2s_bclk_pin: GPIO15 # SCK
  - id: i2s_out
    i2s_lrclk_pin: GPIO18 # LRC
    i2s_bclk_pin: GPIO17 # BCLK

microphone: # INMP441
  - platform: i2s_audio
    id: external_microphone
    adc_type: external
    i2s_audio_id: i2s_in
    i2s_din_pin: GPIO5 # SD
    channel: left
    pdm: false
    bits_per_sample: 32bit

speaker: # MAX98357A
  - platform: i2s_audio
    id: external_speaker
    i2s_audio_id: i2s_out
    dac_type: external
    i2s_dout_pin: GPIO9 # DIN
    channel: mono
    bits_per_sample: 32bit
    sample_rate: 16000

micro_wake_word:
  models:
    - model: okay_nabu
  on_wake_word_detected:
    - voice_assistant.start:
        wake_word: !lambda return wake_word;
        silence_detection: true
    - light.turn_on:
        id: led_ww           
        red: 20%
        green: 20%
        blue: 80%
        brightness: 50%
        effect: fast pulse 

number:
  - platform: template
    name: "VA Volume"
    id: volume
    unit_of_measurement: "%"
    min_value: 0
    max_value: 1
    step: 0.1
    mode: SLIDER
    update_interval: never
    optimistic: true
    restore_value: true
    initial_value: 0.5
    icon: "mdi:knob"
    entity_category: config
    on_value:
      - speaker.volume_set: !lambda "return x;" 

switch:
  - platform: template
    id: va_mute
    name: VA Mute
    optimistic: true
    on_turn_on: 
      - micro_wake_word.stop:
      - voice_assistant.stop:
      - light.turn_on:
          id: led_ww           
          red: 100%
          green: 0%
          blue: 0%
          brightness: 50%
          effect: fast pulse          
      - delay: 2s
      - light.turn_off:
          id: led_ww
      - light.turn_on:
          id: led_ww          
          red: 100%
          green: 0%
          blue: 0%
          brightness: 50%

    on_turn_off:
      - micro_wake_word.start:
      - light.turn_on:
          id: led_ww           
          red: 100%
          green: 100%
          blue: 100%
          brightness: 50%
          effect: fast pulse 
      - delay: 2s
      - light.turn_off:
          id: led_ww


binary_sensor:
  - platform: gpio
    id: va_mute_button # Physical Mute Button
    name: "VA Mute Button" 
    pin:
      number: GPIO10   # GPIO for Mute Button
      inverted: True
      mode:
        input: True
        pullup: True
    on_press: 
      then:
        - switch.toggle: va_mute

  - platform: gpio
    id: va_volume_button_plus # Physical Volume UP button
    name: "VA Volume+ Button" 
    pin:
      number: GPIO11   # GPIO for Mute Button
      inverted: True
      mode:
        input: True
        pullup: True
    on_press:
      - lambda: |-
          float new_value = id(volume).state + 0.1f;
          if (new_value > 1.0f) new_value = 1.0f;   // clamp to max
          id(volume).publish_state(new_value);
          on_turn_on: 
      - light.turn_on:
          id: led_ww           
          red: 100%
          green: 100%
          blue: 100%
          brightness: 50%
          effect: volume meter 
      - delay: 1s
      - light.turn_off:
          id: led_ww

  - platform: gpio
    id: va_volume_button_minus # Physical Volume DOWN button
    name: "VA Volume- Button" 
    pin:
      number: GPIO12   # GPIO for Mute Button
      inverted: True
      mode:
        input: True
        pullup: True
    on_press:
      - lambda: |-
          float new_value = id(volume).state - 0.1f;
          if (new_value < 0.0f) new_value = 0.0f;   // clamp to min
          id(volume).publish_state(new_value);
      - light.turn_on:
          id: led_ww           
          red: 100%
          green: 100%
          blue: 100%
          brightness: 50%
          effect: volume meter 
      - delay: 1s
      - light.turn_off:
          id: led_ww

voice_assistant:
  id: va
  microphone: external_microphone 
  speaker: external_speaker
  #use_wake_word: false
  noise_suppression_level: 2 # 0, 1, 2, 3, 4
  auto_gain: 31dBFS # 0dBFS - 31dBFS
  volume_multiplier: 4 # >0
  on_stt_end:
       then: 
         - light.turn_off: led_ww
  #        - light.turn_off: led_strip
  on_error:
          - micro_wake_word.start:  
  on_end:
        then:
          - light.turn_off: led_ww
          # - light.turn_off: led_strip
          - wait_until:
              not:
                voice_assistant.is_running:
          - micro_wake_word.start:  

# Restart button - option to restart esphome remotely
button:
  - platform: restart
    name: "esp03-voice-assistant Restart"
```

## HomeAssistant Config

There are a lot of integration and protocols that you need to configure on HomeAssistant in order to make it work. On YouTube you'll find many videos explaining this process. 

What you'll find here is how I configured the Voice Assistant settings, as well as ESPHome integration to make it work.

1) **Voice Assistant integration**: click on Settings (left panel) and then on Voice Assistant. This is how I created my assistant

<img width="452" height="911" alt="Pasted image 20251110192321" src="https://github.com/user-attachments/assets/26bd4f4d-3b2f-4f8d-bcf4-9d420bcd630b" />


For conversation agent, I'm using HomeAssistant.

For STT (Speech-to-Text), I'm using **Speech-to-Phrase** because it's faster and it runs better in my HomeAssistant - i'm using a Raspberry Pi 5. You'll need to install this ADD-ON first.

For TTS (Text-to-Speech), I'm using **ElevenLabs**. You can use Faster-Whisper as well. but ElevenLabs has some voice that I think it sounds more natural.

And for the Wake-Word, I'm using "Okay Nabu" - This is also another ADD-ON called openWakeWorld that you need to install.

And of course give it a name: "esp32-s3" in my case.


2) ESP-HOME integration: here is where you select the Voice Assistant created previously for your ESP32. You need to go to Settings - Devices and Services - ESPHome. Then you'll see a list of ESPs already integrated with your ESPHome. Click the name of yours (mine is esp03-voice-assistant)

<img width="1497" height="1837" alt="Pasted image 20251110192231" src="https://github.com/user-attachments/assets/a75c9e9d-2894-49d2-ac6c-47ce1bda35e8" />


Here, if everything was already created successfully, all you need to do is associate your Voice Assistant ("esp32-s3") to your ESPHome project. Now you can test it... :D 



https://github.com/user-attachments/assets/b14548ee-9f88-4f26-b7d0-1814624785ff



## 3D Files

https://makerworld.com/en/models/1957124-home-assistant-voice-assistant-cube#profileId-2103455


