---
title: 'Boot process for modern Linux'
date: "2024-09-13T10:30:00"
categories: ["talk"]
tags: ["linux", "boot"]
---
class: center, middle
name: start

# Boot process </br> for modern Linux

.header[![:resize 128](/img/boot_process/moca_camp.png)]

---
background-image: url(https://wallpapercave.com/wp/wp3965612.jpg)
---
class: left, middle

## Who am I?

Danilo Spinella </br>_Research Engineer Linux_ </br> @ SUSE ![:i](fab fa-suse)

---
class: left, middle

## Contents

- Boot Process

- Boot Loader Specification

- TPM 2.0

- Userspace reboot

---
class: left, middle

## Boot Process

---
class: left, middle

# Legacy Boot

- BIOS

- Master Boot Record (MBR)

- Bootloader

- Kernel

- Init

---
class: left, middle

BIOS reads MBR containing GRUB stage 1

Chainload stage 1.5 from the 30KiB following MBR

Load GRUB stage 2 from fs (e.g. /boot/grub)

Load kernel into memory

Execute /bin/init

---
class: center, middle

![:resize 120](/img/boot_process/uefi.svg) </br> UEFI is an open standard designed to overcame BIOS limitations

---
class: left, middle

# GUID Partition Table (GPT)

A standard for the layout of partition tables

Defined as part of UEFI

Wide adoption in ~2014

---
background-image: url(https://upload.wikimedia.org/wikipedia/commons/4/45/GNU_GRUB_components.svg)
background-color: #ffffff

---
class: left, middle

# UEFI

The firmware itself act as a boot manager

Load another boot manager or a boot loader

Adds SecureBoot

---
class: left, middle

# EFI System Partition (ESP)

```bash
/boot/efi/EFI/
├── Boot
│   ├── bootx64.efi
│   ├── LenovoBT.EFI
│   ├── License.txt
│   └── ReadMe.txt
├── Microsoft
│   ├── Boot
│   └── Recovery
└── opensuse
    ├── boot.csv
    ├── grub.cfg
    ├── grub.efi
    ├── grubx64.efi
    ├── MokManager.efi
    └── shim.efi
```

---
class: left, middle

What happens when we have multiple boot loaders?

---
class: middle, left

# Boot Loader Specification

Set of file formats and naming conventions

Allows sharing menu entries between boot loaders

---
class: left, middle

# Type #1 Entries

```ini
# /boot/loader/entries/6a9857a393724b7a981ebb5b8495b9ea-3.8.0-2.fc19.x86_64.conf
title        Fedora 19 (Rawhide)
sort-key     fedora
machine-id   6a9857a393724b7a981ebb5b8495b9ea
version      3.8.0-2.fc19.x86_64
options      root=UUID=6d3376e4-fc93-4509-95ec-a21d68011da2 quiet
architecture x64
linux        /6a9857a393724b7a981ebb5b8495b9ea/3.8.0-2.fc19.x86_64/linux
initrd       /6a9857a393724b7a981ebb5b8495b9ea/3.8.0-2.fc19.x86_64/initrd
```

---
class: left, middle

# Type #2 Entries

EFI Unified Kernel Images

Combines an EFI stub loader and a kernel image

---
class: left, middle

# BLS support

systemd-boot (since v250)

GRUB2 (with BLS patches)

---
class: left, middle

# Encryption

Encrypt the system excluding /boot

Initramfs ask for the passphrase and unlock the system

---
class: left, middle

# TPM 2.0

Able to hold and seal key

Compute the PCR values

Equipped by all modern hardware

---
class: left, middle

# Platform Configuration Register (PCR)

Indexed registers that contains hashes

Every index represent a different stage of the boot

Volatile, can only be read but can be set via API

---
class: left, middle

# Unlocking the disk via TPM

The simple explanation

---
class: left, middle

# Enroll the key to the TPM2

```bash
$ systemd-cryptenroll \
		--tpm2-device=auto \
		--tpm2-pcrlock=/var/lib/systemd/pcrlock.json \
		/dev/sda3
```

---
class: left, middle

# Create a TPM policy

```bash
$ systemd-pcrlock make-policy
```

---
class: left, middle

# Initramfs

Must contains the PCR signature

Make it available for systemd to read

---
class: left, middle

# Tooling

systemd

sdbootutil

disk-encryption-tool

dracut-pcr-signature

---
class: left, middle

# kexec

Skip boot loader and hardware initialization by booting into a new kernel

Available since a long time

---
class: left, middle

# systemd soft-reboot

Able to shutdown all userspace programs and restart systemd

Better compatibility with firmware / devices

Pin resources across reboot

Available since v254

---

class: left, middle

## Thank you for your attention

# Questions ![:i](fa fa-question)

---
template: start

