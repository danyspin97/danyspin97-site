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

.purple[.footer[Danilo Spinella<br>Fosdem 26 ![:resize 40](https://images.crunchbase.com/image/upload/c_pad,h_256,w_256,f_auto,q_auto:eco,dpr_1/j5kbhcqwulktkxkqeyha)]]
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

## Snapshots

Changing the system (adding and removing a package / updating the system) **creates a new shapshot**

Every snapshot has a different boot entry: `rootflags=subvol=@/.snapshots/1/snapshot`

`transactional-update` handles btrfs snapshots and changes

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

# Boot Loader Specification (BLS)

---
class: center, middle

## Linux system

![:resize 800](/img/health-checker-fosdem-26/linux.drawio.svg)

---
class: middle, top

## Adding another system...

--

`/boot/efi` must be **unique** on each disk

--

Which bootloader will be the default? Does it read the other partition `/boot` folder?

---
class: middle, left

## Boot Loader Specification

Allow the boot loader menu entries to be **shared** between _multiple operating systems_

Standardize configuration between booatloader, firmware and system components

Supports two simple formats, found in `/loader/entries`

---
class: left, middle

## Boot Loader Specification type#1 entry

```ini
# /boot/efi/loader/entries/opensuse-microos-6.18.4-5-default-1.conf 
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

**Revert back to the previous version** of the OS or kernel when the system fails to boot

Relies on **Boot Loader Specification** and support from system components

---
class: left, middle

## Boot counting

Add a number of boot attempts on new boot entries

Entries with non-zero tries left are "_indeterminate_"

Entries without boot counting are considered "_good_"

Entries with zero tries left are "_bad_"

---
class: center, middle

![:resize 1000](/img/health-checker-fosdem-26/boot-counting.drawio.svg)

---
class: left, middle

## Installing new entries

The initial counter defined in `/etc/kernel/tries`

**kernel-install** add boot entries with the boot counter enabled

**sdbootutil** manages the boot counter when editing the boot entries

---
class: left, middle

## Bootloader support

Put the entries with zero left tries at the bottom

Adjust boot counter when an entry is booted

**systemd-boot** and **grub2**_*_ supports Automatic Boot Assessment, but support can be extender to any bootloader

---
class: left, middle

## grub2_*_

_BLS_ support added by different set of patches (one by _Fedora_ and one by _openSUSE_)

Upstream *grub2 2.14* added initial **incomplete BLS support**

_Boot counting_ (and partial BLI support) **implemented via downstream** patches in _openSUSE_*

.footer[*upstreamed soon]

---
class: left, middle

## boot.target

Services that check for a successful boot (like health-checker)
```systemd
RequiredBy=boot-complete.target
```

Services that run when the system boots successfully (like systemd-bless-boot)
```systemd
Requires=boot-complete.target
After=boot-complete.target
```

---
class: left, middle

## systemd-bless-boot

Remove the boot counter when the boot succeed

Get the status of the current boot entry

Change the status of a boot entry

---
class: left, middle

## Issues with Automatic Boot Assessment

No automatic reboot on failed boot

Default entry not updated

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

