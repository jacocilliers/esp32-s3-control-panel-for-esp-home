# esp32-s3-control-panel-for-esp-home
Control panel using esp home and Waveshare ESP32-S3-Touch-LCD

Projects Overview

WaveShare ESP32-S3-4.3" LCD Control Panel
This YAML configuration transforms a WaveShare ESP32-S3-4.3" LCD into a versatile control panel. Features include:

Linked action buttons: Control multiple devices in your smart home with on-screen touch buttons.
Real-time feedback: Displays the status of controlled devices.
Customizable interface: Adjust layout and functionality to suit your needs.
Clear, large font: Optimized for readability.
Customizable design: Adapt font sizes, colors, or layouts as needed.

Getting Started

Prerequisites
ESPHome installed on your local machine or accessible via Home Assistant.
Compatible ESP devices:
WaveShare ESP32-S3-4.3" LCD for the control panel.
Basic knowledge of YAML configuration and Home Assistant integration.
Setup Instructions
Copy the desired YAML configuration(s) to your ESPHome directory.
Modify the YAML file to match your hardware setup:
For the WaveShare LCD configurations, adjust the layout and actions to suit your needs.
Upload the configuration to your ESP device using ESPHome.


esphome:
  name: "
  friendly_name: "
  platformio_options:
    build_flags: "-DBOARD_HAS_PSRAM"
    board_build.flash_mode: dio
    board_upload.maximum_ram_size: 524288

esp32:
  board: esp32-s3-devkitc-1
  variant: esp32s3
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_ESP32S3_DEFAULT_CPU_FREQ_240: y
      CONFIG_ESP32S3_DATA_CACHE_64KB: y
      CONFIG_SPIRAM_FETCH_INSTRUCTIONS: y
      CONFIG_SPIRAM_RODATA: y

# Enable logging
logger:


i2c:
  sda: GPIO08
  scl: GPIO09
  id: bus_a

ch422g:
  - id: ch422g_hub

# Enable Home Assistant API
api:
  encryption:
    key: ""

ota:
  - platform: esphome
    password: ""

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: ""
    password: ""

captive_portal:

psram:
  mode: octal
  speed: 80MHz

# Home Assistant Switch Templates
switch:
  - platform: homeassistant
    id: living_room_switch_status
    entity_id: switch.living_room
    internal: true
  - platform: homeassistant
    id: tv_switch_status
    entity_id: switch.tv
    internal: false
  - platform: homeassistant
    id: lamp_switch_status
    entity_id: switch.lamp
    internal: false
  - platform: homeassistant
    id: bathroom_switch_status
    entity_id: switch.bathroom
    internal: false
  - platform: homeassistant
    id: garage_switch_status
    entity_id: switch.garage
    internal: false
  - platform: homeassistant
    id: garage_door_switch_status
    entity_id: switch.garage_door
    internal: false

# Define display
display:
  - platform: rpi_dpi_rgb
    id: my_display
    auto_clear_enabled: false
    color_order: RGB
    pclk_frequency: 16MHZ
    dimensions:
      width: 800
      height: 480
    de_pin:
      number: 5
    hsync_pin:
      number: 46
      ignore_strapping_warning: true
    vsync_pin:
      number: 3
      ignore_strapping_warning: true
    pclk_pin: 7
    reset_pin:
      ch422g: ch422g_hub
      number: 3
    enable_pin:
      ch422g: ch422g_hub
      number: 2  
    hsync_back_porch: 16
    hsync_front_porch: 16
    hsync_pulse_width: 8
    vsync_back_porch: 16
    vsync_front_porch: 16
    vsync_pulse_width: 8
    data_pins:
      red:
        - 1         #r3
        - 2         #r4
        - 42        #r5
        - 41        #r6
        - 40        #r7
      blue:
        - 14        #b3
        - 38        #b4
        - 18        #b5
        - 17        #b6
        - 10        #b7
      green:
        - 39        #g2
        - 0         #g3
        - 45        #g4
        - 48        #g5
        - 47        #g6
        - 21        #g7

touchscreen:
  platform: gt911
  id: my_touch
  interrupt_pin: GPIO4
  reset_pin:
    ch422g: ch422g_hub
    number: 1
    mode: OUTPUT
  on_touch:
    - lambda: |-
        ESP_LOGI("Touch", "Touch detected at x=%d, y=%d", touch.x, touch.y);

