# Setting Up Cartographer Probe on CAN

This guide will walk you through the installation and setup of the Cartographer probe for Klipper over CANbus.

---

## Prerequisites

* Cartographer probe (hardware)
* Pre-installed CANboot firmware
* CANbus already configured and working (`can0` interface active)
* Klipper host VM running in Proxmox
* Cartographer mounted and wired properly to your CAN bus

---

## Step 1: Flash Cartographer with Cartographer Firmware

Follow the official guide here: [Cartographer Firmware Switching Guide](https://docs.cartographer3d.com/cartographer-probe/firmware/firmware-switching/usb-to-canbus)

In summary:

1. Connect the Cartographer via USB
2. Flash Cartographer CANboot firmware using `flash_can.py`
3. Switch to Cartographer CAN firmware (not Klipper)

Use the official [Cartographer GitHub repo](https://github.com/Cartographer3D) for all firmware and instructions.

> **Note:** UUIDs are dynamically assigned — do not hardcode them in your config. Detect UUIDs using `detect-can-devices.sh`.

---

## Step 2: Wire Cartographer to CANbus

* Connect Cartographer CAN\_H and CAN\_L to the shared CANbus
* Ensure 120Ω termination exists at both ends of the bus
* Supply 24V to Cartographer's VIN/GND

---

## Step 3: Add Cartographer Configuration to Klipper

Create a separate file: `cartographer.cfg`

```ini
[cartographer]
canbus_uuid: <auto-detect at runtime>
bus: can0
probe_offset: # Use values obtained from calibration

[adxl345]
bus: spi
cs_pin: !<your-ebb-cs-pin>
axes_map: -1,-2,3
```

> **Note:** The ADXL345 is pre-mounted on Cartographer. No separate wiring required.

---

## Step 4: Calculate Probe Offsets

Use the Cartographer’s `cartographer_touch` utility to calculate offsets.

1. Run the probing sequence
2. Let the firmware calculate the X, Y, Z offsets from the nozzle
3. Update the `probe_offset` field in `cartographer.cfg`

Example:

```ini
probe_offset: 10.1, 5.3, -1.92
```

---

## Step 5: Enable Cartographer in Your Printer Config

Include your Cartographer config in `printer.cfg`:

```ini
[include config/cartographer.cfg]
```

Also include macros and any related probing setup.

---

## Step 6: Validate Functionality

1. Home all axes
2. Run a test probe or mesh level sequence
3. Monitor probe behavior in Mainsail or Fluidd
4. If issues arise, check logs and run `detect-can-devices.sh` again

---

## Troubleshooting

* Check power (24V) to Cartographer
* Verify CAN termination and wiring
* Ensure `can0` interface is up
* Check for UUID collisions (use `detect-can-devices.sh`)

---

Once validated, Cartographer is ready for use as your primary Z-probe and accelerometer source for input shaper calibration.

