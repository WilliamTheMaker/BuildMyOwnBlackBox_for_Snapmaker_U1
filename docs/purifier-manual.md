# Purifier Klipper Module - User Manual (Snapmaker U1/ Klipper Integration)

## 1\. What the purifier module does

The purifier module controls two fans (exhaust and inner) and their power for your Snapmaker/Klipper setup. It monitors chamber temperature and fan RPM, can automatically adjust fan speed, and can pause printing if cooling/heating fails or becomes unsafe.

## 2\. Hardware the module expects

*   **Exhaust fan**: Chamber exhaust fan driven by a PWM pin configured as `exhaust_pin`.
*   **Inner fan**: Internal circulation fan on a PWM pin configured as `inner_pin`.
*   **Tachometer input for Inner fan**: RPM feedback pins for inner fan, using a pulse counter and configurable pulses-per-revolution (PPR).
*   **Power enable pin**: Digital output that turns purifier power on when any fan is running, and off when both fans stop.  
    REMARKS: **power\_enable\_pin** must be well defined in \[purifier\] module, otherwise "\[purifier\] power enable pin not exist!" will be logged
*   **Power detect input**: ADC input that checks if the purifier hardware is present and powered.
*   **External temperature sensor**: A `temperature_sensor` defined in Klipper, used as the chamber temperature source.  
    REMARKS: Stock firmware V1.3.0 defined "cavity" as default, which is actually Snapmaker U1 built-in chamber temperator sensor

The module stores persistent settings in `purifier.json` under the printer’s persistent config directory (exhaust delay time, inner delay time, inner fan work time).

## 3\. Fan behavior and power logic

*   **Speed range**: Speeds are 0.0–1.0 internally, where 1.0 equals the configured maximum power.  
    Example: 0.1 = 10%, 0.4 = 40% and 1 = 100% full speed respectively
*   **Kick-start**: On start, the fan can be briefly driven at full power to ensure reliable spin-up, then drop to the requested speed.
*   **Power enable**: If either fan’s speed is > 0, the power enable pin is set high; if both are zero, it is set low.
*   **Power detection**: If purifier system is detached or pull-down resistor is not detected, 24V power will be off, both fans are turned off, mode resets to IDLE, and a log notes the purifier is offline.

## 4\. Operating modes

Select modes with `SET_PURIFIER_MODE MODE=<0–3> ...`.

### 4.1 Mode 0 – IDLE

*   Lets fans run down after use with configurable exhaust and inner fan delay times.
*   Resets chamber-related state and schedules fans to stop after their delay.

### 4.2 Mode 1 – COOL CHAMBER

*   Cools the chamber down to a target temperature using the exhaust fan.
*   Uses chamber temperature feedback to adjust exhaust fan speed.
*   Raises an error and pauses the print if the chamber remains too hot beyond a timeout.

Key parameters: `FAN_SPEED`, `DESIRE_TEMP`, `ALARM_TEMP`, `DELAY_OFF`, `DYNAMIC_FAN_CONTROL`.

### 4.3 Mode 2 – PREHEAT CHAMBER

*   Helps warm the chamber before printing using the inner fan; exhaust fan is off.
*   Tracks temperature rise over time.
*   Often combined with `WAIT_CHAMBER_TEMP` to block until target temperature.

Key parameters: `DESIRE_TEMP`, `FAN_SPEED`, `DELAY_OFF`.

### 4.4 Mode 3 – HOT CHAMBER

*   Maintains a hot chamber during printing while checking inner fan health.
*   Inner fan runs, exhaust fan is off, RPM is monitored.
*   If fan is enabled but RPM stays zero long enough, an error ("0002-0533-0000-0000") is prompted and the print is paused.

Key parameters: `DESIRE_TEMP`, `FAN_SPEED`, `DELAY_OFF`.

## 5\. G-code commands

### 5.1 `SET_PURIFIER`

