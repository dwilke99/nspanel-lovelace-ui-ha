# NSPanel Lovelace UI

> [!WARNING]
> **Personal fork, no support.** This repository is a personal fork of [joBr99/nspanel-lovelace-ui](https://github.com/joBr99/nspanel-lovelace-ui), maintained for a single panel. Its goal is to make the Home Assistant AppDaemon backend work with the newer 5.x display firmware from the [ioBroker project](https://github.com/ticaki/ioBroker.nspanel-lovelace-ui). There is no support, no release schedule and no guarantee that it works for you. If you want a maintained version, use the original repository.
>
> In HACS this fork appears as "NSPanel Lovelace UI Backend (Fork)" and has to be added as a custom repository (category AppDaemon). Do not install it alongside the original backend, both provide the same AppDaemon module.
>
> Everything below is the original README. Badges, links and the donation link refer to the original project. The license remains GPL-3.0.

If you like this project consider buying me a pizza 🍕 <a href="https://paypal.me/joBr99" target="_blank"><img src="https://img.shields.io/static/v1?logo=paypal&label=&message=donate&color=slategrey"></a>

[![hacs_badge](https://img.shields.io/badge/HACS-Default-41BDF5.svg)](https://github.com/hacs/integration)
![hacs validation](https://github.com/joBr99/nspanel-lovelace-ui/actions/workflows/hacs-validation.yaml/badge.svg)
[![GitHub Release](https://img.shields.io/github/release/joBr99/nspanel-lovelace-ui.svg)](https://github.com/joBr99/nspanel-lovelace-ui/releases)
![Project Maintenance](https://img.shields.io/maintenance/yes/2026.svg)
[![GitHub Activity](https://img.shields.io/github/commit-activity/y/joBr99/nspanel-lovelace-ui.svg)](https://github.com/joBr99/nspanel-lovelace-ui/commits/main)


NsPanel Lovelace UI is a Firmware for the nextion screen inside of NSPanel in the Design of [HomeAssistant](https://www.home-assistant.io/)'s Lovelace UI Design.

**Visit https://docs.nspanel.pky.eu/ for installation instructions and documentation of the configuration.**

Supported Home Automation Systems:

- [Home Assistant](https://docs.nspanel.pky.eu/prepare_nspanel/) - AppDaemon Backend
- [ioBroker](https://docs.nspanel.pky.eu/prepare_nspanel_ioBroker/) - 3rd-party Typescript Backend maintained by @britzelpuf and @Armilar

NsPanel needs to be flashed with Tasmota (or through a 3rd-party Component with ESPHome)

**EU Model and US Model supported (in portrait and landscape orientation)**

![nspanel-rl](docs/img/nspanel-rl.png)

## Features

- Entities Page with support for cover, switch, input_boolean, binary_sensor, sensor, button, number, scenes, script, input_button and light, input_text (read-only), lock, fan, timer and automation
- Grid Page with support for cover, switch, input_boolean, button, scenes, light, lock, timer and automation
- Detail Pages for Lights (Brightness, Temperature and Color of the Light) and Covers (Position)
- Thermostat Page 
- Media Player Card
- Alarm Control Card
- Screensaver Page with Time, Date and Weather Information
- Card with QrCode to display WiFi Information
- Localization possible (currently 38 languages)
- **Everything is dynamically configurable by a yaml config, no need to code or touch Nextion Editor**

It works with [Tasmota](https://tasmota.github.io/docs/) and MQTT. 
To control the panel and update it with content from HomeAssistant, there is an [AppDaemon](https://github.com/AppDaemon/appdaemon) App.

See the following picture to get an idea of the look of this firmware for NSPanel.

![screens](docs/img/screens.png)

Some (not all) screenshots from the US Portrait Version:

![screens-us-p](docs/img/screens-us-p.png)

## Documentation

Visit https://docs.nspanel.pky.eu/ for installation instructions and documentation of the configuration.

## Other Resources

### Alternative Backends for other SmartHome Systems

ioBroker: https://forum.iobroker.net/topic/58170/sonoff-nspanel-mit-lovelace-ui/

IP Symcon: https://community.symcon.de/t/beta-modul-nspanel-lovelace-ui/130109

SmartHomeNG: https://github.com/sisamiwe/shng-nspanel-plugin

OpenHAB: https://github.com/olialb/nspanelMqttBridge

NodeRed: https://github.com/laluz742/node-red-contrib-nspanel-lui

ESPHome without any Backend: https://github.com/olicooper/esphome-nspanel-lovelace-native
