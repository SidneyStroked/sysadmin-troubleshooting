# Windows Server 2016 Migration Issue (vSphere -> Proxmox)

## Problem Description

During migration of a Windows Server 2016 VM from VMware vSphere to Proxmox VE, the system failed to boot after disk migration when using a VirtIO disk.

The VM ended with a BSOD during startup.

---

## Migration Steps Performed

Before migration, the following preparation was done:

- Installed VirtIO drivers inside Windows Server 2016
- Uninstalled VMware Tools
- Created a new VM in Proxmox VE
- Configured VirtIO SCSI controller
- Attached `virtio-win` ISO
- Booted both VMs (source VMware VM and target Proxmox VM) into Clonezilla
- Cloned the system disk over network from VMware to Proxmox

---

## Issue

After the disk clone was completed, the migrated VM would not boot when the system disk was attached as a VirtIO device (`virtio0`).

Even though VirtIO drivers were already installed before migration, Windows did not properly use them during boot.

---

## Workaround / Solution

The following workaround resolved the issue:

1. Attach the migrated system disk temporarily as a SATA disk
2. Boot Windows Server successfully
3. Add a small temporary disk (for example 1 GB) using VirtIO bus
4. Windows detects the new VirtIO storage device and initializes the VirtIO drivers
5. Shut down the VM
6. Change the system disk bus from SATA back to VirtIO
7. Boot the VM again

After these steps, Windows boots correctly using VirtIO.

---

## Possible Cause

It appears that Windows Server 2016 may have the VirtIO drivers installed, but the boot-critical VirtIO storage driver is not properly initialized for the boot device after migration.

Adding an additional VirtIO disk while booted from SATA forces Windows to detect and activate the VirtIO storage stack, after which booting directly from VirtIO works normally.
