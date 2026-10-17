# ESPHome Component for IKEA Frekvens LED Matrix

This repository provides an [ESPHome](https://esphome.io/) external display component (`frekvens_panel`) for driving the 16×16 LED matrix from the [IKEA Frekvens](https://spritesmods.com/?art=frekvens) speaker/light cube with an ESP32 or ESP8266 for use with [Home Assistant](https://www.home-assistant.io/).

## Hardware & Wiring

The IKEA Frekvens LED matrix uses two 8-bit shift registers to drive its 16×16 LED grid. By replacing or bypassing the stock controller board with an ESP32 or ESP8266, you can control the panel as a standard ESPHome monochrome `display` buffer.

* **Hardware teardown & protocol details**: [SpritesMods — Hackable IKEA speaker lights](https://spritesmods.com/?art=frekvens)
* **Wiring diagram**: [`frekvens-hacking.pdf`](https://github.com/frumperino/FrekvensPanel/blob/master/frekvens-hacking.pdf) (from [`frumperino/FrekvensPanel`](https://github.com/frumperino/FrekvensPanel))

### Pin Connections

| Panel Signal | Description | ESPHome Component |
| :--- | :--- | :--- |
| **`LAC` (Latch)** | Shift register latch pin | `frekvens_panel` (`latch_pin`) |
| **`CLK` (Clock)** | Shift register clock pin | `frekvens_panel` (`clock_pin`) |
| **`DA1` (Data)** | Shift register serial data pin | `frekvens_panel` (`data_pin`) |
| **`EN` (Enable)** | Active-low output enable (PWM for brightness) | `output` (`ledc` / `esp8266_pwm`, `inverted: true`) |
| **Buttons (Optional)** | Rear Red and Yellow buttons (active-low to GND) | `binary_sensor` (`gpio`, `inverted: true`) |

## Configuration

Add this repository to your ESPHome configuration using `external_components`, then configure the `frekvens_panel` display platform:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/allenporter/esphome-ikea-led-matrix
      ref: main
    components:
      - frekvens_panel

display:
  - platform: frekvens_panel
    id: panel
    latch_pin: 3
    clock_pin: 4
    data_pin: 5
    lambda: |-
      // Draw using standard ESPHome display methods
```

### Configuration Variables

* **`latch_pin`** (*Required*, `int`): GPIO pin number connected to the panel's latch input.
* **`clock_pin`** (*Required*, `int`): GPIO pin number connected to the panel's clock input.
* **`data_pin`** (*Required*, `int`): GPIO pin number connected to the panel's data input.
* **`update_interval`** (*Optional*, Time): How often to refresh the display buffer. Defaults to `200ms`.
* All other options from the [ESPHome Display Component](https://esphome.io/components/display/index.html) (`id`, `lambda`, `pages`, `rotation`, `auto_clear_enabled`, etc.).

## Full Example Configuration (ESP32-C3)

Below is a complete example for an `esp32-c3-devkitm-1` with PWM brightness control, rear button inputs, and a clock display:

```yaml
esphome:
  name: frekvens

esp32:
  board: esp32-c3-devkitm-1

external_components:
  - source:
      type: git
      url: https://github.com/allenporter/esphome-ikea-led-matrix
      ref: main
    components:
      - frekvens_panel

light:
  - platform: monochromatic
    name: "Brightness"
    id: brightness
    output: matrix_brightness
    restore_mode: RESTORE_DEFAULT_ON

output:
  - platform: ledc
    id: matrix_brightness
    max_power: 0.05
    pin:
      number: GPIO2
      inverted: true

binary_sensor:
  - platform: gpio
    name: "Red Button"
    pin:
      number: GPIO8
      inverted: true
    filters:
      - delayed_on: 100ms
    on_press:
      - light.toggle: brightness

time:
  - platform: sntp
    id: ntp_time
    timezone: "America/Los_Angeles"

font:
  - file: "04B03.ttf"
    id: digi8
    size: 8

display:
  - platform: frekvens_panel
    id: panel
    latch_pin: 3
    clock_pin: 4
    data_pin: 5
    lambda: |-
      auto time = id(ntp_time).now();
      it.strftime(1, 0, id(digi8), "%I", time);
      it.strftime(1, 8, id(digi8), "%M", time);
```

## Credits

* Original Arduino panel driver and wiring guide by [`frumperino/FrekvensPanel`](https://github.com/frumperino/FrekvensPanel)
* Upstream ESPHome component forks by [`phiten/esphome-ikea-led-matrix`](https://github.com/phiten/esphome-ikea-led-matrix) and [`sascha-hemi/esphome-ikea-led-matrix`](https://github.com/sascha-hemi/esphome-ikea-led-matrix)
