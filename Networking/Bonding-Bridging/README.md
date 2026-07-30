# Linux Bonding + Bridge + KVM Networking Lab

## Overview

This project demonstrates a production-style Linux networking architecture that combines:

* Linux Bonding for NIC redundancy
* Linux Bridge for Layer 2 connectivity
* KVM/libvirt virtual machine networking
* Active/Backup failover testing
* Network troubleshooting and root cause analysis

---

## Final Architecture

```text
                         External Network
                                │
                                │
                         ┌──────┴──────┐
                         │     br0     │
                         │ Linux Bridge│
                         └──────┬──────┘
                                │
                              bond0
                         Active / Backup
                         ┌──────┴──────┐
                         │             │
                      ens161        ens224
                       ACTIVE        BACKUP
                               
                                │
                         ┌──────┴──────┐
                         │   KVM VMs   │
                         │   VM1 / VM2  │
                         └─────────────┘
```

---

## Lab Flow

```text
Phase 1
   │
   ▼
Linux Bonding
   │
   ▼
bond0
   │
   ▼
Linux Bridge
   │
   ▼
br0
   │
   ▼
KVM Virtual Machines
   │
   ▼
Failure Testing
   │
   ▼
Troubleshooting
```

---

## Environment

| Component       | Value                    |
| --------------- | ------------------------ |
| OS              | Red Hat Enterprise Linux |
| Network Manager | NetworkManager           |
| Bond            | bond0                    |
| Bond Mode       | active-backup            |
| Bond Members    | ens161, ens224           |
| Bridge          | br0                      |
| Virtualization  | KVM/libvirt              |

---

## Network Design

### Physical Interfaces

```text
ens161
ens224
```

### Bonding Layer

```text
ens161 ───┐
          ├── bond0
ens224 ───┘
```

### Bridge Layer

```text
bond0 ─── br0
```

### Virtualization Layer

```text
VM ─── br0
```

---

## Important Rule

The IP address is configured on the Bridge:

```text
IP Address → br0
```

The following interfaces do not have the Layer 3 IP configuration:

```text
ens161
ens224
bond0
```

---

## Main Objectives

* Build a redundant network path
* Use Bonding for NIC failover
* Use the Bond interface as a Bridge Port
* Connect KVM VMs to the external network
* Test real network failure scenarios
* Troubleshoot using Linux networking tools

---

## Validation Commands

```bash
nmcli device status
```

```bash
nmcli connection show
```

```bash
ip addr
```

```bash
ip route
```

```bash
bridge link
```

```bash
cat /proc/net/bonding/bond0
```

```bash
virsh domiflist <vm-name>
```

---

## Result

This lab demonstrates a complete Linux virtualization networking architecture:

```text
Physical NICs
      ↓
  Bonding
      ↓
   bond0
      ↓
   Bridge
      ↓
    br0
      ↓
   KVM VMs
```

The architecture provides network redundancy and Layer 2 connectivity for virtual machines.
