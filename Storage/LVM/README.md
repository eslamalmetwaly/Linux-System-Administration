# LVM Management Lab

## Overview

This project demonstrates practical Linux storage administration using **Logical Volume Manager (LVM)**.

The lab covers the complete LVM lifecycle, including:

* Creating Physical Volumes (PV)
* Creating Volume Groups (VG)
* Creating Logical Volumes (LV)
* Creating XFS and ext4 filesystems
* Mounting Logical Volumes
* Configuring persistent mounts using `/etc/fstab`
* Extending Logical Volumes
* Reducing ext4 Logical Volumes
* Creating and managing LVM Snapshots
* Recovering data using Snapshots
* Troubleshooting common LVM issues

---

## Lab Architecture

```text
Disk
 │
 ▼
Partition
 │
 ▼
Physical Volume (PV)
 │
 ▼
Volume Group (VG)
 │
 ▼
Logical Volume (LV)
 │
 ▼
Filesystem
 │
 ▼
Mount Point
```

### Lab Configuration

```text
/dev/sdb1
    │
    ▼
PV
    │
    ▼
VG: vg_data
    │
    ├── LV: lv_app
    │      ├── Filesystem: XFS
    │      └── Mount Point: /work
    │
    └── LV: lv_test
           ├── Filesystem: ext4
           └── Mount Point: /test
```

---

## Lab Modules

| Module                                            | Description                                         |
| ------------------------------------------------- | --------------------------------------------------- |
| [01 - Create LVM](01-create-lvm.md)               | Create PV, VG, LV, and filesystem                   |
| [02 - Mount and fstab](02-mount-and-fstab.md)     | Mount LVM storage and configure persistent mounting |
| [03 - Extend LV](03-extend-lv.md)                 | Extend an LVM Logical Volume and filesystem         |
| [04 - LVM Snapshot](04-lvm-snapshot.md)           | Create, mount, test, restore, and remove snapshots  |
| [05 - Reduce LV](05-reduce-lv.md)                 | Safely reduce an ext4 Logical Volume                |
| [06 - Troubleshooting](06-lvm-troubleshooting.md) | Troubleshoot common LVM and mount issues            |

---

## Key Concepts

### Physical Volume (PV)

A Physical Volume is a disk or partition initialized for use by LVM.

Example:

```bash
pvcreate /dev/sdb1
```

### Volume Group (VG)

A Volume Group combines one or more Physical Volumes into a storage pool.

Example:

```bash
vgcreate vg_data /dev/sdb1
```

### Logical Volume (LV)

A Logical Volume is created from the available space inside a Volume Group.

Example:

```bash
lvcreate -L 5G -n lv_app vg_data
```

---

## Filesystem Support

| Filesystem | Online Extend | Reduce          |
| ---------- | ------------- | --------------- |
| XFS        | ✅ Supported   | ❌ Not supported |
| ext4       | ✅ Supported   | ✅ Supported     |

> **Important:** XFS filesystems cannot be reduced. Reducing an XFS Logical Volume without rebuilding the filesystem can cause data loss.

---

## Skills Demonstrated

* Linux Storage Administration
* LVM Management
* XFS Administration
* ext4 Administration
* Persistent Mount Configuration
* LVM Extension
* LVM Reduction
* LVM Snapshot Management
* Snapshot Recovery
* Linux Storage Troubleshooting

---

## Validation Commands

```bash
lsblk

pvs
vgs
lvs

blkid

findmnt

df -h
```

---




---

## Notes

* Always verify the correct disk and Logical Volume before running destructive commands.
* Do not run `mkfs` on an existing Logical Volume containing data.
* Do not run `mkfs` on an LVM Snapshot.
* Always unmount a filesystem before reducing it.
* Reduce the filesystem before reducing the Logical Volume.
* Use LVM Snapshots for short-term recovery, not as a replacement for backups.
* Monitor Snapshot usage using:

```bash
lvs
```

---


