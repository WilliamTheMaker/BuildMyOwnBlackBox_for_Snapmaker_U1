# Snapmaker U1 Black Box Dual PWM Fan Mod

An unofficial Snapmaker U1 mod project to build a custom electronic **Black Box** for two PWM fans, based on reverse-engineering the purifier-related Klipper configuration and control logic.

The key discovery came from comparing the old purifier config with the newer V1.3.0 version. In the old config, `PA9` was used as `tachometer_pin`, while in the new config it became `inner_pin`, and tach feedback moved to `PA6` as `inner_tach_pin`.

That role change on `PA9` is the main reason this project exists.

## Project goal

The goal is to convert the original `PA9` tach-oriented signal path into a usable PWM control path for the inner fan.

Instead of waiting for the official top-cover hardware, I am building my own external Black Box and using the firmware direction that is already visible in the purifier stack.

## Background

I have been building custom 3D printers since 2013, starting with a Delta Mini.

My Voron V2.4 was built in 2021 and evolved through several iterations into a wide-edition 6AWD IDEX machine.

The Snapmaker U1 is the first consumer 3D printer I have purchased. I bought it as a backup machine so I could still print structural parts while my custom Voron was disassembled.

## Firmware findings

Old purifier config:
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

New purifier config in V1.3.0:
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

What changed:

- `PA8` remains a fan-control output.
- `PA9` changed from tach input to PWM output for the inner fan.
- `PA6` became the inner fan tach feedback pin.

## Hardware concept

The Black Box sits between the 6-pin header at the top of the Snapmaker U1 and two PWM fans:

- One exhaust fan
- One inner air-circulation fan

The exhaust path is planned to include a servo-controlled vent shutter so the air outlet can be shut and kept air-tight when the system is off.

The inner fan is intended for chamber air circulation and follows the stock purifier concept, where it is paired with HEPA and active-carbon filtration.

The current `PA9` conversion stage uses a simple transistor interface:

- `Q1`: 2N3904
- `R1`: 10k from `PA9` to base
- `R2`: 4.7k pull-up to `+5V`
- collector output to the fan PWM wire
- shared ground with the printer

See: [`docs/hardware-plan.md`](docs/hardware-plan.md)

## User control

The purifier module supports direct control of both fans.

Basic examples:
```gcode
SET_PURIFIER FAN=exhaust SPEED=0.8 DELAY_OFF=180
SET_PURIFIER FAN=inner SPEED=0.6 DELAY_OFF=120
SET_PURIFIER FAN=exhaust SPEED=0
SET_PURIFIER FAN=inner SPEED=0
GET_PURIFIER
```

Mode-based control is also available:
```gcode
SET_PURIFIER_MODE MODE=1 FAN_SPEED=0.6 DESIRE_TEMP=40 ALARM_TEMP=45 DELAY_OFF=180 DYNAMIC_FAN_CONTROL=1
SET_PURIFIER_MODE MODE=2 DESIRE_TEMP=50 FAN_SPEED=0.6 DELAY_OFF=180
WAIT_CHAMBER_TEMP TIMEOUT=300
```

See: [`docs/user-manual.md`](docs/user-manual.md)

## AI-assisted learning

This project is also about learning with AI.

AI helped me understand digital I/O, MCU pin behavior, PWM fan control, firmware logic, and circuit ideas faster. It did not replace hands-on testing, but it made the loop of reading, questioning, testing, and revising much more efficient.

## Support

If this project is useful to you and you want to support continued development, you can support me via **Ko-fi**.

This support helps fund prototyping costs, parts, test hardware, PCB revisions, and time spent documenting the project.

> Replace this line with your Ko-fi link:  
> `https://ko-fi.com/YOUR_NAME`

## Commercial use

This project is shared for documentation, learning, and community discussion.

Commercial manufacturing, resale, redistribution of design files, or use of this project in products for sale is **not permitted** without my prior written permission.

If you are interested in licensing, collaboration, sponsored development, or an official manufacturing arrangement, please contact me directly.

## Repo layout

```text
README.md
LICENSE.md
docs/
  reverse-engineering-notes.md
  hardware-plan.md
  user-manual.md
images/
  pa9-to-pwm-conversion.jpg
```

## Related docs

- [`docs/reverse-engineering-notes.md`](docs/reverse-engineering-notes.md)
- [`docs/hardware-plan.md`](docs/hardware-plan.md)
- [`docs/user-manual.md`](docs/user-manual.md)

## Disclaimer

This is an unofficial modding and learning project.

Anyone reproducing this work should verify pinout, grounding, power rail selection, current limits, and fan compatibility before connecting hardware to a printer.

## License

This repository is currently shared under an **All Rights Reserved** model.

Please read [`LICENSE.md`](LICENSE.md) before reusing any part of this project.
