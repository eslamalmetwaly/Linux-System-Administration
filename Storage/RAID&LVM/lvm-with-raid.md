# LVM with RAID - Production Storage Lab

## Overview

This lab combines Linux Software RAID with LVM.

The design uses:

- Four physical disks
- RAID 10
- LVM
- XFS filesystem
- Persistent mount at `/app`

## Architecture

```text
/dev/sda  /dev/sdb  /dev/sdc  /dev/sdd
                    |
                    v
             RAID 10 (/dev/md0)
                    |
                    v
          LVM Physical Volume
                    |
                    v
           Volume Group (vg_app)
                    |
                    v
          Logical Volume (lv_data)
                    |
                    v
             XFS Filesystem
                    |
                    v
                  /app
```

## Why Combine RAID and LVM?

| Layer | Purpose |
|---|---|
| RAID | Disk redundancy, fault tolerance, and performance |
| LVM | Flexible volume management and storage allocation |
| XFS | Filesystem for application data |

## Lab Objectives

- Create RAID 10
- Create an LVM PV on the RAID device
- Create a VG and LV
- Create an XFS filesystem
- Mount the filesystem at `/app`
- Configure persistent mounting
- Test data availability after a disk failure
- Replace a failed RAID member

## Environment

| Component | Value |
|---|---|
| Operating System | Red Hat Enterprise Linux |
| RAID Tool | mdadm |
| RAID Level | RAID 10 |
| RAID Device | /dev/md0 |
| Volume Group | vg_app |
| Logical Volume | lv_data |
| Filesystem | XFS |
| Mount Point | /app |

> **Warning:** The following commands destroy data on the selected disks. Use only empty lab disks.

---

# 1. Verify the Disks

This lab uses:

```text
/dev/sda
/dev/sdb
/dev/sdc
/dev/sdd
```

Check devices:

```bash
lsblk
```

Check filesystems:

```bash
lsblk -f
```

Make sure the disks are not mounted and are not used by another RAID or LVM configuration.

---

# 2. Install mdadm

```bash
dnf install -y mdadm
```

---

# 3. Create RAID 10

```bash
mdadm --create /dev/md0   --level=10   --raid-devices=4   /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

Check synchronization:

```bash
cat /proc/mdstat
```

A healthy array should show:

```text
[UUUU]
```

Check details:

```bash
mdadm --detail /dev/md0
```

---

# 4. Save RAID Configuration

```bash
mdadm --detail --scan > /etc/mdadm.conf
```

Verify:

```bash
cat /etc/mdadm.conf
```

---

# 5. Create the LVM Physical Volume

Create the PV on the RAID device:

```bash
pvcreate /dev/md0
```

Verify:

```bash
pvs
```

---

# 6. Create the Volume Group

```bash
vgcreate vg_app /dev/md0
```

Verify:

```bash
vgs
```

Detailed information:

```bash
vgdisplay vg_app
```

---

# 7. Create the Logical Volume

Create a 20 GiB LV:

```bash
lvcreate -L 20G -n lv_data vg_app
```

Verify:

```bash
lvs
```

Detailed information:

```bash
lvdisplay /dev/vg_app/lv_data
```

---

# 8. Create the XFS Filesystem

```bash
mkfs.xfs /dev/vg_app/lv_data
```

Verify:

```bash
lsblk -f
```

---

# 9. Mount the Filesystem

Create the mount point:

```bash
mkdir -p /app
```

Mount:

```bash
mount /dev/vg_app/lv_data /app
```

Verify:

```bash
findmnt /app
df -h /app
```

---

# 10. Configure Persistent Mounting

Get the UUID:

```bash
blkid /dev/vg_app/lv_data
```

Edit:

```bash
vim /etc/fstab
```

Add:

```text
UUID=<LV_UUID>  /app  xfs  defaults  0 0
```

Test without rebooting:

```bash
mount -a
```

Verify:

```bash
findmnt /app
```

---

# 11. Test Application Data

Create a test file:

```bash
echo "LVM on RAID 10 test" > /app/raid-lvm-test.txt
```

Verify:

```bash
cat /app/raid-lvm-test.txt
```

---

# 12. Simulate a Disk Failure

Fail one RAID member:

```bash
mdadm --manage /dev/md0 --fail /dev/sda
```

Check RAID status:

```bash
cat /proc/mdstat
```

Check details:

```bash
mdadm --detail /dev/md0
```

Verify that `/app` is still mounted:

```bash
findmnt /app
```

Verify the data:

```bash
cat /app/raid-lvm-test.txt
```

The data remains available because RAID 10 maintains mirrored copies.

---

# 13. Replace and Rebuild the Failed Disk

Remove the failed member:

```bash
mdadm --manage /dev/md0 --remove /dev/sda
```

After replacing or preparing the disk, add it:

```bash
mdadm --manage /dev/md0 --add /dev/sda
```

Monitor the rebuild:

```bash
watch cat /proc/mdstat
```

Check the final state:

```bash
mdadm --detail /dev/md0
```

The array should return to:

```text
[UUUU]
```

---

# Validation Checklist

```bash
# RAID
cat /proc/mdstat
mdadm --detail /dev/md0

# LVM
pvs
vgs
lvs