Basic fan control and delay settings.

```
SET_PURIFIER FAN=<exhaust|inner> SPEED=<speed> DELAY_OFF=<seconds> WORK=<seconds>
```

*   `FAN`: `exhaust` or `inner`.
*   `SPEED`: 0.0–1.0; sets fan speed directly.
*   `DELAY_OFF`: delay before the fan actually turns off once speed is zero.
*   `WORK`: inner fan accumulated work time (seconds), stored in `purifier.json`.

### 5.2 `GET_PURIFIER`

*   Prints current status: power detection, voltage, fan speed and RPM, delay times, inner fan work time.
*   Also shows mode, desired and critical temperatures, average temperature, and fault flags.

### 5.3 `SET_PURIFIER_MODE`

Selects and configures operating mode 0–3.

```
SET_PURIFIER_MODE MODE=<0|1|2|3> [mode-specific parameters]
```

*   Mode 0 (IDLE): `EXHAUST_FAN_DELAY_OFF`, `INNER_FAN_DELAY_OFF`.
*   Mode 1 (COOL CHAMBER): `FAN_SPEED`, `DESIRE_TEMP`, `ALARM_TEMP`, `DELAY_OFF`, `DYNAMIC_FAN_CONTROL`.
*   Mode 2 (PREHEAT CHAMBER): `DESIRE_TEMP`, `FAN_SPEED`, `DELAY_OFF`.
*   Mode 3 (HOT CHAMBER): `DESIRE_TEMP`, `FAN_SPEED`, `DELAY_OFF`.

### 5.4 `WAIT_CHAMBER_TEMP`

Waits for chamber temperature conditions with a timeout.

```
WAIT_CHAMBER_TEMP TIMEOUT=<seconds>
```

*   Requires purifier to be detected; otherwise returns immediately with a message, "\[purifier\] Cannot wait for temperature: purifier not detected!"
*   Enables internal preheat-wait state and optionally overrides timeout.
*   Periodically reports current temperature, target, remaining timeout, last temperature, and remaining time.

## 6\. Webhook API

The endpoint `control/purifier` accepts `fan`, `speed`, `delay`, and `work` fields similar to `SET_PURIFIER`.

*   `fan`: `exhaust` or `inner`.
*   `speed`: percentage 0–100; converted to 0.0–1.0 internally.
*   `delay`: delay in seconds used for delayed shutoff.
*   `work`: inner fan work time, stored persistently.

Returns JSON: `{"state": "success"}` on success, or `{"state": "error", "message": "..."}` on failure.

## 7\. Safety and automatic pause

*   **Fan RPM fault**: if the inner fan is commanded on but RPM stays zero long enough, a structured error is raised and the print is paused.
*   **Cooling failure**: in COOL CHAMBER mode, if chamber temperature cannot be reduced below critical thresholds within the timeout, an error is prompted, "Failed to cool chamber temperature, Chamber temperature is too high! Current temperature: XX , Critical temperature: XX, 0002-0533-0000-0001" and the print is paused.
*   **Power loss**: if purifier power is lost, both fans are turned off, mode resets to IDLE, and the purifier is logged as offline.

## 8\. Typical usage examples

### 8.1 Cool chamber after a print

```
SET_PURIFIER_MODE MODE=1 FAN_SPEED=0.6 DESIRE_TEMP=40 ALARM_TEMP=45 DELAY_OFF=180 DYNAMIC_FAN_CONTROL=1
```

### 8.2 Preheat chamber before a print

```

SET_PURIFIER_MODE MODE=2 DESIRE_TEMP=50 FAN_SPEED=0.6 DELAY_OFF=180
WAIT_CHAMBER_TEMP TIMEOUT=300
```

### 8.3 Simple manual fan control

```

SET_PURIFIER FAN=exhaust SPEED=0.8 DELAY_OFF=180
SET_PURIFIER FAN=exhaust SPEED=0
```
