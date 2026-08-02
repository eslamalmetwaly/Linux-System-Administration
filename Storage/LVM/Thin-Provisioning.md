# LVM Thin Provisioning Lab

## Overview

This lab demonstrates how to configure **LVM Thin Provisioning** on Linux and simulate a real-world virtualization environment.

The scenario represents a **KVM Virtualization Host** that provides virtual disks to multiple virtual machines while consuming physical storage only when data is written.

---

## Lab Scenario

A company has a KVM virtualization server with limited physical storage.

The company needs to deploy three virtual machines:

| Virtual Machine    | Virtual Disk Size |
| ------------------ | ----------------: |
| Web Server         |             20 GB |
| Application Server |             20 GB |
| Database Server    |             20 GB |

Total virtual storage assigned:

```text
20 GB + 20 GB + 20 GB = 60 GB
```

However, the physical storage available is less than 20 GB.

LVM Thin Provisioning allows the administrator to assign **60 GB of logical storage** without reserving all 60 GB immediately.

Physical storage is consumed only when data is written.

---

## Storage Architecture

```text
Physical Disk
     │
     ▼
Physical Volume (PV)
     │
     ▼
Volume Group (VG)
vg_kvm
     │
     ▼
Thin Pool
thinpool
     │
     ├── vm_web  → 20 GB
     ├── vm_app  → 20 GB
     └── vm_db   → 20 GB
```

---

## Requirements

* Linux system with LVM installed
* One unused disk
* Root privileges
* The disk used in this lab must not contain important data

> Warning: The following commands can erase data from the selected disk.

---

# Part 1: Prepare the Storage

## 1. Verify Available Disks

```bash
lsblk
```

Example:

```text
NAME   SIZE TYPE
sda     40G disk
├─sda1   1G part
└─sda2  39G part

sdb     20G disk
```

In this lab:

```text
/dev/sdb
```

is used as the LVM disk.

---

## 2. Remove Existing Signatures

```bash
wipefs -a /dev/sdb
```

---

## 3. Create a Physical Volume

```bash
pvcreate /dev/sdb
```

Verify:

```bash
pvs
```

---

## 4. Create a Volume Group

```bash
vgcreate vg_kvm /dev/sdb
```

Verify:

```bash
vgs
```

---

# Part 2: Create the Thin Pool

Create a Thin Pool:

```bash
lvcreate -L 15G -T vg_kvm/thinpool
```

Verify:

```bash
lvs
```

Expected output:

```text
LV       VG      Attr       LSize
thinpool vg_kvm  twi-a-tz-- 15.00g
```

The `t` in the attributes indicates a Thin Pool.

---

# Part 3: Create Thin Volumes

Create a 20 GB virtual disk for the Web Server:

```bash
lvcreate -V 20G -T vg_kvm/thinpool -n vm_web
```

Create a 20 GB virtual disk for the Application Server:

```bash
lvcreate -V 20G -T vg_kvm/thinpool -n vm_app
```

Create a 20 GB virtual disk for the Database Server:

```bash
lvcreate -V 20G -T vg_kvm/thinpool -n vm_db
```

Verify:

```bash
lvs -o lv_name,lv_size,pool_lv,data_percent
```

---

## Over-Provisioning

The total virtual storage assigned is:

```text
vm_web = 20 GB
vm_app = 20 GB
vm_db  = 20 GB

Total = 60 GB
```

The Thin Pool is only:

```text
15 GB
```

This is possible because Thin Volumes do not reserve all physical storage when they are created.

---

# Part 4: Create File Systems

For this lab, each Thin Volume is used as a simulated VM disk.

Create XFS file systems:

```bash
mkfs.xfs /dev/vg_kvm/vm_web
mkfs.xfs /dev/vg_kvm/vm_app
mkfs.xfs /dev/vg_kvm/vm_db
```

Create mount points:

```bash
mkdir /vm_web
mkdir /vm_app
mkdir /vm_db
```

Mount the Thin Volumes:

```bash
mount /dev/vg_kvm/vm_web /vm_web
mount /dev/vg_kvm/vm_app /vm_app
mount /dev/vg_kvm/vm_db /vm_db
```

Verify:

```bash
df -h
```

Each simulated VM sees a 20 GB disk.

---

# Part 5: Simulate Real Data Usage

Write 5 GB to the Web Server volume:

