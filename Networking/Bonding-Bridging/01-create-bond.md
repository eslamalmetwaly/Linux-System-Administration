# 01 - Create Linux Bond

## Objective

Create a Bond interface named `bond0`.

The Bond will use:

```text
Mode: active-backup
MII Monitoring: 100 ms
```

---

## Create the Bond

```bash
nmcli connection add type bond \
con-name bond0 \
ifname bond0 \
bond.options "mode=active-backup,miimon=100"
```

---

## Verify

```bash
nmcli connection show
```

```bash
ip link show bond0
```

---

## Check Bond Configuration

```bash
nmcli connection show bond0
```

Expected:

```text
bond.options: mode=active-backup,miimon=100
```

---

## Explanation

### mode=active-backup

Only one slave is active.

Example:

```text
ens161 = ACTIVE
ens224 = BACKUP
```

If `ens161` fails:

```text
ens224 → ACTIVE
```

---

### miimon=100

The Bond checks the link status every:

```text
100 milliseconds
```

If the active link fails, the Bond detects the failure and performs failover.

---

## Result

```text
bond0
```

has been created and is ready to receive physical interfaces.

# 02 - Add Physical Interfaces to Bond

## Objective

Add physical interfaces to `bond0`.

The interfaces are:

```text
ens161
ens224
```

---

## Add First Interface

```bash
nmcli connection add type ethernet \
con-name bond0-slave1 \
ifname ens161 \
master bond0
```

---

## Add Second Interface

```bash
nmcli connection add type ethernet \
con-name bond0-slave2 \
ifname ens224 \
master bond0
```

---

## Verify

```bash
nmcli connection show
```

Expected:

```text
bond0
bond0-slave1
bond0-slave2
```

---

## Check Bond Status

```bash
cat /proc/net/bonding/bond0
```

Expected:

```text
Bonding Mode: fault-tolerance (active-backup)
Currently Active Slave: ens161
```

---

## Final Bond Topology

```text
ens161 ───┐
          ├── bond0
ens224 ───┘
```
# 03 - Validate Bond Configuration

## Check Bond Status

```bash
cat /proc/net/bonding/bond0
```

Important values:

```text
Bonding Mode: active-backup
Currently Active Slave: ens161
MII Status: up
MII Polling Interval: 100
```

---

## Check Interfaces

```bash
ip link show
```

The physical interfaces should be members of:

```text
bond0
```

---

## Check NetworkManager

```bash
nmcli device status
```

Expected:

```text
ens161    connected
ens224    connected
bond0     connected
```

---

## Expected Architecture

```text
ens161 ───┐
          ├── bond0
ens224 ───┘
```

The Bond is now ready to be used as a Bridge Port.
