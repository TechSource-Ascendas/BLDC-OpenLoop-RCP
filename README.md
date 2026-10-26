# BLDC Open-Loop Control with Speedgoat IO397 and DRV8305

This repository contains a Simulink Real-Time example for open-loop, six-PWM control of a three-phase BLDC motor. It targets a Speedgoat real-time machine fitted with an IO397-50k module, a Texas Instruments BOOSTXL-DRV8305EVM inverter, and a MOONS 42BL30L2 motor.

The model generates a three-phase voltage command, sends six complementary PWM signals to the DRV8305 EVM, measures phase-A and phase-B current, and can measure rotational speed from one motor Hall channel.

> **Safety:** This is an open-loop demonstration, not a production motor drive. Begin with a current-limited, low-voltage DC supply. Keep the gate driver disabled until all PWM, ground, power, and feedback connections have been checked with the DC bus off.

## Repository contents

| File | Purpose |
|---|---|
| `speedgoat_IO397_RCP_BLDC_openloop.slx` | Main Simulink Real-Time model |
| `speedgoat_IO397_RCP_OpenLoop_Initialize.mat` | Model parameters and initialization data |
| `speedgoat_drv8305_wiring_interface.md` | Detailed pin mapping, wiring notes, and commissioning procedure |
| `speedgoat_drv8305_wiring_diagram_v2.png` | Speedgoat-to-DRV8305 wiring diagram |
| `speedgoat_42bl30l2_hall_speed_wiring.png` | 42BL30L2 Hall-to-Speedgoat speed measurement diagram |

## Wiring diagrams

### Speedgoat IO397 to DRV8305 EVM

![Speedgoat IO397 to DRV8305 EVM wiring](speedgoat_drv8305_wiring_diagram_v2.png)

### Motor Hall speed measurement

![42BL30L2 Hall speed wiring](speedgoat_42bl30l2_hall_speed_wiring.png)

See [`speedgoat_drv8305_wiring_interface.md`](speedgoat_drv8305_wiring_interface.md) for the complete pin tables and electrical notes.

## Hardware requirements

### Required

- Speedgoat real-time target machine with an **IO397-50k** configurable I/O module.
- Host computer connected to the Speedgoat target over Ethernet.
- Texas Instruments **BOOSTXL-DRV8305EVM** three-phase inverter board.
- MOONS **42BL30L2** BLDC motor:
  - 24 VDC
  - 30 W
  - 6 poles / 3 pole pairs
  - 4000 rpm rated speed
  - 5000 rpm maximum speed
- Current-limited DC power supply suitable for the motor and inverter. Use a reduced bus voltage and conservative current limit for initial commissioning.
- Wiring for six PWM signals, gate enable, digital ground, motor phases, Hall supply/ground, and the selected Hall output.
- Appropriate mating connectors or breakout cables for the Speedgoat IO397 and DRV8305 EVM headers.

### Strongly recommended

- Oscilloscope with suitable probes for checking PWM signals, dead time, Hall output level, and gate-driver behavior.
- Bench multimeter.
- Fuse or resettable protection in the DC-bus supply path.
- Accessible emergency stop or DC-bus disconnect.
- Mechanical restraint or guarded motor mounting.
- Logic-level conditioning or a pull-up resistor if the measured Hall output requires it.

### Optional protection connections

- DRV8305 `nFAULT` to Speedgoat Digital B pin 15 / DIO13 (`*Optional`). The present model does **not** act on this signal; a digital input block and shutdown logic must be added before it can automatically disable `EN_GATE`.
- DRV8305 `PWRGD` to a spare input or oscilloscope (`*Optional`).

## Software requirements

This repository was inspected and verified with the following installed environment:

| Product | Verified version |
|---|---|
| MATLAB | R2025b Update 5 |
| Simulink | R2025b |
| Simulink Real-Time | R2025b |
| Simulink Coder | R2025b |
| Motor Control Blockset | R2025b |
| C2000 Microcontroller Blockset | R2025b; reported by model dependency analysis |
| Speedgoat I/O Blockset | 9.11.1 for R2025b |
| Speedgoat I/O Configuration Package | 9.11.1 with the IO397 RCP configuration |

The model uses the Simulink Real-Time system target file `speedgoat.tlc` and the prebuilt IO397 RCP bitstream:

```text
C:\ProgramData\Speedgoat\bitstreams\io_configuration_package\R2025b\io397\speedgoat_IO397_RCP.mat
```

If another MATLAB release is used, install the matching Speedgoat packages and select the corresponding release-specific bitstream path in the model's **Setup** block. HDL Coder is not required when using the supplied prebuilt bitstream; it is needed only if the FPGA configuration is customized and rebuilt.

## Model configuration

The supplied model is configured as follows:

- Simulink Real-Time target: `speedgoat.tlc`
- Solver: fixed-step discrete
- Stop time: infinite
- Control/PWM sample time: 50 microseconds
- PWM frequency: 20 kHz
- FPGA clock: 75 MHz
- Motor pole pairs: 3
- Initial speed reference in the model: 500 rpm
- Initial voltage amplitude (`Boost Voltage`): 0.15 per unit
- Initial `Start` value: 0, which keeps the drive disabled

The initialization MAT-file defines `PWM_frequency`, `T_pwm`, `Ts`, `dataType`, `motor`, `inverter`, and `target`. Its `target.model` field contains a legacy C2000 value and does not select the Speedgoat real-time target; the Simulink model configuration and Simulink Real-Time target object control deployment.

## First-time setup

