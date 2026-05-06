# Reverse Engineering Notes

## Overview

This document records the firmware-side findings behind the Snapmaker U1 Black Box project.

The project started by comparing the old purifier config with the newer V1.3.0 version. The most important result is that `PA9` changed from a tachometer input role into a PWM control role, while tach feedback moved to `PA6`.

## Config comparison

### Old purifier config

```ini
[purifier]
pin: !PA8
max_power: 1.0
tachometer_pin: PA9
tachometer_ppr: 2
tachometer_poll_interval: 0.0005
enable_pin: PE15
extra_fan_tach_pin: PA6
power_det_pin: PA7
power_det_threshold: 0.88
```

### New purifier config in V1.3.0

```ini
[purifier]
# exhaust fan
exhaust_pin: !PA8
exhaust_max_power: 1.0
exhaust_cycle_time: 0.001
exhaust_hardware_pwm: True
# inner fan
inner_pin: !PA9
inner_max_power: 1.0
inner_cycle_time: 0.001
inner_hardware_pwm: True
inner_tach_pin: PA6
inner_tach_ppr: 2
inner_tach_poll_interval: 0.0005
# global
power_enable_pin: PE15
power_det_pin: PA7
power_det_threshold: 0.88
external_temp_sensor: cavity
```

## Key findings

### 1. The purifier design became dual-channel

The old config uses one main control output on `PA8`.

The new config splits fan control into:

- `exhaust_pin: !PA8`
- `inner_pin: !PA9`

That indicates a move toward two independently controlled fan channels.

### 2. `PA9` changed role

Old config:
- `tachometer_pin: PA9`

New config:
- `inner_pin: !PA9`

This is the main trigger for the project. It suggests that `PA9`, previously used for fan-speed feedback, was reassigned as the PWM control output for the inner fan.

### 3. Tach feedback moved to `PA6`

Old config:
- `extra_fan_tach_pin: PA6`

New config:
- `inner_tach_pin: PA6`

This suggests `PA6` now carries the inner fan tach feedback function.

### 4. Hardware PWM is explicit

The new config adds:

- `exhaust_hardware_pwm: True`
- `inner_hardware_pwm: True`

That is a strong sign that both fan channels were intended to be driven as hardware PWM outputs.

## Why it matters

This config comparison turns the idea from guesswork into a practical hardware plan.

It supports the following interpretation:

- `PA8` stays as one fan-control output.
- `PA9` is repurposed from tach input into a second fan-control output.
- `PA6` becomes the tach input for the inner fan.

That role change on `PA9` is the basis for the Black Box conversion circuit.

## Module behavior

The attached purifier manual confirms that the module now controls two fans, `exhaust` and `inner`, and supports direct commands, mode-based control, chamber-temperature waiting, and a web endpoint for integration.

See: [`purifier-manual.md`](purifier-manual.md)

## Conclusion

The firmware path appears ready enough to justify hardware development now.

That is why this project focuses on building the electrical interface needed to turn the new `PA9` role into a usable real-world PWM fan control path.
