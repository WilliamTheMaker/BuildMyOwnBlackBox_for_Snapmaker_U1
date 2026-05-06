# Hardware Plan

## Overview

This document outlines the hardware direction for the Snapmaker U1 Black Box Dual PWM Fan Mod.

The Black Box is designed to connect between the 6-pin header on the top of the Snapmaker U1 and two PWM fans:

- One exhaust fan
- One inner air-circulation fan

The exhaust path is planned to include a vent shutter so the air outlet can be shut and kept air-tight when the system is off.

The inner air-circulation path follows the stock purifier concept, where the fan works with HEPA and active-carbon filtration.

## Core function

The most important function of the Black Box is converting the old `PA9` tach-oriented path into a practical PWM control path for the inner fan.

This design follows the firmware change where the old config used `tachometer_pin: PA9`, while the new config uses `inner_pin: !PA9` and moves tach feedback to `PA6`.

## Current conversion circuit

The present proof-of-concept uses a simple NPN transistor stage:

- `Q1`: 2N3904
- `R1`: 10k base resistor from `PA9`
- `R2`: 4.7k pull-up resistor to `+5V`
- collector output to the fan PWM wire
- shared ground with the printer

The idea is to create an open-collector style interface that better matches a standard 4-wire fan PWM input.

## Why a custom PCB

I am now working on designing my own PCB in KiCad.

This is also a self-learning goal. I have never designed a PCB before, and I am not a certified electronics engineer.

That is part of the value of the project: it combines firmware reverse-engineering, circuit experimentation, and first-time PCB design into one practical build.

## PCB roadmap

The purpose of making my own PCB is to integrate and add:

1. A conversion circuit that changes `PA9` from tach to PWM.
2. A 24V/12V fan selector.
3. An Arduino or ESP32 board.
4. Servo control for a vent shutter.
5. A DC-to-DC buck converter.
6. A PPTC 24V 1A fuse on the input power rail.

This PCB is meant to turn the current proof-of-concept wiring into a compact and expandable system.

## Planned features

### 1. PA9 tach-to-PWM conversion

This is the core electrical task.

The board needs to provide a reliable PWM interface so the new firmware-side role of `PA9` can drive the inner fan properly.

### 2. 24V / 12V fan selector

The board should support both 24V and 12V fan options.

This gives more flexibility in testing, sourcing, and future upgrades.

### 3. Arduino / ESP32 support

The PCB should reserve space or headers for an Arduino-class board or an ESP32.

This allows future logic, monitoring, signal processing, and automation.

### 4. Servo control for vent shutter

The exhaust path should support a servo-driven shutter.

The goal is not only airflow control during operation, but also sealing the outlet when the exhaust fan is off.

### 5. DC-to-DC buck converter

The board should include a buck converter stage for lower-voltage rails needed by fans, logic boards, or servo hardware.

### 6. PPTC 24V 1A fuse

The input power rail should include a resettable PPTC fuse rated for 24V and 1A.

This provides a basic layer of protection for the Black Box during development and fault conditions.

## Build stages

### Stage 1: Bench validation

- Validate `PA9` behavior
- Verify the transistor stage output
- Confirm fan response across speed range
- Check grounding and pull-up behavior

### Stage 2: Wired prototype

- Integrate both fan channels
- Test exhaust and inner fan separation
- Check tach feedback on `PA6`
- Verify wiring and connectors

### Stage 3: KiCad design

- Draw the full schematic
- Define connectors and power rails
- Add the transistor interface
- Reserve area for MCU integration
- Plan fan-voltage selection and servo support

### Stage 4: PCB prototype

- Assemble the first revision
- Validate power conversion
- Test fan and servo behavior
- Record issues and revise layout

## Related docs

- [`../README.md`](../README.md)
- [`reverse-engineering-notes.md`](reverse-engineering-notes.md)
- [`user-manual.md`](user-manual.md)
