# NIC Bonding — High Availability Lab

## Overview

This lab demonstrates how to configure NIC Bonding on a Linux server using NetworkManager.

NIC Bonding combines multiple physical network interfaces into one logical interface to provide:

* Network redundancy
* High availability
* Failover
* Increased resilience against NIC or link failure

## Lab Architecture

```text
                    bond0
                      │
          ┌───────────┴───────────┐
          │                       │
       ens224                  ens161
       ACTIVE                  BACKUP
          │                       │
          └────────── Network ────┘
```

## Environment

* OS: RHEL / CentOS Stream
* NetworkManager: Enabled
* Bond Interface: `bond0`
* Bond Mode: `active-backup`
* Primary Interface: `ens224`
* Backup Interface: `ens161`

## Existing Network Interface

```text
ens160 → Existing network connection
```

The existing interface was not modified during this lab.

## Objective

Configure a redundant network connection and verify automatic failover when the active NIC becomes unavailable.

## Key Technologies

* NetworkManager
* nmcli
* Linux NIC Bonding
* Active-Backup
* Network Failover
* Network Troubleshooting

## Result

The server maintained network connectivity when the active physical interface was disabled, automatically switching traffic to the backup interface.
