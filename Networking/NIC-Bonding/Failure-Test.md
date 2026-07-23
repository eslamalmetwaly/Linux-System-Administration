# NIC Bonding Failure Test

## Objective

Simulate a physical NIC failure and verify that the Bond automatically switches to the backup interface.

---

## Initial State

```text
bond0
├── ens224 → ACTIVE
└── ens161 → BACKUP
```

Verify:

```bash
cat /proc/net/bonding/bond0
```

Check connectivity:

```bash
ping -c 10 192.168.43.1
```

---

## Simulate Active NIC Failure

Disable the active interface:

```bash
ip link set ens224 down
```

Or:

```bash
nmcli device disconnect ens224
```

---

## Verify Failover

```bash
cat /proc/net/bonding/bond0
```

Expected:

```text
Currently Active Slave: ens161
```

The bond should automatically switch traffic from:

```text
ens224 → ens161
```

---

## Test Connectivity After Failover

```bash
ping -c 10 192.168.43.1
```

The server should remain connected to the network.

---

## Restore the Failed Interface

```bash
ip link set ens224 up
```

Or:

```bash
nmcli device connect ens224
```

Verify:

```bash
cat /proc/net/bonding/bond0
```

---

## Failure Scenario

```text
Normal:

ens224 ─── ACTIVE ───┐
                     ├── bond0
ens161 ─── BACKUP ───┘

Failure:

ens224 ─── DOWN

ens161 ─── ACTIVE ─── bond0
```

---

## Result

The active network interface was disabled intentionally.

The Bond detected the failure and moved network traffic to the backup interface.

Network redundancy and failover were successfully verified.
