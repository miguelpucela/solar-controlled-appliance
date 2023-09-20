# solar-controlled-appliance
Control an electric appliance based on scheduling and solar electricity production

## Introduction

This blueprint allows you to control an electric appliance based on solar panels production, house instant power consumpsion, and scheduling. It also adds a minimum daily on time (for cloudy days), maximum on time, and holidays mode.

## Versions

- 1.0: Initial public release.

## Installation

## Prerrequisites

You need to identity the next instances for the blueprint to work:

- `switch_appliance`: switch of the appliance to be controlled.
- `active_power`: active power of your home (possitive: export, negative: import).

You need to create the next instances:

- `bool_appliance`: `input_boolean` which is On when the appliance _may_ be switched on. It is intended to be controlled by a scheduling. The real control of the appliance is done by the blueprint. It can be On 24/7 or you can limit the hours when the appliance may be On.
- `bool_cloudy`: `input_boolean` which is On when the appliance _must_ be switched on in case a minimum on time has not reached during the day. The Off->On is intended to be controlled by a scheduling; the On->Off is controlled by the blueprint. Usually it should be On some time after noon.
- `bool_holidays`: `input_boolean` which is set to on if the appliance _must_ not be switched on.
- `daily_on_time`: `sensor` with the number of hours the appliance has been On during present day. It can be created in configuration.yaml with next code:
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
- `timer_appliance`: `timer` with the maximum time the appliance must be On. After this time, the appliance and `bool_appliance` are switched off. This is optional.

Schedulings can be easily created with HACS integration 
[scheduler-component](https://github.com/nielsfaber/scheduler-component) and HACS lovelace integration [scheduler-card](https://github.com/nielsfaber/scheduler-card).

## Configuration

Settings -> Automations and scenes -> Create automation -> Solar controlled appliance.

Parameters of the blueprint (for a description go back to [Prerrequisites](#prerrequisites)):

- **Appliance switch**: `switch_appliance`. Required.
- **Active power**: `active_power`. Required.
- **Appliance boolean**: `bool_appliance`. Required.
- **Time on (hours)**: `daily_on_time`. Required.
- **Cloudy boolean**: `bool_cloudy`. Required.
- **Minimum on time (minutes)**: minimum daily on time in minutes (for cloudy days). Required. Range: 0-240 minutes.
On cloudy days, when `bool_cloudy` is on, if `daily_on_time` is lower than this value, appliance is switched on until reaching this value. When it happens, boch `bool_cloudy` and `bool_appliance` (and `switch_appliance) are set to Off.
- **Holidays boolean**: `bool_holidays`. Required.
- **Nominal appliance power (Watts)**: It can be found in the appliance manual or nameplate. Required
- **switch-off hysteresis time (seconds)**: when `active_power` is negative, wait for this time to switch off appliance to avoid continuous switching when active power is close to zero. Optional, default (300 s).
- **switch-on hysteresis time (seconds)**: when `active_power` is above _Nominal appliance power_, wait for this time to switch on appliance to avoid continuous switching. Optional, default (10 s).
- **Appliance timer**: `timer_appliance`. Optional. If not present, no timer is used.

## Footnotes

- `bool_appliance` scheduling can be as simple as setting it On sunrise and Off at sunset.
- `bool_cloudy` scheduling should be set to On several hours after `bool_appliance`, in such a way that only in cloudy or rainy days, appliance is on when panels are not providing enough energy.
- You can create as many automations as the number of appliances you want to control.
- In this case, hysteresis parameteres should be different in each one, to avoid all the appliances switching on and off at the same time with active power variations. It is recommented to set lower switch-off times and higher switch-on times for higher nominal power appliances. It's enough to vary values 5-10 seconds between appliances.
