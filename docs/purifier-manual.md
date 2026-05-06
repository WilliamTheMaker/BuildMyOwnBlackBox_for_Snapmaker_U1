# Purifier Manual

The purifier module controls two fans (exhaust and inner) and their power for your Snapmaker/Klipper setup. It monitors chamber temperature and fan RPM, can automatically adjust fan speed, and can pause printing if cooling/heating fails or becomes unsafe.

## Klipper config items

`exhaust_pin`

`inner_pin`

`temperature_sensor`  
defined in Klipper, used as the chamber temperature source.

The module stores persistent settings in `purifier.json` under the printer’s persistent config directory (exhaust delay time, inner delay time, inner fan work time).

## Operating modes

Select modes with:

```gcode
SET_PURIFIER_MODE MODE=<0–3> ...
```

Key parameters:

- `FAN_SPEED`
- `DESIRE_TEMP`
- `ALARM_TEMP`
- `DELAY_OFF`
- `DYNAMIC_FAN_CONTROL`

Use:

`WAIT_CHAMBER_TEMP`

to block until target temperature.

Additional key parameters:

- `DESIRE_TEMP`
- `FAN_SPEED`
- `DELAY_OFF`

Additional key parameters:

- `DESIRE_TEMP`
- `FAN_SPEED`
- `DELAY_OFF`

## Basic fan control

`SET_PURIFIER`

Basic fan control and delay settings.

```gcode
SET_PURIFIER FAN=<exhaust|inner> SPEED=<speed> DELAY_OFF=<seconds> WORK=<seconds>
```

`FAN`  
`exhaust` or `inner`

`SPEED`  
0.0–1.0; sets fan speed directly.

`DELAY_OFF`  
delay before the fan actually turns off once speed is zero.

`WORK`  
inner fan accumulated work time (seconds), stored in `purifier.json`

## Status and mode commands

`GET_PURIFIER`

`SET_PURIFIER_MODE`

Selects and configures operating mode 0–3.

```gcode
SET_PURIFIER_MODE MODE=<0|1|2|3> [mode-specific parameters]
```

Parameters mentioned in the manual:

- `EXHAUST_FAN_DELAY_OFF`
- `INNER_FAN_DELAY_OFF`
- `FAN_SPEED`
- `DESIRE_TEMP`
- `ALARM_TEMP`
- `DELAY_OFF`
- `DYNAMIC_FAN_CONTROL`

Additional mode parameters:

- `DESIRE_TEMP`
- `FAN_SPEED`
- `DELAY_OFF`

Additional mode parameters:

- `DESIRE_TEMP`
- `FAN_SPEED`
- `DELAY_OFF`

## Chamber wait

`WAIT_CHAMBER_TEMP`

Waits for chamber temperature conditions with a timeout.

```gcode
WAIT_CHAMBER_TEMP TIMEOUT=<seconds>
```

## Web endpoint

The endpoint `control/purifier` accepts `fan`, `speed`, `delay`, and `work` fields similar to `SET_PURIFIER`.

`fan`  
`exhaust` or `inner`

`speed`  
percentage 0–100; converted to 0.0–1.0 internally.

`delay`  
delay in seconds used for delayed shutoff.

`work`  
inner fan work time, stored persistently.

Returns JSON:

```json
{"state": "success"}
```

on success, or

```json
{"state": "error", "message": "..."}
```

on failure.

## Examples

```gcode
SET_PURIFIER_MODE MODE=1 FAN_SPEED=0.6 DESIRE_TEMP=40 ALARM_TEMP=45 DELAY_OFF=180 DYNAMIC_FAN_CONTROL=1
```

```gcode
SET_PURIFIER_MODE MODE=2 DESIRE_TEMP=50 FAN_SPEED=0.6 DELAY_OFF=180
WAIT_CHAMBER_TEMP TIMEOUT=300
```

```gcode
SET_PURIFIER FAN=exhaust SPEED=0.8 DELAY_OFF=180
SET_PURIFIER FAN=exhaust SPEED=0
```
