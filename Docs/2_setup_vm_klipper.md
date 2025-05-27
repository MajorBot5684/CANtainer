# 🖥️ Klipper VM Setup on Pimox 8 (Ubuntu 24.04 LTS)

This guide will walk you through setting up a Klipper virtual machine running on Pimox 8 (Proxmox for Raspberry Pi). The VM will run Klipper host services.

---

## ✅ Requirements

* Proxmox 8 running on a Raspberry Pi 4B (8GB recommended)
* Headless Ubuntu Server 24.04 LTS ISO
* Proxmox GUI access
* 2 CPU cores and 2048MB RAM allocated to the VM
* USB passthrough enabled to connect your Octopus board

---

## 📥 Upload Ubuntu ISO via Proxmox Web Interface

1. Open the Proxmox web interface in your browser.
2. In the left panel, select your node (e.g. `cantainer`).
3. Expand **`local (cantainer)`** or your designated ISO storage.
4. Click on the **`ISO Images`** section.
5. Hit **Upload**, then download and select the ISO file:

   * [Ubuntu Server 24.04 LTS (64-bit) ISO](https://ubuntu.com/download/server)
   * Filename will look like: `ubuntu-24.04-live-server-amd64.iso`
6. Wait for the upload to complete.

---

## 💻 Create the Klipper VM

1. Click **Create VM** in the top-right corner.
2. Set the following:

   * **VM ID**: Choose any ID (e.g. 100)
   * **Name**: `klipper`
3. Click **Next** and configure the following tabs:

### OS:

* **ISO image**: Select the Ubuntu 24.04 ISO you uploaded
* **Guest OS**: `Linux 5.x or higher kernel`

### System:

* BIOS: Default (OVMF UEFI is not required)
* SCSI Controller: `VirtIO SCSI`
* Check `Qemu Agent` (optional but helpful)

### Hard Disk:

* Bus/Device: `VirtIO Block`
* Disk size: `8GB` (or more if needed)

### CPU:

* Cores: `2`

### Memory:

* Allocate `2048MB` (2GB)

### Network:

* Model: `VirtIO (paravirtualized)`

4. Click **Finish** to create the VM.

---

## ⚙️ Enable USB Passthrough

To allow your Octopus board to act as a CAN adapter over USB, we need to pass through the USB device to the VM:

1. Identify the USB device from the Proxmox host:

```bash
lsusb
```

2. Note the **Vendor ID** and **Product ID** (e.g. `1a86:7523` for CH340 USB-Serial)
3. Edit the VM hardware and **Add > USB Device** with the correct IDs

You can also manually add to the VM’s config:

```bash
echo 'usb0: host=1a86:7523' >> /etc/pve/qemu-server/100.conf
```

Replace `100` with your VM ID.

---

## 🧠 First Boot and Setup

1. Start the VM
2. Open the console via Proxmox GUI
3. Follow Ubuntu installation prompts:

   * Minimal install
   * Configure your user (e.g. `cantainer`)
   * Enable OpenSSH server
4. Once finished, SSH into the VM from your local system:

```bash
ssh cantainer@<your-vm-ip>
```

---

## 🧰 Post-Install Steps

Inside the VM:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git python3 python3-pip
```

Install `tmux` and any other helpful tools:

```bash
sudo apt install -y tmux neofetch
```

You now have a clean Ubuntu VM ready to install Klipper and interact with CAN-connected devices via the Octopus mainboard passed through USB.

---

**Next step:** Set up the CAN interface inside the VM → see `docs/can_interface_setup.md`
