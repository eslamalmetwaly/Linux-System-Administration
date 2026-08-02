# Linux Software RAID Administration

## Overview

This section documents Linux Software RAID administration using `mdadm`.

Topics covered:

- RAID 0 - Striping
- RAID 1 - Mirroring
- RAID 5 - Distributed Parity
- RAID 6 - Double Parity
- RAID 10 - Mirror + Stripe
- Disk failure testing
- RAID recovery and rebuild
- Production use cases

## Environment

| Component | Value |
|---|---|
| Operating System | Red Hat Enterprise Linux |
| RAID Tool | mdadm |
| Filesystem | XFS |
| Storage | Virtual Disks |

## RAID Architecture

```text
Physical Disks
      |
      v
Software RAID
      |
      v
Filesystem
      |
      v
Application Data
```

## RAID and LVM Architecture

```text
Physical Disks
      |
      v
RAID
      |
      v
LVM
      |
      v
Filesystem
      |
      v
Applications
```

## Responsibilities

| Technology | Responsibility |
|---|---|
| RAID | Performance, redundancy, and disk-failure protection |
| LVM | Flexible storage management and logical volumes |
| Filesystem | Stores files and directories |

## Important Notes

RAID is **not** a backup solution.

RAID protects mainly against disk failure. A production environment should also include:

- Backups
- Monitoring
- Alerting
- Disaster Recovery

## Useful Commands

Check RAID status:

```bash
cat /proc/mdstat
```

Show RAID details:

```bash
mdadm --detail /dev/mdX
```

Show disk metadata:

```bash
mdadm --examine /dev/sdX
```

## RAID Production Summary

| RAID | Common Production Use |
|---|---|
| RAID 0 | Temporary high-performance workloads |
| RAID 1 | OS disks and critical small workloads |
| RAID 5 | File servers and NAS |
| RAID 6 | Large storage and high-capacity arrays |
| RAID 10 | Databases, KVM, VMware, and enterprise applications |
