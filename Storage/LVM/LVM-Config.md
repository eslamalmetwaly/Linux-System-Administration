# LVM Management and Storage Administration Lab

## Overview

This lab demonstrates practical Linux storage administration using **LVM (Logical Volume Manager)**.

The lab covers:

* Physical Volumes (PV)
* Volume Groups (VG)
* Logical Volumes (LV)
* XFS and ext4 filesystems
* Mounting storage
* Persistent mounting using `/etc/fstab`
* Extending Logical Volumes
* Reducing Logical Volumes
* LVM Snapshots
* Snapshot recovery and rollback
* LVM troubleshooting

---

# Lab Architecture

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

Example used in this lab:

```text
/dev/sdb1
    │
    ▼
PV
    │
    ▼
VG: vg_data
    │
    ├── LV: lv_app  → XFS  → /work
    │
    └── LV: lv_test → ext4 → /test
```

---

# 1. Create an LVM Physical Volume

Check available disks:

```bash
lsblk
```

Create a Physical Volume:

```bash
pvcreate /dev/sdb1
```

Verify:

```bash
pvs
pvdisplay
```

---

# 2. Create a Volume Group

Create the Volume Group:

```bash
vgcreate vg_data /dev/sdb1
```

Verify:

```bash
vgs
vgdisplay vg_data
```

---

# 3. Create a Logical Volume

Create a 5 GB Logical Volume:

```bash
lvcreate -L 5G -n lv_app vg_data
```

Verify:

```bash
lvs
lvdisplay /dev/vg_data/lv_app
```

---

# 4. Create an XFS Filesystem

Create the filesystem:

```bash
mkfs.xfs /dev/vg_data/lv_app
```

Check filesystem information:

```bash
blkid /dev/vg_data/lv_app
```

---

# 5. Mount the Logical Volume

Create the mount point:

```bash
mkdir -p /work
```

Mount the LV:

```bash
mount /dev/vg_data/lv_app /work
```

Verify:

```bash
findmnt /work
df -h /work
```

---

# 6. Configure Persistent Mounting

Get the UUID:

```bash
blkid /dev/vg_data/lv_app
```

Add the UUID to `/etc/fstab`:

```fstab
UUID=<UUID> /work xfs defaults 0 0
```

Validate the configuration:

```bash
mount -a
```

Verify:

```bash
findmnt /work
```

---

# 7. Extend an LVM Logical Volume

Check free space inside the Volume Group:

```bash
vgs
```

Extend the LV by 2 GB and automatically resize the filesystem:

```bash
lvextend -L +2G -r /dev/vg_data/lv_app
```

Verify:

```bash
lvs
df -h /work
```

The `-r` option resizes the filesystem automatically.

---

# 8. LVM Snapshot

Create test data before creating the Snapshot:

```bash
echo "old data" > /work/file1.txt
```

Create a 1 GB Snapshot:

```bash
lvcreate -L 1G -s -n snap_test /dev/vg_data/lv_app
```

Verify:

```bash
lvs
```

Create a mount point:

```bash
mkdir -p /mnt/snap
```

Because the original filesystem is XFS, mount the Snapshot using `nouuid`:

```bash
mount -o nouuid /dev/vg_data/snap_test /mnt/snap
```

Verify:

```bash
ls -la /mnt/snap
```

---

## Snapshot Behavior

A Snapshot represents the state of the Logical Volume at the time it was created.

| Operation                    | Original LV | Snapshot      |
| ---------------------------- | ----------- | ------------- |
| File created before Snapshot | Available   | Available     |
| File created after Snapshot  | Available   | Not available |
| File deleted after Snapshot  | Deleted     | Available     |
| Existing file modified       | New version | Old version   |

Example:

```bash
touch /work/new_after_snapshot.txt
```

The file exists in:

```bash
ls /work
```

But does not exist in:

```bash
ls /mnt/snap
```

---

# 9. Remove an LVM Snapshot

Unmount the Snapshot:

