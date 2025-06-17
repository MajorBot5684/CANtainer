# Flashing CANboot and Klipper Firmware

This guide walks through flashing all boards using CANboot and Klipper. You will flash:

* **Octopus Pro 1.1** (as USB-to-CAN adapter and MCU)
* **EBB42** toolhead board
* **Cartographer** probe (with Cartographer software)

You will flash everything from inside the Klipper VM.

---

## Prerequisites

* `can0` interface is configured and working in the VM
* All devices are connected to the CAN bus
* Octopus is connected to the VM via USB
* All devices have been flashed previously with CANboot (see Cartographer docs)

---

## Step 1: Install Dependencies

SSH into your Klipper VM and run:

```bash
sudo apt install -y git python3 python3-pip can-utils
pip3 install pyserial intelhex
```

Clone the flashing tools:

```bash
git clone https://github.com/Arksine/CanBoot.git
cd CanBoot/scripts
```

---

## Step 2: Identify Devices

List all CANboot devices:

```bash
python3 flash_can.py -i can0 -l
```

Note: UUIDs will vary each time. You can identify devices based on connection order or board behavior (e.g., unplug Cartographer and recheck list).

---

## Step 3: Flash Octopus and EBB42 with Klipper

Get the appropriate Klipper `.bin` files from your Klipper repo `out/` folder or build them with `make menuconfig`.

### Flash Octopus:

```bash
python3 flash_can.py -i can0 -u <octopus_uuid> -f ~/klipper/out/klipper.bin
```

### Flash EBB42:

```bash
python3 flash_can.py -i can0 -u <ebb42_uuid> -f ~/klipper/out/klipper.bin
```

Replace `<octopus_uuid>` and `<ebb42_uuid>` with values shown in the `-l` listing.

---

## Step 4: Flash Cartographer Probe

Download the precompiled firmware or follow Cartographer’s official guide:

* Guide: [https://docs.cartographer3d.com/cartographer-probe/firmware/firmware-switching/usb-to-canbus](https://docs.cartographer3d.com/cartographer-probe/firmware/firmware-switching/usb-to-canbus)
* Repo: [https://github.com/Cartographer3D](https://github.com/Cartographer3D)

Then flash the firmware:

```bash
python3 flash_can.py -i can0 -u <cartographer_uuid> -f path/to/cartographer-canbus.bin
```

---

## Step 5: Verify Flash

List devices again:

```bash
python3 flash_can.py -i can0 -l
```

Each should report a valid bootloader and firmware type. You can also verify by running Klipper and checking connection in your web UI.

---

## Bitrate Note

All devices should use `1000000` as the CAN bitrate. Ensure this matches your `can0` setup.

---

Continue to [cartographer\_setup.md](./cartographer_setup.md) to configure the Cartographer probe.
