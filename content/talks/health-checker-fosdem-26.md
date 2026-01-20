---
title: 'Error recovery at boot with MicroOS and systemd-bless-boot'
date: "2026-01-01T15:00:00"
categories: ["workshop"]
tags: ["health-checker", "fosdem"]
---
class: center, middle
name: start

### Error recovery
## with MicroOS and systemd-bless-boot

.purple[.footer[![:i](fas fa-gear) Fosdem 26]]
---
background-image: url(https://i.pinimg.com/736x/e4/bb/f6/e4bbf6d987196ca49097a7604d734d89.jpg)
---
class: left, middle

## Who am I

### Danilo Spinella</br>_Linux Research Engineer_ @ .green[SUSE ![:i](fab fa-suse)]

---
class: left, middle

# Contents

health-checker

Boot Loader Specification

Automatic Boot Assessment

---
class: left, middle

# health-checker

---
class: left, middle

## openSUSE MicroOS

Immutable system (by using **btrfs snapshots** and **transactional-update**)

Applications installed in containers

Reliability first by leveraging snapshots

---
class: left, middle

## health-checker

Check the status of the system during the boot process

**Plug-in based**, each plug-in checks a different system component

**Fail-safe mechanism**, reboot on error to a working snapshot

---
class: left, middle

## health-checker fail-safe

If a boot entry is new, **automatically reboot** up to 3 times when the boot fails

**Update** the default entry to a working snapshot

If the entry was working before: reboot one time, if it still fails, **open an emergency shell**

---
class: left, middle

## Projects involved

`systemd`

`grub2`

`sdbootutil`

`transactional-update`

---
class: left, middle

# Boot Loader Specification (BLS)

---
class: center, middle

## Linux system

![:resize 800](/img/health-checker-fosdem-26/linux.drawio.svg)

---

class: left, top

## Adding another system...

--

`/boot/efi` must be **unique** on each disk

Which bootloader will be the default? Does it read the other partition `/boot` folder?

---
class: middle, left

## Boot Loader Specification

Allow the boot loader menu entries to be **shared** between _multiple operating systems_

```ini
# /boot/efi/loader/entries/opensuse-microos-6.18.4-5-default-1.conf 
# Boot Loader Specification type#1 entry
title      openSUSE MicroOS 1
version    1@6.18.4-5-default
sort-key   opensuse-microos
options    quiet rootflags=subvol=@/.snapshots/1/snapshot systemd.show_status=yes console=ttyS0,115200 console=tty0 ignition.platform.id=qemu security=selinux selinux=1 root=UUID=8f71b02c-9905-b5e6-fafe-9efdb7c0ecd0
linux      /opensuse-microos/6.18.4-5-default/linux-03046d37a644bdf18e0d0dbe4677405926c6df24
initrd     /opensuse-microos/6.18.4-5-default/initrd-95afcf129ca424d5481026705a360d3bc92d5d75
```

---
class: middle, left

# Automatic Boot Assessment

---
class: middle, left

## Automatic Boot Assessment

Revert back to the previous version of the OS or kernel when the system fails to boot

---
class: center, middle

## Boot Counting

![:resize 1000](/img/health-checker-fosdem-26/boot-counting.drawio.svg)


---
class: left, top

## How does it work?

**kernel-install** and **sdbootutil** will add boot entries with the initial counter defined in `/etc/kernel/tries`


**systemd-boot** and **grub2**_*_ sort the entries with non-zero tries left first and decrease the counter on boot

Services (like **health-checker**) check that the system booted successfully

**systemd-bless-boot** marks the boot entry as "good" (removes the boot counter)

---
class: left, middle

## Bootloader entries' sorting

---
class: left, middle

## boot.target

Services that check for a successful boot
```systemd
RequiredBy=boot-complete.target
```

Services that run when the system boots successfully
```systemd
Requires=boot-complete.target
After=boot-complete.target
```

---
class: left, middle

## grub2_*_

_BLS_ support added by different set of patches (one by _Fedora_ and one by _openSUSE_)

Upstream *grub2 2.14* added initial **incomplete BLS support**

_Boot counting_ (and partial BLI support) implemented via downstream patches in _openSUSE_

---
class: middle, left

## Boot entries and snapshots

```bash
# /boot/efi/loader/entries/opensuse-microos-6.18.4-5-default-1.conf
title      openSUSE MicroOS 1
version    1@6.18.4-5-default
options    quiet rootflags=subvol=@/.snapshots/1/snapshot systemd.show_status=yes console=ttyS0,115200 console=tty0 ignition.platform.id=qemu security=selinux selinux=1 root=UUID=8f71b02c-9905-b5e6-fafe-9efdb7c0ecd0
```

---
class: left, middle

## Issues with Automatic Boot Assessment

No automatic reboot on failed boot

Default entry (`LoaderEntryDefault`) not updated

Simple logic and control by design (to be implemented by bootloaders)

---
class: left, middle

## health-checker fail-safe (in detail)

Checks the status of the boot entry by using `systemd-bless-boot`

**Update** or **remove** the current default entry to use the bootloader's sorting

If the entry has boot counting enabled, **reboot automatically** (can be disabled via kernel cmdline)

If the entry was working before: reboot one time, if it still fails, **opens an emergency shell**

---
class: left, middle

## Thank you for your attention

# Questions ![:i](fa fa-question)

---
template: start