```bash
umount /mnt/snap
```

Remove the Snapshot:

```bash
lvremove /dev/vg_data/snap_test
```

Verify:

```bash
lvs
```

---

# 10. Snapshot Rollback

Merge the Snapshot with the original LV:

```bash
lvconvert --merge /dev/vg_data/snap_test
```

The Snapshot is removed after a successful merge.

If the original LV is mounted, the merge may be completed after reboot.

For a non-root filesystem, the LV can be unmounted before merging:

```bash
umount /work

lvconvert --merge /dev/vg_data/snap_test

mount /work
```

---

# 11. LVM Reduce Using ext4

> Important: XFS filesystems cannot be reduced.

Create a test LV:

```bash
lvcreate -L 2G -n lv_test vg_data
```

Create an ext4 filesystem:

```bash
mkfs.ext4 /dev/vg_data/lv_test
```

Create a mount point:

```bash
mkdir -p /test
```

Mount the LV:

```bash
mount /dev/vg_data/lv_test /test
```

Create test data:

```bash
echo "LVM Reduce Lab" > /test/info.txt
```

---

## Safe LVM Reduce Procedure

Unmount the filesystem:

```bash
umount /test
```

Check the filesystem:

```bash
e2fsck -f /dev/vg_data/lv_test
```

Reduce the filesystem:

```bash
resize2fs /dev/vg_data/lv_test 1G
```

Reduce the Logical Volume:

```bash
lvreduce -L 1G /dev/vg_data/lv_test
```

Mount the filesystem again:

```bash
mount /dev/vg_data/lv_test /test
```

Verify:

```bash
df -h /test

ls -l /test

cat /test/info.txt
```

---

# Filesystem Support

| Filesystem | Extend | Reduce |
| ---------- | -----: | -----: |
| XFS        |    Yes |     No |
| ext4       |    Yes |    Yes |

---

# LVM Troubleshooting

## Problem: Mount Point Is Not Mounted

Check:

```bash
findmnt /work
```

Check the LV:

```bash
lvs
```

Mount manually:

```bash
mount /dev/vg_data/lv_app /work
```

---

## Problem: Volume Group Cannot Be Deactivated

Error:

```text
Logical volume contains a filesystem in use.
Can't deactivate volume group.
```

Check mounted filesystems:

```bash
findmnt | grep vg_data
```

Unmount the filesystems:

```bash
umount /work
umount /test
```

Deactivate the VG:

```bash
vgchange -an vg_data
```

Activate it again:

```bash
vgchange -ay vg_data
```

---

## Problem: Files Appear After Unmount

When a filesystem is mounted on a directory, the filesystem hides the original contents of that directory.

Example:

```bash
umount /work

ls -la /work
```

Files displayed after unmount may belong to the root filesystem and not to the Logical Volume.

Mount the LV again:

```bash
mount /dev/vg_data/lv_app /work
```

---

# Important Production Notes

* Always verify the correct device before running destructive commands.
* Do not run `mkfs` on an existing LVM Snapshot.
* Do not reduce an LV before reducing the filesystem.
* XFS cannot be reduced.
* Use Snapshots for short-term recovery, not as permanent backups.
* Monitor Snapshot usage with:

```bash
lvs
```

* Stop applications before performing storage maintenance.
* Unmount filesystems before deactivating a Volume Group.
* Always maintain a tested backup before performing destructive storage operations.

---

# Skills Demonstrated

* Linux Storage Administration
* LVM Management
* Filesystem Administration
* XFS Management
* ext4 Management
* Storage Expansion
* Storage Reduction
* Snapshot Management
* Snapshot Recovery
* Linux Troubleshooting

---

# Conclusion

This lab demonstrates the complete LVM lifecycle:

```text
Disk
 ↓
Partition
 ↓
PV
 ↓
VG
 ↓
LV
 ↓
Filesystem
 ↓
Mount
 ↓
Extend
 ↓
Snapshot
 ↓
Recovery
 ↓
Troubleshooting
```
