# Water meter for Apator16 built with ESP32 + CC1101 on ESPHome & Home Assistant
<img src="/images/Apator_AT-WMBUS-16-2a.jpg" alt="Apator AT-WMBUS-16-2a" style="width:50%; height:auto;">

<img src="/images/20260529_083235.jpg" alt="ESP32 + CC1101" style="width:50%; height:auto;">

Used library:
https://github.com/SzczepanLeon/esphome-components/  
You can find there detailed explanation of the CC1101 and other modules.

Tested on:
* ESP32, ESP-WROOM-32 38-pin, CP2102 USB-C
* Radio module CC1101 RF 868 MHz + antena
* Home Assistant 2026.5.1
* ESPHome  2026.5.1
* SzczepanLeon/esphome-components for CC1101 5.1.6

## Pinout for CC1101 
You can find an example of how to connect the ESP to a radio module in the example config file. Of course, you can choose totally different pins on the ESP side.
<img src="/images/CC1101-868mhz-radio-module-pinout.jpg" alt="CC1101 pinout" style="width:50%; height:auto;">



## Example config file for ESP HOME: water-meter.yaml
```yaml
substitutions:
  name: "water-meter"
  friendly_name: "Water meter"

esphome:
  name: "${name}"
  friendly_name: "${friendly_name}"

esp32:
  board: esp32dev
  framework:
    type: esp-idf

external_components:
  - source: github://SzczepanLeon/esphome-components@main
    components:
      - socket_transmitter
      - wmbus_common
      - wmbus_radio
      - wmbus_meter  

# Enable logging
logger:
  id: component_logger
  level: VERBOSE
  baud_rate: 115200

# Enable Home Assistant API
api:


ota:
  - platform: esphome
    password: "xxx"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "${friendly_name}  Fallback Hotspot"
    password: "xxx"

captive_portal:

spi:
  mosi_pin: GPIO32
  clk_pin: GPIO33
  miso_pin: GPIO19

wmbus_radio:  
  radio_type: CC1101
  cs_pin: GPIO23
  irq_pin: GPIO21
  #Interrupt pin (GDO0)
  #gdo0_pin: GPIO21
  #gdo2_pin: GPIO22
  


button:
  - platform: restart
    name: "Restart device"

wmbus_meter:
  - id: cold_water_meter
    #you can find fumber of your meter on label
    meter_id: "0xXXXXXXX"
    type: apator162
    #9x% of meters have this key. In some case you need to ask your water supplier
    key: "00000000000000000000000000000000"
    on_telegram:
      - logger.log:
          format: "Telegram received from main water meter"
  - id: garden_water_meter
    meter_id: "0xXXXXXXX"
    type: apator162
    key: "00000000000000000000000000000000"
    on_telegram:
      - logger.log:
          format: "Telegram received from garden water meter"

sensor:
  - platform: wmbus_meter
    parent_id: cold_water_meter
    name: Cold water consumption
    field: total_m3
    unit_of_measurement: "m³"
    state_class: total_increasing
    device_class: "water"
    accuracy_decimals: 3
    icon: "mdi:counter"

  - platform: wmbus_meter
    parent_id: cold_water_meter
    field: rssi_dbm
    name: Cold water Meter RSSI
    unit_of_measurement: "dBm"
    entity_category: "diagnostic"


  - platform: wmbus_meter
    parent_id: garden_water_meter
    name: Garden water consumption
    field: total_m3
    unit_of_measurement: "m³"
    state_class: total_increasing
    device_class: "water"
    accuracy_decimals: 3
    icon: "mdi:counter"

  - platform: wmbus_meter
    parent_id: garden_water_meter
    field: rssi_dbm
    name: Garden water Meter RSSI
    unit_of_measurement: "dBm"
    entity_category: "diagnostic"

  - platform: wifi_signal
    name: "${friendly_name} WiFi Signal Strength"
    update_interval: 60s
  - platform: uptime
    name: "${friendly_name} Uptime"
    
binary_sensor:   
  - platform: status
    name: "${friendly_name} Status"

switch:
  - platform: restart
    name: "${friendly_name} Restart"
    
text_sensor:
  - platform: version
    name: "${friendly_name} ESPHome Version"

time:
  - platform: homeassistant
```

## Case for module
This case is designed specifically for the 38-pin ESP32 (ESP-WROOM-32 with CP2102 USB-C) development board.  
If needed, you can easily modify the dimensions to fit a different board. This is a fully parametric model—at least, I hope it is! :)
[https://github.com/palmar125/water-meter-esp32-cc1101/case/ESP32_case_v2.f3d](https://github.com/palmar125/water-meter-esp32-cc1101/blob/main/case/ESP32_case_v2.f3d)  
You can find STLs on: https://www.printables.com/model/1740701-esp32-cc1101-case

<img src="/images/20260531_102027.jpg" alt="Case" style="width:50%; height:auto;">
<img src="/images/20260531_102101.jpg" alt="Case" style="width:50%; height:auto;">
<img src="/images/20260531_102112.jpg" alt="Case" style="width:50%; height:auto;">
<img src="/images/20260531_102400.jpg" alt="Case" style="width:50%; height:auto;">
<img src="/images/20260531_102725.jpg" alt="Case" style="width:50%; height:auto;">

## ESPHome & Home Assistant
Here you will find some brief how-to: https://esphome.io/guides/getting_started_hassio/
