# Water meter for Apator16 build with ESP32 + CC1101 on ESPHome & Home Assistant

Used library:
https://github.com/SzczepanLeon/esphome-components/
You can find there detailed explanation of the CC1101 and other modules.

Tested on:
* ESP32, ESP-WROOM-32 38-pin, CP2102 USB-C
* Radio module CC1101 RF 868 MHz + antena
* Home Assistant 2026.5.1
* ESPHome  2026.5.1
* SzczepanLeon/esphome-components for CC1101 5.1.6
* 


# Example config file for ESP HOME: water-meter.yaml
```yaml
esphome:
  name: water-meter
  friendly_name: Water meter

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
    ssid: "Water-Meter Fallback Hotspot"
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
  #gdo0_pin: GPIO21
  #gdo2_pin: GPIO22
  


button:
  - platform: restart
    name: "Restart device"

wmbus_meter:
  - id: cold_water_meter
    meter_id: "0xXXXXXXX"
    type: apator162
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

time:
  - platform: homeassistant
```

# Case for module


