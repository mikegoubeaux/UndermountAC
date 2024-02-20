
![PXL_20240105_172521763_480x480](https://github.com/mikegoubeaux/UndermountAC/assets/9661510/37b48cda-64d1-40a0-bc20-0e199065ab30)

# About

[Undermount AC](https://undermountac.com) has developed an [ESPHome HVAC Controller](https://undermountac.com/pages/hass) for their [air conditioning hardware](https://undermountac.com). This ESPHome configuration was developed by [Mike Goubeaux](https://github.com/mikegoubeaux) in conjunction with Undermount AC. The ESPHome configuration is free and open source. Use at your own risk.

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
10. In your ESPHome dashboard, you should eventually see the device with the option to "Adopt" (may require a restart of Home Assistant).
11. Click "Adopt" - here you can rename the device. You will be prompted to install the new configuration.
12. From here on out, you have full control of the YAML configuration of your Undermount AC ESPHome HVAC Controller.

# Hardware & Wiring

The board developed by Undermount AC includes an esp32-S3-devkit-1. Pinouts for the outputs are at the bottom of the page for reference.

The ESPHome HVAC Controller is a simple module with 6 independent outputs. Two outputs are Low (Ground) only, and four are Low/High configurable using a jumper. Our A/C system requires 4 of the 6 outputs which leaves two outputs available for the Heating side of your system (Solenoid, Heater control, etc).

The onboard power supply can handle input of **10-31V DC**.

The most recent version of the V3 AC kits will require:
- Neg = **Black Wire**
- Pos = **Red Wire**
- Output 1 -  **White Wire** (jumper to High) - PWM Blower
- Output 2 - **Brown Wire** (jumper to High) - Voltage to Blower Fan Relay
- Output 3 - **Blue Wire** (jumper to High) - Engage Cool Relay
- Output 4 - **Green Wire** (jumper to High) High Speed Compressor Relay
- Output 5 = Unused (available for other uses)
- Output 6 = Unused (available for other uses)


Outputs 5 and 6 are free to be used for heat or other functions.

In ESPHome two switch entities are created and exposed to the Home Assistant frontend for those outputs. You can rename or remove those switches from either the Home Assistant frontend or the ESPHome YAML configuration if they are not needed.


# Using this ESPHome configuration

Once you've installed the firmware on your [Controller](https://undermountac.com/pages/hass) and clicked "Configure" on the newly discovered device in **Settings -> Devices & Services** in [Home Assistant](https://www.home-assistant.io), a [climate entity](https://esphome.io/components/climate/) will be added to Home Assistant for full control of your Undermount AC system.

![Screenshot 2024-02-05 at 9 57 34 AM](https://github.com/mikegoubeaux/UndermountAC/assets/9661510/16ae24cd-9ffa-4505-9cf9-c2d04e9b7c10)
![Screenshot 2024-02-20 at 1 40 30 PM](https://github.com/mikegoubeaux/UndermountAC/assets/9661510/cea70e0b-1972-49f9-bc46-b22eeda854d1)


If you wish to make changes to the ESPHome YAML configuration, you will additionally need to "Adopt" the device into your [ESPHome dashboard](https://esphome.io/guides/getting_started_hassio.html) to take over control of the device firmware.

# Configuration and Operation

If you want or need to make changes to the configuration, refer to the [ESPHome Climate Component documentation](https://esphome.io/components/climate/index.html).

## Status Light

The onboard RGB LED of the esp32 is used as a status light and is visible through the case. These are the indicators:
```
Flashing Red - not connected to Home Assistant
Faint Blue - Idle (on but not currently cooling)
Slow Pulsing Blue - Cooling
Fast Pulsing Blue - Cooling - Compressor on high
Slow Pulsing Cyan - Fan Only Mode
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
This will provide a fan entity to the Home Assistant frontend for full control of the UndermountAC blower. Please note that fan speed percentages are remapped from 40-98% if the compressor is on low speed and 60-98% if the compressor is running at normal speed to protect your evaporator. If your Fan mode is set to Auto, this will override manual control of the fan.

## Fan Only Mode

The Climate Thermostat can be put into "Fan Only" mode via the Climate Card in the Home Assistant frontend. This is currenlty configured to act as a manual fan control for ventilation. You can choose Low, Medium, High and the fan will run at that speed regardless of the set point. However, if you choose Auto the controller will choose the fan speed based on the set point tempurature, but will not turn off. If your set point has been reached, the fan will stay at the lowest speed.

If you'd rather the fan be used as a cooling fan (at any speed) - activating only when the setpoint has not been reached - add this configuration variable to the Climate entity:
```
fan_only_cooling: true
```

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

The ```min_power``` value is increased to ```0.60``` when high speed compressor is enabled.

# Diagnostics

A single diagnostics sensor is created in Home Assistant: ```Blower Speed```. Though this is not feedback from the blower itself, it represents the actual fan percentage the blower should be working at based on all factors including fan mode (Low, Medium, High) and at which speed the compressor is running.

# Pinouts

This ESPHome configuration should work with all recent models of Undermount AC units. The pin numbers are already defined for all relavant outputs.

However, for reference, the included esp32 pinouts are as follows:
```
SCL GPIO 14
SDA GPIO 21
RGB GPIO 48
Output 1 = GPIO 7
Output 2 = GPIO 9
Output 3 = GPIO 8
Output 4 = GPIO 10
Output 5 = GPIO 20
Output 6 = GPIO 19
```