# Define fonts
font:
  - file: "gfonts://Roboto"
    id: roboto24
    size: 24
    bpp: 4
  - file: "gfonts://Roboto"
    id: roboto18
    size: 18
    bpp: 4

lvgl:
  displays:
    - my_display
  touchscreens:
    - my_touch
  theme:
    button:
      bg_color: 0x333333
      bg_grad_dir: VER
      text_color: 0xFFFFFF
      checked:
        bg_color: 0x0000FF  # Bright Blue when checked/ON
        text_color: 0xFFFFFF
        bg_grad_color: 0x000080  # Darker blue gradient
      pressed:
        border_color: 0xff6600
  pages:
    - id: main_page
      layout:
        type: flex
        flex_flow: column_wrap
      width: 100%
      bg_color: 0x000000
      bg_opa: cover
      pad_all: 5
      widgets:

        # Living Room Button
        - button:
            checkable: true
            id: lv_button_living_room
            width: 252
            height: 225
            widgets:
              - label:
                  text_font: roboto24
                  text: "Living\nRoom\nLight"
              - label:
                  id: living_room_label
                  text: "switch"
                  align: bottom_left
            on_click:
              then:
                - homeassistant.service:
                    service: switch.toggle
                    data:
                      entity_id: switch.living_room    # link your own entities here

        # Lamp Button
        - button:
            checkable: true
            id: lv_button_lamp
            width: 252
            height: 225
            widgets:
              - label:
                  text_font: roboto24
                  text: "Lamp\nLight"
              - label:
                  id: lamp_label
                  text: "switch"
                  align: bottom_left
            on_click:
              then:
                - homeassistant.service:
                    service: switch.toggle
                    data:
                      entity_id: switch.lamp   # link your own entities here

        # Tv Button
        - button:
            checkable: true
            id: lv_button_tv
            width: 252
            height: 225
            widgets:
              - label:
                  text_font: roboto24
                  text: "TV\nLight"
              - label:
                  id: tv_label
                  text: "switch"
                  align: bottom_left
            on_click:
              then:
                - homeassistant.service:
                    service: switch.toggle
                    data:
                      entity_id: switch.tv   # link your own entities here

        # Garage Button
        - button:
            checkable: true
            id: lv_button_garage
            width: 252
            height: 225
            widgets:
              - label:
                  text_font: roboto24
                  text: "Garage\nLight"
              - label:
                  id: garage_label
                  text: "switch"
                  align: bottom_left
            on_click:
              then:
                - homeassistant.service:
                    service: switch.toggle
                    data:
                      entity_id: switch.garage   # link your own entities here

        # Bathroom Button
        - button:
            checkable: true
            id: lv_button_bathroom
            width: 252
            height: 225
            widgets:
              - label:
                  text_font: roboto24
                  text: "Bathroom\nLight"
              - label:
                  id: bathroom_label
                  text: "switch"
                  align: bottom_left
            on_click:
              then:
                - homeassistant.service:
                    service: switch.toggle
                    data:
                      entity_id: switch.bathroom     # link your own entities here



        # Garage Door Button
        - button:
            checkable: true
            id: lv_button_garage_door
            width: 252
            height: 225
            widgets:
              - label:
                  text_font: roboto24
                  text: "Garage\nDoor"
              - label:
                  id: garage_door_label
                  text: "switch"
                  align: bottom_left
            on_click:
              then:
                - homeassistant.service:
                    service: switch.toggle
                    data:
                      entity_id: switch.demo  # link your own entities here
                      
interval:
  - interval: 10s
    then:
      - lambda: |-
          auto update_button_state = [](bool state, lv_obj_t* button, lv_obj_t* label) {
            if (state) {
              lv_obj_add_state(button, LV_STATE_CHECKED);
              lv_label_set_text(label, "On");
            } else {
              lv_obj_clear_state(button, LV_STATE_CHECKED);
              lv_label_set_text(label, "Off");
            }
          };

          update_button_state(id(living_room_switch_status).state, id(lv_button_living_room), id(living_room_label));
          update_button_state(id(tv_switch_status).state, id(lv_button_tv), id(tv_label));
          update_button_state(id(lamp_switch_status).state, id(lv_button_lamp), id(lamp_label));
          update_button_state(id(bathroom_switch_status).state, id(lv_button_bathroom), id(bathroom_label));
          update_button_state(id(garage_switch_status).state, id(lv_button_garage), id(garage_label));
          update_button_state(id(garage_door_switch_status).state, id(lv_button_garage_door), id(garage_door_label));
