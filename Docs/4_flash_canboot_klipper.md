# Setting up the Klipper VM on Pimox 8

This guide explains how to create and configure a Ubuntu 24.04 Server LTS VM to run Klipper on your Pimox 8 host (`cantainer`).

---

## Prerequisites

- Pimox 8 installed and running on Raspberry Pi host (`cantainer`)
- Access to Proxmox web GUI at `https://<pi-ip>:8006`
- Ubuntu 24.04 Server LTS (64-bit ARM) ISO downloaded and uploaded to Proxmox ISO storage

---

## Step 1: Upload Ubuntu ISO to Proxmox

1. Log into the Proxmox web GUI.
2. Select the `cantainer` node.
3. Go to **Storage** → your ISO storage.
4. Click **Upload**, select the Ubuntu 24.04 Server ARM64 ISO, and upload.

---

## Step 2: Create the Klipper VM

1. Click **Create VM** on the top right.
2. Enter VM ID and Name (e.g., `100` and `klipper`).
3. Under **OS**:
   - Select the uploaded Ubuntu 24.04 ISO.
4. Under **System**:
   - BIOS: Default (SeaBIOS)
   - Machine: Default
5. Under **Hard Disk**:
   - Storage: Choose your preferred storage (e.g., local-lvm)
   - Disk size: 20 GB (adjust as needed)
   - Format: QCOW2
6. Under **CPU**:
   - Cores: 2
7. Under **Memory**:
   - RAM: 2048 MB
8. Under **Network**:
   - Model: VirtIO (paravirtualized)
9. Click **Finish** to create the VM.

---

## Step 3: Configure USB Passthrough for Octopus Board

Your Octopus board will be connected via USB to the Raspberry Pi host and passed through to the VM:

1. SSH into your Pimox host (`cantainer`).
2. Identify the Octopus USB device with:

   ```bash
   lsusb
