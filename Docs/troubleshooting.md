# Troubleshooting CANtainer Setup

This guide covers common issues encountered when setting up Klipper, CAN devices (Octopus Pro, EBB42, Cartographer), and running them under a virtualized Pimox environment.

---

## General Debugging Principles

* Double check CAN wiring and termination
* Confirm all boards are flashed properly (CANBoot, Klipper, Cartographer)
* Run `detect-can-devices.sh` to confirm connectivity
* Power cycle all CAN devices after flashing
* Review `dmesg`, `journalctl -u klipper` and `moonraker.log` for error traces

---

## Common Issues & Fixes

### ❌ CAN Device Not Found in `detect-can-devices.sh`

**Symptoms:**

* Your Cartographer, Octopus, or EBB42 isn't showing up

**Fixes:**

* Check power: ensure 24V is reaching devices
* Ensure proper 120Ω termination resistors at both ends
* Retry flashing with `flash-canboot.sh` followed by `flash-klipper.sh`
* Make sure your `can0` interface is up inside the Klipper VM:

  ```bash
  ip link set can0 up type can bitrate 1000000
  ip link show can0
  ```

### ❌ Octopus Not Flashing via CAN

**Symptoms:**

* Flash script hangs or times out when targeting the Octopus

**Fixes:**

* Double check that Octopus is connected via USB to VM and passed through
* Confirm it is flashed with CANBoot first
* Retry with slower baudrate or re-run `setup-can-interface.sh` in case can0 dropped

### ❌ Cartographer Doesn't Respond to Probe

**Symptoms:**

* Nothing happens when calling `cartographer_touch` macro

**Fixes:**

* Ensure firmware is Cartographer-specific, not Klipper
* Confirm CAN UUID maps correctly in `cartographer.cfg`
* Run diagnostics via `probe-output.txt` in `examples/`
* Make sure you have `[cartographer]` section defined in printer.cfg

### ❌ ADXL345 Not Detected

**Symptoms:**

* Cannot get input from accelerometer

**Fixes:**

* ADXL345 is pre-mounted — ensure Cartographer firmware supports ADXL
* Use `ls /dev` and check SPI address mapping
* Ensure `[resonance_tester]` section is configured

### ❌ `can0` Interface Missing After Reboot

**Symptoms:**

* `ip link show` does not list `can0`

**Fixes:**

* Re-run `setup-can-interface.sh` after reboot
* Automate interface setup in `/etc/network/interfaces.d/can0` or with systemd service
* Check passthrough in Proxmox VM hardware settings

---

## Advanced Debug Tools

### Logging CAN Traffic

Install `can-utils`:

```bash
sudo apt install can-utils
```

Monitor bus:

```bash
candump can0
```

### Rebooting Devices via `reset.py`

If devices become unresponsive:

```bash
python3 ~/klipper/scripts/reset.py
```

---

## Still Stuck?

* Use the Klipper and Cartographer GitHub Issues
* Post logs to forums or Discord
* Double-check UUIDs and wiring against official diagrams in `docs/diagrams/`

> Tip: Keep a printed CAN UUID chart taped to your printer.

---

Next: [diagrams/](./diagrams/) for wiring and network flow

