# About

[Undermount AC](https://undermountac.com) has developed an ESPHome HVAC Controller for their air conditioning hardware. This ESPHome configuration was developed by [Mike Goubeaux](https://github.com/mikegoubeaux) in conjunction with Undermount AC. The ESPHome configuration is free and open source. Use at your own risk.

If you have purchased an [Undermount AC ESPhome HVAC Controller](https://undermountac.com/pages/hass), this page can be used to install a ready-made ESPHome configuration.

**This is optional!** 

Please feel free to develop your own ESPHome Climate configuration YAML for use with the Undermount AC ESPHome HVAC Conroller. This is being provided to help you get started with a configuration that works.

# Using this ESPHome configuration

Once you've installed the firmware on your [Controller](https://undermountac.com/pages/hass) (see Installation below) and adopted the device into your [ESPHome](https://esphome.io) integration on [Home Assistant](https://www.home-assistant.io), a [climate entity](https://esphome.io/components/climate/) will be added to Home Assistant for full control of your Undermount AC system.

![Screenshot 2024-01-22 at 4 49 48 PM](https://github.com/mikegoubeaux/UndermountAC/assets/9661510/8054f70b-17a0-45c2-9f98-bd88c766dda4)


# Installation

You can use the button below to install the pre-built firmware directly to your ESPHome HVAC Controoler via USB from a compatible browser.

<esp-web-install-button manifest="./manifest.json"></esp-web-install-button>

<script type="module" src="https://unpkg.com/esp-web-tools@9/dist/web/install-button.js?module"></script>


# Configuration and Operation

If you want or need to make changes to the configuration, refer to the [ESPHome Climate Component documentation](https://esphome.io/components/climate/index.html).

## Temperature / Humidity Sensor

By default, the provided configuration uses the included 1 meter SHT30 Temperature and Humidity sensor as the current temperature for the Climate Componenent. Both temperature and humidty are exposed to Home Assistant as entites for use in the front end/automations/etc. 

Optionally, a Home Assistant temperature sensor can be [imported into ESPHome](https://esphome.io/components/sensor/homeassistant.html) and used in place of the included onboard sensor in the Climate Component by modifying the configuration accordingly. _You may want to do this if you are averaging multiple temperature sensors together in Home Assistant, or want to sample the temperature somewhere other than within 1 meter of the Undermount AC controller._

## Fan

The Fan entity is hidden from the Home Assistant front end by default. It is recommended to simply use the the Climate Compoenet which has preset fan speeds of Low, Medium, High, and Auto. Auto changes fan speed based on the delta of temperature set point and current temperature for a hands-off experience. Fan speeds can also be included in Climate presets.

If you want to have granular control over the fan, change the configuration for the Fan component to be:
```
internal: false
```
This will provide a fan entity to the Home Assistant front end for granular control. Please note that fan speed percentages are remapped from 40-98% to protect your evaporator. See below for more information. 

## Climate Presets

There are two Climate presets provided with this configuration.

- Home - target temp: 75 °F, Fan mode: auto
- Sleep - target temp: 71 °F, Fan mode: low

Presets can be added or modified in the configuration.

# Disclaimer!

Please **NOTE** that that short cycling the compressor can cause damage to the equipment. Additionally, a minimum fan speed should be maintained to protect the evaporator from freezing.

The provided ESPHome configuration has the following Climate Component safeguards in place:

**Minimum Cooling / Off Time:**
```
min_cooling_off_time: 2min
min_cooling_run_time: 2min
```
Both of these Climate Confiration configuration variables should be set to a minimum of 2 minutes.

**Fan Speed Output**

```
min_power: 0.40
max_power: 0.98
```

This clamps the effective fan speed between 40% and 98% regardless of the fan percentage that is chosen in the UI. The fan speeds are effectively remapped.
