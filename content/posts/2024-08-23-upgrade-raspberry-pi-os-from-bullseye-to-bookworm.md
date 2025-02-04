---

layout: post  
title: Upgrade Raspberry Pi OS from Bullseye to Bookworm  
date: 2024-08-23 10:26 -0600  
categories: \[Linux, RaspberryPi\]  
tags: \[Upgrade,linux,bookworm,bullseye\] # TAG names should always be lowercase

---

From: https://gist.github.com/jauderho/6b7d42030e264a135450ecc0ba521bd8

WARNING: READ CAREFULLY BEFORE ATTEMPTING

Officially, this is not recommended. YMMV (But tested and work)

https://www.raspberrypi.com/news/bookworm-the-new-version-of-raspberry-pi-os/

This mostly works if you are on 64bit. You are on your own if you are on 32bit or mixed 64/32bit

Credit to anfractuosity and fgimenezm for figuring out additional details for kernels

## Make sure everything is up-to-date

```
sudo apt-get update && sudo apt-get dist-upgrade
```

## Point to bookworm repos instead

```
sudo sed -i -e 's/bullseye/bookworm/g' /etc/apt/sources.list
sudo sed -i -e 's/bullseye/bookworm/g' /etc/apt/sources.list.d/raspi.list
```

## Contents of /etc/apt/sources.list

```
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
```

## Uncomment deb-src lines below then 'apt-get update' to enable 'apt-get source'

```
#deb-src http://deb.debian.org/debian bookworm main contrib non-free
#deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free
#deb-src http://deb.debian.org/debian bookworm-updates main contrib non-free
```

## Contents of /etc/apt/sources.list.d/raspi.list

```
deb http://archive.raspberrypi.org/debian/ bookworm main
```

## Uncomment line below then 'apt-get update' to enable 'apt-get source'

```
#deb-src http://archive.raspberrypi.org/debian/ bookworm main
```

## Do actual update

```
sudo apt update && sudo apt -y full-upgrade && sudo apt -y clean && sudo apt -y autoremove
```

## Reboot

```
sudo reboot
```

## Remove old config files after doing sanity checks

```
sudo apt purge ?config-files
```

Switch to the new kernels

WARNING: Since this has bitten several folks. The following can completely brick your system requiring a reinstall

DO NOT do this if you are unsure

## Prep

```
sudo dpkg --purge --force-depends raspberrypi-kernel raspberrypi-bootloader
sudo umount /boot
sudo fsck -y /boot
sudo mkdir /boot/firmware
sudo sed -i.bak -e "s#boot#boot/firmware#" /etc/fstab
sudo systemctl daemon-reload
sudo mount /boot/firmware
sudo apt install raspi-firmware
```

## Actually install the kernels. Make sure you pick the right version for your Pi

```ruby
sudo apt install linux-image-rpi-v8 linux-headers-rpi-v8 # 64bit
sudo apt install linux-image-rpi-v7l linux-headers-rpi-v7l # 32bit
sudo apt install linux-image-rpi-v6 linux-headers-rpi-v6
Append auto_initramfs=1 to the bottom of file where it says [all]
sudo sed -i.bak '$ a\auto_initramfs=1' /boot/firmware/config.txt
```

## Reboot

sudo reboot

```
Verify using "uname -a" (correct as of 10/2023)
```

# Old kernel

```
Linux raspberrypi 6.1.21-v8+ #1642 SMP PREEMPT Mon Apr 3 17:24:16 BST 2023 aarch64 GNU/Linux
```

# New kernel

```
Linux raspberrypi 6.1.0-rpi4-rpi-v8 #1 SMP PREEMPT Debian 1:6.1.54-1+rpt2 (2023-10-05) aarch64 GNU/Linux
```

If you are not converted to using NetworkManager, you might lose networking upon reboot.

Check the ARP table to see what the new DHCP assigned IP address is. You may have to manually set the IP address again

Thanks to solsticedhiver for identifying this

## Install NetworkManager if not already installed

```
sudo apt-get install --no-install-recommends network-manager
#
```

## Switch to NetworkManager from dhcpcd

```
sudo systemctl enable --now NetworkManager
sudo systemctl disable --now dhcpcd
#
```

## Set up static IP. Adjust as necessary

```
sudo nmcli -p connection show
sudo nmcli -p connection show "Wired connection 1"
sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.1.5/24 ipv4.gateway 192.168.1.1
```

## Reboot

```
sudo reboot
```

## Bonus steps

## Install btop

```
sudo apt-get install btop
```

## Update /etc/ssh/sshd\_config for up to date, secure by default config. Use ssh-audit to verify

```
KexAlgorithms sntrup761x25519-sha512@openssh.com,curve25519-sha256,curve25519-sha256@libssh.org
HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-ed25519
```

## Ciphers chacha20-poly1305@openssh.com # Disabled due to CVE-2023-48795 for now

```
Ciphers aes128-gcm@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```