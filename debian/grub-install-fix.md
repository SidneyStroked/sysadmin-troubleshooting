# Debian Installation – GRUB Failure (Supermicro / Older Hardware)

On some older servers (e.g. Supermicro), GRUB installation may fail during Debian setup.

## Typical symptoms

- Installer completes successfully  
- GRUB is not installed correctly  
- System fails to boot after reboot  

---

## Important

If the installer prompts for a reboot:

**Do not reboot immediately. You still have a chance to fix the issue.**

---

## Manual GRUB Installation Before Reboot

Open a console (e.g. `Ctrl+Alt+F2`) and run:

### 1. Bind system mounts
```bash
mount --rbind /sys /target/sys
```
### 2. Chroot into the new system
```bash
chroot /target /bin/bash
```

### 3. Install GRUB on disks

Example for RAID1 / dual disk setup:

```bash
grub-install /dev/sda
grub-install /dev/sdb
```

### 4. Update GRUB configuration
```bash
update-grub
```

### 5. Exit chroot
```bash
exit
```

You can now safely reboot.

---

## What if manual installation also fails?

Try adding the following parameter:

```bash
grub-install /dev/sda --no-nvram
```

---

## What does `--no-nvram` do and why it helps

On UEFI systems, GRUB installation normally:

- writes a boot entry into NVRAM (UEFI firmware)  
- creates a boot option (e.g. "debian")  

### Problem

- Older BIOS/UEFI implementations (often on Supermicro boards) may have bugs  
- Writing to NVRAM may fail or be blocked  
- As a result, `grub-install` fails  

---

## `--no-nvram` solution

The parameter:

```bash
--no-nvram
```

tells GRUB:

**Do not write anything to UEFI NVRAM, just install the bootloader to disk.**

### Result

- GRUB is installed to the EFI partition / disk  
- Firmware may still find it via fallback path (`EFI/BOOT/BOOTX64.EFI`)  
- Or it can be chainloaded by another bootloader  

---

## When to use this

- Older servers (Supermicro, older Dell generations)  
- Unreliable or buggy UEFI behavior  
- Installer reports GRUB installation errors  
- Manual `grub-install` fails without this flag  

---

## Practical tip

If using RAID1:

**Always install GRUB on all disks**, otherwise the system may not boot if one disk fails.
