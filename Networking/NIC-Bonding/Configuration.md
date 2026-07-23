# NIC Bonding Configuration

## 1. Verify Available Interfaces

```bash
nmcli device status
```

Available interfaces:

```text
ens160
ens161
ens224
ens256
```

The following interfaces were selected for bonding:

```text
ens224 → Primary
ens161 → Backup
```

`ens160` was left unchanged because it was already in use.

---

## 2. Create the Bond Interface

```bash
nmcli connection add type bond \
con-name bond0 \
ifname bond0 \
bond.options "mode=active-backup,miimon=100"   miimon=100 → Check the physical link every 100ms and trigger failover if the active NIC link fails.
```

## Bond Configuration

```text
Bond Interface: bond0
Mode:           active-backup
Link Monitoring: miimon=100
```

---

## 3. Configure the Bond IP Address

```bash
nmcli connection modify bond0 \
ipv4.method manual \
ipv4.addresses 192.168.43.50/24 \
ipv4.gateway 192.168.43.1 \
ipv4.dns "8.8.8.8 8.8.4.4"
```

---

## 4. Create Bond Slave Profiles

```bash
nmcli connection add type bond-slave \
con-name bond0-slave1 \
ifname ens224 \
master bond0
```

```bash
nmcli connection add type bond-slave \
con-name bond0-slave2 \
ifname ens161 \
master bond0
```

---

## 5. Activate the Bond

```bash
nmcli connection up bond0
```

```bash
nmcli connection up bond0-slave1
```

```bash
nmcli connection up bond0-slave2
```

---

## 6. Verify the Configuration

```bash
nmcli connection show
```

Expected:

```text
bond0          bond      bond0
bond0-slave1   ethernet  ens224
bond0-slave2   ethernet  ens161
```

Check the IP address:

```bash
ip addr show bond0
```

Check the routing table:

```bash
ip route
```

Check Bond status:

```bash
cat /proc/net/bonding/bond0
```

Expected:

```text
Bonding Mode: fault-tolerance (active-backup)
Currently Active Slave: ens224
Slave Interface: ens224
Slave Interface: ens161
```
