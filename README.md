# solar-controlled-appliance
Control an electric appliance based on scheduling and solar electricity production in Home Assistant.

## Introduction

This blueprint allows you to control an electric appliance based on solar panels production, house instant power consumpsion, and scheduling. It also adds a minimum daily on time (for cloudy days), maximum on time, and holidays mode.

## Changelog

- 1.0: Initial public release.

## Installation

## Discussion Forum

You can discuss about this blueprint in Home Assistant Community.

## Prerrequisites

You need to identity the next entities for the blueprint to work (the name is only a way to referring to it here, it can change in your HA installation):

| Name      | Type | Description |
| --------- | ---- | ----------- |
| `switch_appliance` | `switch` | Appliance to be controlled |
| `active_power` | `sensor` | Active power of home (possitive: export, negative: import) |

You need to create the next entities:

| Name      | Type | Description |
| --------- | ---- | ----------- |
| `bool_appliance` | `input_boolean` | It is On when the appliance _may_ be switched on. It is intended to be controlled by a scheduling. The real control of the appliance is done by the blueprint. It can be On 24/7 or you can limit the hours when the appliance may be On. |
| `bool_cloudy` | `input_boolean` | It is On when the appliance _must_ be switched on in case a minimum on time still has not reached during the day. The Off->On is intended to be controlled by a scheduling; the On->Off is controlled by the blueprint. Usually it should be On some time after noon. |
| `bool_holidays` | `input_boolean` | It is set to on if the appliance _must_ not be switched on. |
| `daily_on_time` | `sensor` | Number of hours the appliance has been On during present day. |
| `timer_appliance` | `timer` | Initialized to the maximum time the appliance must be On. After this time, the appliance and `bool_appliance` are switched off. This is optional. |

Sensor `daily_on_time` can be created in `configuration.yaml` with this code:
```
sensor:
  - platform: history_stats
    name: Daily on time appliance 1
    entity_id: switch.my_appliance1
    state: 'on'
    type: time
    start: "{{ now().replace(hour=0, minute=0, second=0) }}"
    end: "{{ now() }}"
```

Schedulings can be easily created with HACS integration 
[scheduler-component](https://github.com/nielsfaber/scheduler-component) and HACS lovelace integration [scheduler-card](https://github.com/nielsfaber/scheduler-card).

## Configuration

Settings -> Automations and scenes -> Create automation -> Solar controlled appliance.

Parameters of the blueprint (for a description go back to [Prerrequisites](https://github.com/miguelpucela/solar-controlled-appliance#Prerrequisites)):

| Name | Entity | Required/Optional | Description | Comments |
| -------------------- | ------ | ----------------- | ----------- | -------- | 
| **Appliance switch** | `switch_appliance` | Required | | |
| **Active power** | `active_power` | Required | | |
| **Appliance boolean** | `bool_appliance` | Required | | |
| **Daily On-Time** | `daily_on_time` | Required | | Unit: hours |
| **Cloudy boolean** | `bool_cloudy` | Required | | |
| **Minimum On time** | | Required | Minimum daily On time (for cloudy days) | On cloudy days, when `bool_cloudy` is on, if `daily_on_time` is lower than this value, appliance is switched on until reaching this value. When it happens, boch `bool_cloudy` and `bool_appliance` (and `switch_appliance) are set to Off. Unit: minutes. Range: 0-240 |
| **Holidays boolean** | `bool_holidays` | Required | | |
| **Nominal appliance power** | | Required | Nominal appliace power | It can be found in the appliance manual or nameplate. Unit: Watts |
| **switch-off hysteresis delay** | | Optional (default: 300 s) | When `active_power` is negative, wait for this time to switch off appliance to avoid continuous switching when active power is close to zero | Unit: seconds |
| **switch-on hysteresis delay** | | Optional (default: 10 s) | When `active_power` is above _Nominal appliance power_, wait for this time to switch on appliance to avoid continuous switching | Unit: seconds |
| **Appliance timer** | `timer_appliance` | Optional (no default) | | If not present, no timer is used |

## Footnotes

- `bool_appliance` scheduling can be as simple as setting it On sunrise and Off at sunset.
- `bool_cloudy` scheduling should be set to On several hours after `bool_appliance`, in such a way that only in cloudy or rainy days, appliance is on when panels are not providing enough energy.
- You can create as many automations as the number of appliances you want to control.
- In this case, hysteresis parameteres should be different in each one, to avoid all the appliances switching on and off at the same time with active power variations. It is recommented to set lower switch-off times and higher switch-on times for higher nominal power appliances. It's enough to vary values 5-10 seconds between appliances.