```bash
dd if=/dev/zero of=/vm_web/web-data.img bs=1M count=5120 status=progress
```

Write 4 GB to the Application Server volume:

```bash
dd if=/dev/zero of=/vm_app/app-data.img bs=1M count=4096 status=progress
```

Write 3 GB to the Database Server volume:

```bash
dd if=/dev/zero of=/vm_db/db-data.img bs=1M count=3072 status=progress
```

Total data written:

```text
5 GB + 4 GB + 3 GB = 12 GB
```

---

## Monitor Thin Pool Usage

Run:

```bash
lvs -o lv_name,lv_size,data_percent
```

Example:

```text
LV       LSize   Data%
thinpool 15.00g  67.53
vm_app   20.00g  20.21
vm_db    20.00g  20.21
vm_web   20.00g  10.23
```

The Thin Pool is shared by all Thin Volumes.

The `Data%` value shows how much physical storage is currently consumed.

---

# Part 6: Extend the Thin Pool

During the lab, the Thin Pool reached approximately:

```text
67.53%
```

The Thin Pool was extended using all remaining free space in the Volume Group:

```bash
lvextend -l +100%FREE /dev/vg_kvm/thinpool
```

Verify:

```bash
lvs -o lv_name,lv_size,data_percent
```

Also verify the remaining free space:

```bash
vgs
```

---

## Why Did the First Extension Fail?

The following command was attempted:

```bash
lvextend -L +4.96G /dev/vg_kvm/thinpool
```

The output showed:

```text
Insufficient free space:
1270 extents needed,
but only 1269 available
```

LVM allocates storage using **Physical Extents (PEs)**.

The requested size required one more Physical Extent than was available.

The solution was:

```bash
lvextend -l +100%FREE /dev/vg_kvm/thinpool
```

This command uses all available Physical Extents without manually calculating the exact size.

---

# Important Production Warning

The total Thin Volume size is:

```text
60 GB
```

The physical storage is less than:

```text
20 GB
```

This is called:

```text
Over-Provisioning
```

or:

```text
Overcommitment
```

This is useful but requires monitoring.

If the Thin Pool becomes full, applications and virtual machines may experience:

* I/O errors
* File system errors
* VM failures
* Data corruption risks

---

# Recommended Monitoring Commands

Check Thin Pool usage:

```bash
lvs -o lv_name,lv_size,data_percent
```

Check Volume Group free space:

```bash
vgs
```

Display detailed Thin Pool information:

```bash
lvs -a
```

Check mounted file systems:

```bash
df -h
```

---

# Auto-Extension Configuration

Edit:

```bash
vi /etc/lvm/lvm.conf
```

Configure:

```text
thin_pool_autoextend_threshold = 80
thin_pool_autoextend_percent = 20
```

Meaning:

```text
When the Thin Pool reaches 80% usage,
LVM attempts to extend it by 20%.
```

> Auto-extension requires free space inside the Volume Group.

---

# Key Concepts

| Term              | Description                                          |
| ----------------- | ---------------------------------------------------- |
| PV                | Physical storage initialized for LVM                 |
| VG                | Storage pool created from one or more PVs            |
| Thin Pool         | Shared physical storage used by Thin Volumes         |
| Thin LV           | Virtual logical volume that consumes space on demand |
| Virtual Size      | Storage size visible to the VM                       |
| Physical Usage    | Actual storage consumed by written data              |
| Over-Provisioning | Assigning more virtual storage than physical storage |
| Data%             | Current physical usage of the Thin Pool              |

---

# Final Result

```text
Physical Storage
       │
       ▼
VG: vg_kvm
       │
       ▼
Thin Pool: thinpool
       │
       ├── vm_web → 20 GB
       ├── vm_app → 20 GB
       └── vm_db  → 20 GB

Total Virtual Storage = 60 GB
Physical Storage = Less than 20 GB
```

---

# Conclusion

LVM Thin Provisioning allows administrators to allocate more logical storage than the currently available physical storage.

Storage is consumed dynamically when data is written.

This technology is commonly used in:

* KVM Virtualization
* Private Cloud
* Virtual Machine Storage
* Enterprise Storage Systems
* Linux Infrastructure

Thin Provisioning improves storage utilization but requires:

* Monitoring
* Capacity planning
* Free space for extension
* Automatic alerts
* Thin Pool protection
