# Setting Up CAN Interface Inside the Klipper VM

This guide explains how to set up the CAN interface using an Octopus Pro board connected via USB and acting as the CAN-USB bridge using CANBoot and Klipper firmware. All setup is done within the Klipper VM on Proxmox.

---

## Overview

* **CAN Host Interface**: The Octopus Pro 1.1 board serves as the CAN bridge and is connected to the Klipper VM via USB passthrough.
* **Interface Name**: The CAN interface is named `can0`.
* **Bitrate**: 1,000,000 bps (1 Mbps) is used for all devices.
* **UUIDs**: Device UUIDs vary per setup — do **not** hardcode them. Use dynamic detection.

---

## Step 1: Verify USB Passthrough

Ensure your Octopus board is passed through to the VM as a USB device.

* In Proxmox GUI, select the Klipper VM → *Hardware* → *Add* → *USB Device*
* Choose your Octopus device from the list (vendor ID is usually `0483` for STM32).
* Reboot the VM after passthrough is configured.

---

## Step 2: Enable CAN Interface in VM

Run the setup script to enable the `can0` interface using `slcan`:

```bash
sudo ./scripts/setup-can-interface.sh
```

This script should:

* Identify the `/dev/serial/by-id/` path to the Octopus
* Attach the interface using `slcand`
* Bring up the `can0` interface at 1 Mbps

You can also manually run the steps if needed:

```bash
sudo modprobe can
sudo modprobe can_raw
sudo modprobe slcan

sudo slcand -o -c -s8 /dev/serial/by-id/usb-Klipper_stm32* /dev/slcan0
sudo ip link set up slcan0
sudo ip link set slcan0 name can0
```

> **Note:** `-s8` corresponds to 1,000,000 bps bitrate.

---

## Step 3: Make Interface Persistent

To persist the CAN setup at boot, enable the service:

```bash
sudo cp config/klipper_host_service/can0.service /etc/systemd/system/
sudo systemctl enable can0.service
sudo systemctl start can0.service
```

---

## Step 4: Test the CAN Bus

With all devices powered and connected on the CAN bus:

```bash
python3 ~/klipper/scripts/canbus_query.py can0
```

This should return a list of UUIDs for all connected CAN devices.

You may also run:

```bash
./scripts/detect-can-devices.sh
```

This will list all known devices and their detected UUIDs.

---

✅ Your CAN interface (`can0`) is now ready and can communicate with devices like the EBB42 and Cartographer probe.

