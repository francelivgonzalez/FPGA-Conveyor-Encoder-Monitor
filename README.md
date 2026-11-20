# FPGA Conveyor Encoder Monitor
> Catching unexpected motor stops on FPGA hardware, distinguishing a commanded stop from a sudden loss of motion using quadrature decoding, speed estimation, and serial timing logs.

**Status:** Planned
**Projected timeline:** August 16-September 30, 2027
**Hardware:** Digilent Cora Z7-07S, DC gearmotor with a quadrature encoder, motor driver, and matching power supply
**Tools:** SystemVerilog, AMD Vivado, and Python for serial logging

## What I’m building

I’ll build an FPGA monitor that checks whether a motor shaft is moving when it has been told to run. This is a bench-scale version of a conveyor monitoring problem. The encoder on my first rig will measure the motor shaft directly.

The main test is two stops that look the same at the encoder. In one, I tell the motor to stop. In the other, the run command stays active and the motor loses power. The monitor should flag only the unexpected stop.

## How it will work

The encoder produces two pulse signals, A and B. Their order tells me the direction of rotation, and their timing lets me estimate speed. I’ll read both signals through compatible inputs on the Cora Z7 and compare the measured motion with the run command.

I’ll try two ways of measuring speed:

- Count encoder pulses during a fixed time window.
- Measure the time between consecutive pulses.

Low speed is the case I’m most interested in. A short window can contain zero pulses even while the shaft is turning. If the shaft stops completely, there is no next pulse to measure. I’ll use the bench data to choose a timeout and see how that choice affects false alarms.

## What I’ll test

| Test | Expected behavior |
|---|---|
| Start from rest | Allow time for the motor to begin moving. |
| Run at three steady speeds | Report speed and record any false alarms. |
| Slow down while RUN stays active | Flag underspeed after the configured delay. |
| Issue a normal STOP command | Let the shaft coast down without an unexpected-stop alarm. |
| Disable the driver while RUN stays active | Flag loss of motion after the encoder timeout. |

## How I’ll measure it

The FPGA will timestamp the command change, driver-disable signal, last encoder edge, and alarm. A Python script will save those timestamps to CSV over USB serial. This lets me measure two different intervals: how long the motor coasts and how long the monitor takes to react after the final encoder pulse.

## Timeline

- **August 16-31:** Write the encoder decoder and monitor logic. Test synthetic pulse sequences in Vivado.
- **September 1-15:** Connect the motor rig, verify encoder counts per shaft revolution, and record normal runs.
- **September 16-30:** Run the slowdown and stop tests, compare both speed estimates, and record the demo.

## Results I plan to publish

A bench demo, a wiring diagram with the actual parts and signal levels, and timing plots for a normal stop and an unexpected stop. I’ll report detection delay and false alarms at each tested speed.

## Planned folders

- `hardware/` - parts list, wiring diagram, and pin assignments
- `rtl/` - encoder decoder, speed estimators, and alarm logic
- `tb/` - simulated encoder signals and test cases
- `vivado/` - Cora Z7 project scripts and constraints
- `scripts/` - Python serial logger and plotting code
- `results/` - bench data, timing plots, and demo
