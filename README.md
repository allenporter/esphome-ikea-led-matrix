# ESPHome component for IKEA Frekvens LED matrix 

Frekvens is 16 by 16 LED Matrixes from IKEA and they are [Hackable](https://spritesmods.com/?art=frekvens) for Usage with HomeAssistant / ESPHome.

This Repository is a custom component for ESPHome to interface these Matrixes. This is a
fork of upstream repositories, fixed for newer versions of ESPHome.

See upstream forks for details on how to hack it.

## Software

Here is a short config to demonstrate the usage to display time on Frekvens and Obegränsad:

### Frekvens

```yaml
esphome:
  name: frekvens-clock
  platform: ESP8266
  board: d1_mini
  platformio_options:
    lib_deps:
      - Wire                            # Also required by GFX.
      - SPI                             # Also required by GFX.
      - adafruit/Adafruit BusIO         # Required by GFX Library.
      - adafruit/Adafruit GFX Library   # Required for FrekvensPanel.

external_components:
  - source:
      type: git
      url: https://github.com/allenporter/esphome-ikea-led-matrix
      ref: main
    components: [ frekvens_panel ]

light:
  - platform: monochromatic
    name: 'Brightness'
    output: matrix_brightness
    restore_mode: RESTORE_DEFAULT_ON

output:
  - platform: esp8266_pwm
    # Enables brightness control.
    max_power: 0.05
    id: matrix_brightness
    pin:
      number: GPIO14
      inverted: True

time:
  - platform: sntp
    id: ntp_time
    timezone: 'Europe/Berlin'

font:
  - file: "04B03.ttf"
    id: b03
    size: 8

display:
  - platform: frekvens_panel
    latch_pin: 12
    clock_pin: 04
    data_pin: 05

    lambda: |-
      it.strftime(4, 0, id(b03), "%H", id(ntp_time).now());
      it.strftime(4, 8, id(b03), "%M", id(ntp_time).now());
```