# Filesystem
lsblk -f
findmnt /app
df -h /app

# Application data
cat /app/raid-lvm-test.txt
```

---

# Production Use Cases

RAID 10 + LVM is commonly used for:

- Databases
- KVM virtual-machine storage
- Enterprise applications
- High-performance services

Benefits:

- High read performance
- High write performance
- Disk redundancy
- Flexible LVM storage management
- Fast rebuilds in many workloads

## Design Rule

Build the layers in this order:

```text
Physical Disks
      |
     RAID
      |
      LVM
      |
 Filesystem
      |
 Mount Point
      |
 Application
```

RAID provides the reliable storage layer, while LVM provides flexible storage management.
## RAID Advantages

RAID combines multiple physical disks into one logical storage device and can provide performance, redundancy, or both, depending on the RAID level.

| RAID Advantage                 | Description                                                                                           |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- |
| **Disk Fault Tolerance**       | Some RAID levels allow the system to continue operating after one or more disks fail.                 |
| **High Availability**          | Storage can remain available during a disk failure, reducing service downtime.                        |
| **Improved Read Performance**  | RAID levels that use striping can read data from multiple disks in parallel.                          |
| **Improved Write Performance** | RAID 0 and RAID 10 can provide high write performance.                                                |
| **Data Redundancy**            | RAID 1 and RAID 10 maintain mirrored copies of data.                                                  |
| **Parity Protection**          | RAID 5 and RAID 6 use distributed parity to recover data after disk failures.                         |
| **Single Logical Device**      | Multiple physical disks are presented to the operating system as one RAID device, such as `/dev/md0`. |
| **Flexible RAID Levels**       | Different RAID levels can be selected based on performance, capacity, and availability requirements.  |

> **Important:** RAID protects against disk failure. It does not protect against accidental deletion, filesystem corruption, ransomware, or site failure. RAID is not a replacement for backups.

---

## LVM Advantages

LVM provides a flexible storage-management layer above physical storage devices.

| LVM Advantage                   | Description                                                                                                  |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Flexible Storage Allocation** | Storage can be divided into multiple logical volumes based on application requirements.                      |
| **Easy Volume Expansion**       | Logical volumes can be extended when additional storage is required.                                         |
| **Volume Groups**               | Multiple physical volumes can be combined into one storage pool called a Volume Group (VG).                  |
| **Logical Volumes**             | Storage can be allocated as independent logical volumes for different applications.                          |
| **Online Expansion**            | Many logical volumes and filesystems can be expanded without unmounting them.                                |
| **Snapshots**                   | LVM snapshots can capture the state of a logical volume for backup or testing.                               |
| **Storage Abstraction**         | Applications use logical volumes without needing to know the underlying physical-disk layout.                |
| **Better Storage Management**   | Administrators can manage storage using PVs, VGs, and LVs instead of working directly with individual disks. |
| **Migration Support**           | Physical extents can be moved between physical volumes using LVM tools when required.                        |

> **Important:** LVM does not provide disk redundancy by itself. If the underlying storage fails and no RAID or other redundancy exists, LVM cannot recover the lost data.

---

## Why Use RAID and LVM Together?

RAID and LVM solve different storage problems:

```text
RAID
│
├── Protects against disk failure
├── Provides redundancy
└── Can improve storage performance

LVM
│
├── Provides flexible storage allocation
├── Makes logical volumes easy to manage
├── Supports volume expansion
└── Supports snapshots
```

When combined:

```text
Physical Disks
      |
      v
RAID 10
      |
      |  Redundancy + Performance
      v
LVM
      |
      |  Flexible Storage Management
      v
XFS Filesystem
      |
      v
Application Data
```

### Combined Benefits

| Feature                    | RAID                  | LVM                           | RAID + LVM |
| -------------------------- | --------------------- | ----------------------------- | ---------- |
| Disk-failure protection    | ✅                     | ❌                             | ✅          |
| Redundancy                 | ✅                     | ❌                             | ✅          |
| High storage performance   | ✅                     | ⚠️ Depends on storage         | ✅          |
| Flexible volume allocation | ❌                     | ✅                             | ✅          |
| Logical volume expansion   | ❌                     | ✅                             | ✅          |
| Snapshots                  | ❌                     | ✅                             | ✅          |
| Easy storage management    | Limited               | ✅                             | ✅          |
| Production suitability     | Depends on RAID level | Depends on underlying storage | **High**   |

## Production Example

```text
4 SSD Disks
      |
      v
RAID 10
      |
      v
Volume Group: vg_app
      |
      ├── lv_database → /var/lib/mysql
      │
      ├── lv_application → /app
      │
      └── lv_logs → /var/log/app
```

In this design:

* **RAID 10** protects the storage layer and provides high performance.
* **LVM** divides the RAID storage into separate logical volumes.
* Each application can have its own filesystem and storage allocation.
* Logical volumes can be expanded independently when more capacity is required.

## Key Design Rule

```text
Physical Disks
      ↓
RAID
      ↓
LVM
      ↓
Filesystem
      ↓
Mount Point
      ↓
Application
```

**RAID provides reliable storage. LVM provides flexible storage management. Together, they create a scalable and production-ready storage architecture.**
