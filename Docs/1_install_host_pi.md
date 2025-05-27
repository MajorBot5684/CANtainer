Setting up the Raspberry Pi 4B as a Pimox 8 Host

This guide walks you through configuring a Raspberry Pi 4B (8GB RAM) as a host for Pimox 8 using Raspberry Pi OS Lite 64-bit and Proxmox VE 8 for ARM64.

Requirements

Raspberry Pi 4B with 8GB RAM (or compatible Pi with sufficient RAM)

Raspberry Pi Imager

Raspberry Pi OS 64-bit Lite (Debian 12 Bookworm)

Reliable microSD or USB storage

Ethernet or Wi-Fi network

Flash Raspberry Pi OS Lite

Download tools:

Raspberry Pi Imager: https://www.raspberrypi.com/software/

Raspberry Pi OS Lite 64-bit image: Direct link

Use Raspberry Pi Imager:

Choose OS → Scroll to bottom → Use Custom → Select 2024-11-19-raspios-bookworm-arm64-lite.img.xz

Choose storage → Your USB/SD device

Click the gear icon:

Set hostname: cantainer

Enable SSH

Set username/password: cantainer / your-password

Click Write and flash the image.

Boot your Pi

Insert the storage into your Pi and power it on.

SSH into the Pi from your Windows machine:

ssh cantainer@<your-pi-ip>

Prepare the Pi

Set root password:

sudo passwd root
sudo su -

Install utilities:

apt install -y neofetch tmux chrony ifupdown2

Start tmux session (optional):

tmux new -s pimoxinstall

Update and upgrade:

apt update && apt upgrade -y
reboot

SSH back in and switch to root:

sudo su -

Install Proxmox VE 8 (ARM64)

Add Proxmox mirror key:

curl https://mirrors.apqa.cn/proxmox/debian/pveport.gpg -o /etc/apt/trusted.gpg.d/pveport.gpg

Add Proxmox repo:

echo "deb https://mirrors.apqa.cn/proxmox/debian/pve bookworm port" > /etc/apt/sources.list.d/pveport.list
apt update && apt full-upgrade

Update /etc/hosts:

nano /etc/hosts

Comment out 127.0.1.1 if it exists.

Ensure correct IP and hostname mapping for cantainer.

Install PVE packages:

apt install -y proxmox-ve postfix open-iscsi

When prompted by postfix, choose Local only and enter hostname cantainer

Reboot:

reboot

Post-install Configuration

Verify system is up to date:

apt update

Disable Proxmox subscription nag screen:

sed -Ezi.bak "s/(Ext.Msg.show\\(\\{\\s+title: gettext\\('No valid sub)/void\\(\\{ \\/\\/\\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

Increase swap size:

nano /etc/dphys-swapfile
# Set: CONF_SWAPSIZE=1024
reboot

Enable memory stats for containers:

nano /boot/firmware/cmdline.txt
# Append: cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1

Check system info:

neofetch

Verify timezone and chrony:

timedatectl
chronyc sources
# Chrony config: /etc/chrony/chrony.conf

Login to Proxmox Web UI

Go to https://<pi-ip>:8006

Login as root with your set password

Configure bridge, storage, and VM settings as needed.

Your Raspberry Pi is now a functional Proxmox VE 8 ARM64 host named cantainer. Next step: setup_vm_klipper.md
