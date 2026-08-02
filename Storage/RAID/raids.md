# Linux Software RAID Levels

## RAID 0 - Striping

### Concept

RAID 0 distributes data blocks across multiple disks.

```text
Disk 1          Disk 2

Block 1         Block 2
Block 3         Block 4
```

### Advantages

- High read and write performance
- Uses the full capacity of all disks

### Disadvantages

- No redundancy
- No fault tolerance
- Failure of one disk causes loss of the entire array

### Production Usage

Use RAID 0 only when performance is more important than availability, such as:

- Temporary data
- Cache
- Scratch storage
- Rebuildable data

### Create RAID 0

```bash
mdadm --create /dev/md1   --level=0   --raid-devices=2   /dev/sdc /dev/sdd
```

---

## RAID 1 - Mirroring

### Concept

RAID 1 stores an identical copy of data on another disk.

```text
Disk 1          Disk 2

Data A    ==    Data A
Data B    ==    Data B
```

### Protection

RAID 1 can survive one disk failure in a two-disk mirror.

### Production Usage

Common uses:

- Operating-system disks
- Boot volumes
- Critical configuration data
- Small servers requiring simple redundancy

### Create RAID 1

```bash
mdadm --create /dev/md0   --level=1   --raid-devices=2   /dev/sda /dev/sdb
```

### Check Status

```bash
cat /proc/mdstat
mdadm --detail /dev/md0
```

Healthy example:

```text
[UU]
```

### Fail a Disk

```bash
mdadm --manage /dev/md0 --fail /dev/sda
```

### Remove the Failed Disk

```bash
mdadm --manage /dev/md0 --remove /dev/sda
```

### Add a Replacement

```bash
mdadm --manage /dev/md0 --add /dev/sda
```

### Monitor Rebuild

```bash
watch cat /proc/mdstat
```

---

## RAID 5 - Distributed Parity

### Concept

RAID 5 combines:

- Striping
- Distributed parity

Parity is distributed across all member disks. There is no single dedicated parity disk.

```text
Disk 1       Disk 2       Disk 3

Data A       Data B       Parity

Data C       Parity       Data D

Parity       Data E       Data F
```

### Protection

RAID 5 can survive one disk failure.

### Capacity

Usable capacity:

```text
(Number of Disks - 1) × Smallest Disk Size
```

### Production Usage

Common uses:

- File servers
- NAS
- Shared storage
- Archive storage

### Create RAID 5

```bash
mdadm --create /dev/md5   --level=5   --raid-devices=3   /dev/sdc /dev/sdd /dev/sde
```

### Failure and Recovery

Fail a disk:

```bash
mdadm --manage /dev/md5 --fail /dev/sdc
```

Remove it:

```bash
mdadm --manage /dev/md5 --remove /dev/sdc
```

Add a replacement:

```bash
mdadm --manage /dev/md5 --add /dev/sdc
```

Monitor rebuild:

```bash
watch cat /proc/mdstat
```

---

## RAID 6 - Double Parity

### Concept

RAID 6 uses two independent parity calculations.

```text
Disk 1     Disk 2     Disk 3     Disk 4

Data A     Data B     P1         P2

Data C     P1         P2         Data D
```

### Protection

RAID 6 can survive two disk failures.

### Capacity

Usable capacity:

```text
(Number of Disks - 2) × Smallest Disk Size
```

### Production Usage

Common uses:

- Large NAS systems
- Enterprise storage
- High-capacity storage
- Storage where availability is more important than write speed

### Create RAID 6

```bash
mdadm --create /dev/md6   --level=6   --raid-devices=4   /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

---

## RAID 10 - Mirror + Stripe

### Concept

RAID 10 combines:

```text
RAID 1 + RAID 0
```

It provides:

- High performance through striping
- Redundancy through mirroring
- Fast rebuilds compared with parity RAID in many workloads

### Four-Disk Example

```text
Mirror Set A:

sda <-> sdc

Mirror Set B:

sdb <-> sdd
```

The mirrored sets are then striped.

### Data Distribution

```text
Block 1 -> sda
           |
           +-> sdc (mirror copy)

Block 2 -> sdb
           |
           +-> sdd (mirror copy)
```

### Linux mdadm Layout

Example:

```text
Layout : near=2

set-A:
sda <-> sdc

set-B:
sdb <-> sdd
```

### Production Usage

Recommended for:

- Databases
- KVM virtual-machine storage
- VMware storage
- High-performance enterprise applications

### Create RAID 10

```bash
mdadm --create /dev/md10   --level=10   --raid-devices=4   /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

### Check the Layout

```bash
mdadm --detail /dev/md10
```

### Failure Test

Fail one member:

```bash
mdadm --manage /dev/md10 --fail /dev/sda
```

Check the array:

```bash
cat /proc/mdstat
mdadm --detail /dev/md10
```

Verify that data is still available:

```bash
ls /work
```

