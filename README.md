
# WARNING!!!!  This is a work in progress

Despite how it might look and all the words below, this is just a plan.  I'm happy to answer questions, but the short version is that I haven't even yet proven this will work as I expect.  At this time, consider this more of a design spec than anything even remotel close to working.


# esphome-remote_topside

Wirelessly control your spa by emulating the topside for Hydroquip and (some) Gecko spa / hot tub systems by using the ESP32 and the [ESPHome](https://esphome.io) framework.

## Features
* Does not require or use anything relatetd to in.touch
* Control all functions (including setting temperature) normally available at the spa topside
* Retrieve current temperature

## Requirements

* ESPHome 1.15.0-dev or greater
* A Hydropquip or Gecko spa system that uses the 8-pin top-side connector (see picture below). Other connector types *might* work
* ESP32 with enough spare GPIOs to match your needs
* Topside extension cable

## Supported Microcontrollers
This  should work on most ESP32 platforms.  It likely won't work on ESP8266 devices because of their poor support for acting as a SPI slave.

## Supported Controllers Units

(TBD)



## Usage
### Step 1: Pick GPIO pins.

Interface to your ESP32.

Avoid these pins: (TBD)

### Step 2: Use ESPHome 1.15.0-dev or higher

The code in this repository makes use of a number of features in the as-yet unreleased 1.15.0 version of ESPHome, including various Fan modes.

### Step 3: Clone this repository into your ESPHome configuration directory

This repository needs to live in your ESPHome configuration directory, as it
doesn't work correctly when used as a Platform.IO library, and there doesn't
seem to be an analog for that functionality for ESPHome code.

On Home Assistant (with supervisor) you'll want to do something like:

* Change directories to your esphome configuration directory.
* `mkdir -p src`
* `cd src`
* `git clone https://github.com/mrand/esphome-remote_topside.git`

### Step 4: Configure your ESPHome device with YAML

Create an ESPHome YAML configuration with the following sections:
 * `esphome: libraries: SPI slave` - if we can't get ESPHome library to do it
 * `esphome: includes: [src/esphome-remote_topside]`
 * `climate:` - set up a custom climate entry, change the SPI port as needed.

The custom climate definition should use `platform: custom` and contain a
`lambda` block, where you instantiate an instance of the remote_topside
class, and then register it with ESPHome. It should also contain a "climates"
entry. You can choose `&VSPI` or `&HSPI` and 
re-enable logging to the main serial port.

If that's all greek to you, here's an example. Change "Spa" if you'd like.

```yaml
climate:
  - platform: custom
    lambda: |-
      auto my_topside = new SpaTopSide(&Spi);
      App.register_component(my_topside);
      return {my_topside};
    climates:
      - name: "Spa"
```


# Example configuration

Below is an example configuration which will include wireless strength
indicators and permit over the air updates. You'll need to create a
`secrets.yaml` file inside of your `esphome` directory with entries for the
various items prefixed with `!secret`.

```yaml
esphome:
  name: spa
  platform: ESP32
  board: put_board_type_here  # https://docs.platformio.org/en/latest/platforms/espressif32.html#boards

  libraries:
#    - SwiCago/HeatPump  # this will change or (hopefully) go away

  includes:
    - src/esphome-remote_topside

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Spa Hotspot"
    password: !secret fallback_password

captive_portal:

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:

# Enable Web server.
web_server:
  port: 80

  # Sync time with Home Assistant.
time:
  - platform: homeassistant
    id: homeassistant_time

# Sensors with general information.
sensor:
  # Uptime sensor.
  - platform: uptime
    name: Spa Uptime

  # WiFi Signal sensor.
  - platform: wifi_signal
    name: Spa WiFi Signal
    update_interval: 60s


climate:
  - platform: custom
    lambda: |-
      auto my_topside = new remote_topside(&Spi);
      App.register_component(my_topside);
      return {my_topside};
    climates:
      - name: "Spa"
```

# See Also

## References

First, this was based on work done for a different project: 
* https://github.com/geoffdavis/esphome-mitsubishiheatpump

Other useful documentation and information:
* https://esphome.io/custom/spi.html and https://esphome.io/api/spi_8h.html
* https://esphome.io/components/sensor/custom.html
* https://esphome.io/components/climate/custom.html
* https://gist.github.com/liads/c702fd4b8529991af9cd52d03b694814 (another custome ESPHome climate component) 
* https://github.com/SwiCago/HeatPump (library originally used by the Mitsubishi Heat Pump project)
