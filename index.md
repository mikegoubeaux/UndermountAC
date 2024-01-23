
![PXL_20240105_172521763_480x480](https://github.com/mikegoubeaux/UndermountAC/assets/9661510/37b48cda-64d1-40a0-bc20-0e199065ab30)

# About

[Undermount AC](https://undermountac.com) has developed an ESPHome HVAC Controller for their air conditioning hardware. This ESPHome configuration was developed by [Mike Goubeaux](https://github.com/mikegoubeaux) in conjunction with Undermount AC. The ESPHome configuration is free and open source. Use at your own risk.

If you have purchased an [Undermount AC ESPhome HVAC Controller](https://undermountac.com/pages/hass), this page can be used to install a ready-made ESPHome configuration to allow control of your UndermountAC via Home Assistant.

This firmware and guide assumes you have a Home Assistant instance running with the ESPHome add-on.

**This is optional!** 

Please feel free to develop your own ESPHome Climate configuration YAML for use with the Undermount AC ESPHome HVAC Controller. This is being provided to help you get started with a configuration that works.

# Installation

You can use the button below to install the pre-built firmware directly to your ESPHome HVAC Controller via USB from a compatible browser.

<esp-web-install-button manifest="./manifest.json"></esp-web-install-button>

<script type="module" src="https://unpkg.com/esp-web-tools@9/dist/web/install-button.js?module"></script><br><br>

**Note: Use the UART port on the device to connect to your computer**

## Adding to Home Assistant

1. After the firmware is installed on your device, the onboard RGB LED (on the esp32) should be blinking red.
2. Use another device and look for an "undermount-ac" wifi access point.
3. Connect to it using password: **12345678**
4. A captive portal will load. Choose the wifi network of your Home Assistant server and add your wifi credentials.
5. The device will reboot and join your wifi network.
6. **If you do not have ESPHome installed in Home Assistant, pause here and visit [this link](https://esphome.io/guides/getting_started_hassio) to set it up.**
7. You should now see a newly discovered ESPHome device in Home Assistant under **Settings** -> **Devices & Services**.
8. Click "Configure" and the device will be added to Home Assistant.
9. Congrats! You should now have an Undermount AC Climate Component in Home Assistant that will control your Undermount AC hardware! If you want or need to make code changes, continue on to the following steps.
10. In your ESPHome dashboard, you should eventually see the device with the option to "Adopt".
11. Click "Adopt" - here you can rename the device. You will be prompted to install the new configuration. From here on out, you have full control of the YAML configuration of your Undermount AC ESPHome HVAC Controller.

# Hardware & Wiring

The board developed by Undermount AC includes an esp32-S3-devkit-1. Pinouts for the outputs are at the bottom of the page for reference.

The ESPHome HVAC Controller is a simple module with 6 independent outputs. Two outputs are Low (Ground) only, and four are Low/High configurable using a jumper. Our A/C system requires 4 of the 6 outputs which leaves two outputs available for the Heating side of your system (Solenoid, Heater control, etc).

The onboard power supply can handle input of **10-31V DC**.

The most recent version of the V3 AC kits will require:
- Neg = **Black Wire**
- Pos = **Red Wire**
- Output 1, Set to High (+Supply Voltage) for the PWM blower - **White Wire**
- Output 4, Set to Low (Ground) for 2nd Speed compressor relay - **Green Wire**
- Output 5 = Low to supply voltage to blower fan - **Brown Wire**
- Output 6 = Low to engage Cool Relay - **Blue Wire**

Output 2 and 3 are free to be used for heat or other functions that require either a High or Low signal (jumper selectable).

In ESPHome two switch entities are created and exposed to the Home Assistant frontend for Outputs 2 and 3. You can rename or remove those switches from the configuration if they are not needed.


# Using this ESPHome configuration

Once you've installed the firmware on your [Controller](https://undermountac.com/pages/hass) and adopted the device into your [ESPHome](https://esphome.io) integration on [Home Assistant](https://www.home-assistant.io), a [climate entity](https://esphome.io/components/climate/) will be added to Home Assistant for full control of your Undermount AC system.

![Screenshot 2024-01-22 at 4 49 48 PM](https://github.com/mikegoubeaux/UndermountAC/assets/9661510/8054f70b-17a0-45c2-9f98-bd88c766dda4)


# Configuration and Operation

If you want or need to make changes to the configuration, refer to the [ESPHome Climate Component documentation](https://esphome.io/components/climate/index.html).

## Status Light

The onboard RGB LED of the esp32 is used as a status light and is visible through the case. These are the indicators:
```
Flashing Red - not connected to Home Assistant
Faint Blue - Idle (on but not currently cooling)
Slow Pulsing Blue - Cooling
Fast Pulsing Blue - Cooling - Compressor on high
Steady Orange - Thermostat Off (System in Standby)
```

## High Compressor Speed

The compressor has a high speed that can be selectively engaged. In this configuration, the compressor is set to high under two conditions:

- When the system has been cooling for over 30 minutes
- When the delta between the target temp and current temp is greater than 5 °C

The compressor speed is reset once the target temperature is reached or if the AC is turned off.

These settings can be modified under the Climate Component:
```
max_cooling_run_time: 30min
supplemental_cooling_delta: 5
```

## Temperature / Humidity Sensor

By default, the provided configuration uses the included 1 meter SHT30 Temperature and Humidity sensor as the current temperature for the Climate Componenent. Both temperature and humidty are exposed to Home Assistant as entites for use in the frontend/automations/etc. 

Optionally, a Home Assistant temperature sensor can be [imported into ESPHome](https://esphome.io/components/sensor/homeassistant.html) and used in place of the included onboard sensor in the Climate Component by modifying the configuration accordingly. _You may want to do this if you are averaging multiple temperature sensors together in Home Assistant, or want to sample the temperature somewhere other than within 1 meter of the Undermount AC controller using another temperature sensor device._

## Fan

By default the Fan entity is hidden from the Home Assistant frontend. It is recommended to simply use the the Climate Component which has preset fan speeds of Low, Medium, High, and Auto. **Auto** chooses fan speed based on the delta of temperature set point and current temperature for a hands-off experience. Fan speeds can also be included in Climate presets.

If you want to have granular control over the fan, change the following configuration variable in the Fan component:
```
internal: false
```
This will provide a fan entity to the Home Assistant frontend for full control of the UndermountAC blower. Please note that fan speed percentages are remapped from 40-98% to protect your evaporator. If your Fan mode is set to Auto, this will override manual control of the fan.

## Climate Presets

There are three Climate presets provided with this configuration.

- Home - target temp: 75 °F, Fan mode: auto, Thermostat: Cool
- Sleep - target temp: 71 °F, Fan mode: low, Thermostat: Cool
- Standby - target temp: 75 °F, Fan mode: auto, Thermostat: Off

If the HVAC Controller is restarted, state is restored from memory if possible. If no memory exists, the 'Standby' preset (off) is used.

Presets can be added or modified in the configuration.

# Disclaimer!

Please **NOTE** that short cycling the compressor can cause damage to the equipment. Additionally, a minimum fan speed should be maintained to protect the evaporator from freezing.

The provided ESPHome configuration has the following Climate Component safeguards in place:

**Minimum Cooling / Off Time:**
```
min_cooling_off_time: 2min
min_cooling_run_time: 2min
```
Both of these Climate Control configuration variables should be set to a minimum of 2 minutes to avoid short cycling.

**Fan Speed Output**

```
min_power: 0.40
max_power: 0.98
```

This clamps the effective fan speed between 40% and 98% regardless of the fan percentage that is chosen in the UI. The fan speeds are effectively remapped. It is not recommended to change these values.

# Pinouts

This ESPHome configuration should work with all recent models of Undermount AC units. The pin numbers are already defined for all relavant outputs.

However, for reference, the included esp32 pinouts are as follows:
```
SCL GPIO 14
SDA GPIO 21
RGB GPIO 48
Output 1 = GPIO 7
Output 2 = GPIO 46
Output 3 = GPIO 18
Output 4 = GPIO 09
Output 5 = GPIO 20
Output 6 = GPIO 19
```
