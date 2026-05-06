# User Manual

## Overview

This document explains how to control the two fans used by the purifier module in this Snapmaker U1 Black Box project.

The purifier module controls two fans:

- `exhaust`
- `inner`

It also stores persistent settings in `purifier.json` for delay and work-time related values.

## Fan roles

### Exhaust fan

The exhaust fan pushes air out of the enclosure.

In this project, the exhaust path is planned to include a vent shutter so the outlet can be closed when the system is off.

### Inner fan

The inner fan is used for internal air circulation.

In the stock purifier concept, it is paired with HEPA and active-carbon filtration.

## Direct control

Use `SET_PURIFIER` for manual control.

Syntax:
```gcode
SET_PURIFIER FAN=<exhaust|inner> SPEED=<0.0-1.0> DELAY_OFF=<seconds> WORK=<seconds>
```

Parameters:

- `FAN`: `exhaust` or `inner`
- `SPEED`: direct speed from `0.0` to `1.0`
- `DELAY_OFF`: delay before the fan turns off after speed is set to `0`
- `WORK`: accumulated inner-fan work time in seconds

Examples:
```gcode
SET_PURIFIER FAN=exhaust SPEED=0.8 DELAY_OFF=180
SET_PURIFIER FAN=inner SPEED=0.6 DELAY_OFF=120
SET_PURIFIER FAN=exhaust SPEED=0
SET_PURIFIER FAN=inner SPEED=0
```

## Read current state

Use:
```gcode
GET_PURIFIER
```

This returns the current purifier-related state managed by the module.

## Mode-based control

Use `SET_PURIFIER_MODE` to select one of the supported operating modes.

Syntax:
```gcode
SET_PURIFIER_MODE MODE=<0|1|2|3> [mode-specific parameters]
```

Common parameters:

- `FAN_SPEED`
- `DESIRE_TEMP`
- `ALARM_TEMP`
- `DELAY_OFF`
- `DYNAMIC_FAN_CONTROL`

Examples:
```gcode
SET_PURIFIER_MODE MODE=1 FAN_SPEED=0.6 DESIRE_TEMP=40 ALARM_TEMP=45 DELAY_OFF=180 DYNAMIC_FAN_CONTROL=1
SET_PURIFIER_MODE MODE=2 DESIRE_TEMP=50 FAN_SPEED=0.6 DELAY_OFF=180
```

## Wait for chamber condition

Use:
```gcode
WAIT_CHAMBER_TEMP TIMEOUT=<seconds>
```

Example:
```gcode
WAIT_CHAMBER_TEMP TIMEOUT=300
```

This is useful when a workflow should wait for chamber conditions before continuing.

## Web API

The module also exposes:
```text
control/purifier
```

Accepted fields:

- `fan`
- `speed`
- `delay`
- `work`

The `fan` value is `exhaust` or `inner`.

The `speed` value is provided as `0` to `100` and converted internally.

A successful response returns:
```json
{"state": "success"}
```

An error response returns:
```json
{"state": "error", "message": "..."}
```

## Notes

Use direct `SET_PURIFIER` commands when you want manual control.

Use `SET_PURIFIER_MODE` when you want the purifier logic to manage fan behavior by mode or temperature.

Because this project uses custom hardware, always verify wiring, power selection, and fan compatibility before relying on automatic control.

## Related docs

- [`../README.md`](../README.md)
- [`reverse-engineering-notes.md`](reverse-engineering-notes.md)
- [`hardware-plan.md`](hardware-plan.md)