### Important RAID 10 Failure Note

RAID 10 can survive multiple disk failures only when the failed disks are not both members of the same mirror set.

For example:

```text
sda + sdc failed
```

may cause data loss because they belong to the same mirror set.

---

# RAID Status and Recovery

## Check RAID Status

```bash
cat /proc/mdstat
```

Examples:

```text
[UU]
```

All members are healthy.

```text
[U_]
```

One member is missing or failed.

```text
[UUU]
```

All three RAID members are healthy.

## Show Detailed Information

```bash
mdadm --detail /dev/mdX
```

## Fail a Disk

```bash
mdadm --manage /dev/mdX --fail /dev/sdX
```

`--fail` marks a RAID member as faulty while it is still associated with the array.

## Remove a Disk

```bash
mdadm --manage /dev/mdX --remove /dev/sdX
```

`--remove` removes a member from the RAID array. Normally, a disk is failed first and then removed.

## Add a Replacement

```bash
mdadm --manage /dev/mdX --add /dev/sdX
```

## Monitor Rebuild

```bash
watch cat /proc/mdstat
```

## Save RAID Configuration

```bash
mdadm --detail --scan > /etc/mdadm.conf
```

Verify:

```bash
cat /etc/mdadm.conf
```

> The configuration path can differ by distribution. Some systems use `/etc/mdadm/mdadm.conf`.
## RAID Levels Comparison

| RAID Level  | Relationship / Design         | Minimum Disks | Usable Capacity       | Fault Tolerance              | Security Level | Performance                | Common Production Use                           |
| ----------- | ----------------------------- | ------------: | --------------------- | ---------------------------- | -------------- | -------------------------- | ----------------------------------------------- |
| **RAID 0**  | Striping only                 |             2 | `N × Disk Size`       | No disk failure              | **None**       | Very High Read/Write       | Cache, temporary data, scratch storage          |
| **RAID 1**  | Mirroring                     |             2 | `(N ÷ 2) × Disk Size` | One disk per mirror can fail | **High**       | High Read / Good Write     | OS disks, boot volumes, critical data           |
| **RAID 5**  | Striping + Distributed Parity |             3 | `(N - 1) × Disk Size` | One disk can fail            | **Medium**     | High Read / Moderate Write | File servers, NAS, shared storage               |
| **RAID 6**  | RAID 5 + Double Parity        |             4 | `(N - 2) × Disk Size` | Two disks can fail           | **Very High**  | High Read / Lower Write    | Large NAS, enterprise storage, archive systems  |
| **RAID 10** | RAID 1 + RAID 0               |             4 | `(N ÷ 2) × Disk Size` | Depends on which disks fail  | **Very High**  | Very High Read/Write       | Databases, KVM, VMware, enterprise applications |

> `N` = Number of disks in the RAID array.

## Relationship Between RAID Levels

```text
RAID 0
│
└── Striping only
    └── Performance without redundancy


RAID 1
│
└── Mirroring only
    └── Redundancy without striping


RAID 5
│
├── Striping
└── Single Distributed Parity
    └── Performance + protection against one disk failure


RAID 6
│
├── Striping
└── Double Distributed Parity
    └── Protection against two disk failures


RAID 10
│
├── RAID 1 → Mirroring
└── RAID 0 → Striping
    └── High performance + high redundancy
```

## RAID Security and Fault-Tolerance Summary

| RAID        | Can the Array Continue After Disk Failure? |       Maximum Disk Failures | Data Protection           |
| ----------- | ------------------------------------------ | --------------------------: | ------------------------- |
| **RAID 0**  | ❌ No                                       |                           0 | No protection             |
| **RAID 1**  | ✅ Yes                                      | 1 disk in a two-disk mirror | Mirrored copy             |
| **RAID 5**  | ✅ Yes                                      |                           1 | Single distributed parity |
| **RAID 6**  | ✅ Yes                                      |                           2 | Double distributed parity |
| **RAID 10** | ✅ Yes                                      | Depends on the failed disks | Mirrored copies           |

## Important RAID 10 Note

RAID 10 does not simply mean that **any two disks** can fail.

For example:

```text
Mirror Set A:

sda <-> sdc

Mirror Set B:

sdb <-> sdd
```

If:

```text
sda + sdb fail
```

The array may continue because each mirror set still has one working disk.

However, if:

```text
sda + sdc fail
```

The array may fail because both disks belong to the same mirror set.

## Quick Selection Guide

| Requirement                                          | Recommended RAID |
| ---------------------------------------------------- | ---------------- |
| Maximum performance with no required protection      | RAID 0           |
| Simple redundancy for OS disks                       | RAID 1           |
| Good capacity with protection from one disk failure  | RAID 5           |
| High capacity with protection from two disk failures | RAID 6           |
| High performance and high availability               | RAID 10          |

> **Important:** RAID provides availability and protection against disk failure. RAID does not replace backups.