1. Install MATLAB, the required MathWorks products, and the matching Speedgoat software packages.
2. Connect the host computer to the Speedgoat target and verify that it appears in Simulink Real-Time Explorer.
3. Confirm that the IO397 RCP bitstream file exists at the path configured in the model's **Setup** block.
4. Complete the power, PWM, enable, ground, motor phase, and feedback wiring using the supplied diagrams.
5. Keep the DC bus switched off and leave the model `Start` value at `0`.
6. Open MATLAB and make this repository the current folder.

Load the initialization data and open the model:

```matlab
load('speedgoat_IO397_RCP_OpenLoop_Initialize.mat');
open_system('speedgoat_IO397_RCP_BLDC_openloop');
```

## Build and deploy

1. In the model, open the **Setup** block and confirm:
   - Board: `IO397-50k`
   - Bitstream: `speedgoat_IO397_RCP.mat`
   - Bitstream directory matches the installed MATLAB/Speedgoat release.
2. Select the correct Speedgoat target in Simulink Real-Time.
3. Build the real-time application from the **Real-Time** tab.
4. Load the application onto the Speedgoat target.
5. Start the real-time application while `Start = 0` and the inverter DC bus remains off.
6. Check that the application runs without I/O or bitstream errors.

## Motor and power-stage connections

The initial phase assignment is:

| 42BL30L2 phase connector | DRV8305 EVM |
|---|---|
| Pin 2, orange — `U` | J4 pin 3 — `MOT_A` |
| Pin 3, yellow — `V` | J4 pin 2 — `MOT_B` |
| Pin 1, blue — `W` | J4 pin 1 — `MOT_C` |

If the rotation direction is opposite to the requested direction, change the commanded phase sequence. Do not randomly exchange Hall wires.

## Hall speed measurement

QAE is a quadrature **encoder generator** and is not used for measuring this motor. The model uses a QAD block to decode an incoming signal.

The motor's `Hu`, `Hv`, and `Hw` signals are 120-electrical-degree commutation signals, not 90-degree quadrature A/B signals. For speed-only measurement, use one Hall channel:

| 42BL30L2 Hall connector | Speedgoat IO397 |
|---|---|
| Pin 1, red — `VCC` | Digital B pin 2 — `+5 VDC` |
| Pin 5, black — `GND` | Digital B pin 17 — `GND` |
| Pin 6, orange — `Hu` | Digital B pin 10 — QAD channel 1 `A` |
| Pin 3, yellow — `Hv` | Not connected; insulate the wire |
| Pin 4, blue — `Hw` | Not connected; insulate the wire |
| Pin 2 | NC |

Configure QAD channel 1 as follows:

| Parameter | Setting |
|---|---|
| Captured Signal Edges | `Capture Rising Edges A Signal Only` |
| Number of Slots Vector | `3` |
| Index Control Mode | `Ignore Index` |
| QAD-B and QAD-index | Leave unconnected |

For one Hall channel and three pole pairs:

```text
RPM = 60 × Hall frequency / 3
RPM = 20 × Hall frequency
```

Expected Hall frequencies are approximately 200 Hz at 4000 rpm and 250 Hz at 5000 rpm.

The motor drawing does not specify whether the Hall outputs are push-pull or open-collector. Before connecting `Hu` to the IO397, power the Hall sensors and inspect `Hu` with an oscilloscope. Match the IO397 3.3 V/5 V TTL pull configuration to the observed output type and voltage.

## Safe commissioning procedure

1. Switch off and discharge the inverter DC bus.
2. Verify continuity and pin numbers for all six PWM wires, `EN_GATE`, digital ground, motor phases, current feedback, and Hall wiring.
3. Keep `Start = 0` and initially disconnect the motor power stage if practical.
4. Run the real-time application and inspect all six PWM signals with an oscilloscope.
5. Verify complementary high/low operation and adequate non-overlap before applying significant bus voltage.
6. Apply a low, current-limited DC-bus voltage. Confirm the EVM supply rails, `PWRGD`, and `nFAULT` state.
7. Connect or energize the motor with the voltage amplitude at its low initial value and the speed reference at a conservative value.
8. Change `Start` to `1` only after the PWM and feedback signals are valid.
9. Increase voltage amplitude and speed gradually while monitoring DC-bus current, phase-current feedback, motor temperature, and measured speed.
10. If `nFAULT` asserts, current rises unexpectedly, the motor stalls, or the phase signals are incorrect, set `Start` to `0` and remove DC-bus power immediately.

## Current feedback notes

The model reads phase-A and phase-B current from the DRV8305 EVM. It uses offsets of 1.6945 V and 1.6930 V and a scale factor of 14.286 A/V. With the motor power disabled, confirm that both displayed current values are close to zero. Recalibrate the offsets if necessary before powered testing.

## Known limitations

- Control is open-loop; the measured Hall speed is for observation and is not used to close the speed loop.
- The model does not currently process `nFAULT` or `PWRGD`.
- The IO397 RCP bitstream does not provide SPI control for this example. The DRV8305 register configuration and detailed diagnostics are therefore not managed by the model.
- FPGA PWM deadband is configured as zero. Confirm that the DRV8305 dead-time/handshake behavior provides safe switching before increasing bus voltage.
- Only two phase-current signals are measured.
- One Hall channel provides speed magnitude but not independent direction information.

## Reference hardware

- [MOONS 42BL30L2 motor](https://www.moonsindustries.com/p/s42-series-brushless-dc-motors/42bl30l2-000004611130000001)
- [Speedgoat IO397 module](https://www.speedgoat.com/products/simulink-programmable-fpgas-fpga-i-o-modules-io397)
- [TI BOOSTXL-DRV8305EVM](https://www.ti.com/tool/BOOSTXL-DRV8305EVM)

