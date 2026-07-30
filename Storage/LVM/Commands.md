# LVM Quick Commands

## Display Storage Information

```bash
lsblk
blkid
df -h
findmnt
```

## Physical Volumes

```bash
pvs
pvdisplay
pvcreate /dev/sdb1
```

## Volume Groups

```bash
vgs
vgdisplay
vgcreate vg_data /dev/sdb1
vgchange -ay vg_data
vgchange -an vg_data
```

## Logical Volumes

```bash
lvs
lvdisplay

lvcreate -L 5G -n lv_app vg_data

lvextend -L +2G -r /dev/vg_data/lv_app

lvremove /dev/vg_data/lv_app
```

## XFS

```bash
mkfs.xfs /dev/vg_data/lv_app

xfs_growfs /work
```

> XFS can be extended but cannot be reduced.

## ext4

```bash
mkfs.ext4 /dev/vg_data/lv_test

e2fsck -f /dev/vg_data/lv_test

resize2fs /dev/vg_data/lv_test 1G
```

## Mounting

```bash
mount /dev/vg_data/lv_app /work

umount /work

findmnt /work
```

## Snapshot

Create:

```bash
lvcreate -L 1G -s -n snap_test /dev/vg_data/lv_app
```

Mount an XFS Snapshot:

```bash
mount -o nouuid /dev/vg_data/snap_test /mnt/snap
```

Remove:

```bash
umount /mnt/snap

lvremove /dev/vg_data/snap_test
```

Merge:

```bash
lvconvert --merge /dev/vg_data/snap_test
```

## Safe ext4 Reduce

```bash
umount /test

e2fsck -f /dev/vg_data/lv_test

resize2fs /dev/vg_data/lv_test 1G

lvreduce -L 1G /dev/vg_data/lv_test

mount /dev/vg_data/lv_test /test
```
