# NIC Bonding Troubleshooting

## 1. Check Bond Status

```bash
cat /proc/net/bonding/bond0
```

Check:

```text
Bonding Mode
Currently Active Slave
MII Status
Slave Interfaces
```

---

## 2. Check NetworkManager

```bash
systemctl status NetworkManager
```

If stopped:

```bash
systemctl start NetworkManager
```

---

## 3. Check Devices

```bash
nmcli device status
```

Expected:

```text
bond0    connected
ens224   connected
ens161   connected
```

---

## 4. Check Connection Profiles

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

## 5. Problem: Slave Profile Uses Wrong Interface

Example error:

```text
No suitable device found
mismatching interface name
```

Check the profile:

```bash
nmcli connection show bond0-slave1
```

Check:

```bash
connection.interface-name
```

The profile must match the real interface name:

```text
ens224
```

Not:

```text
224
```

---

## 6. Problem: Slave Does Not Activate

Check:

```bash
nmcli device status
```

Check the physical interface:

```bash
ip link show ens224
```

Bring it up:

```bash
ip link set ens224 up
```

Then:

```bash
nmcli connection up bond0-slave1
```

---

## 7. Problem: Bond Has No IP Address

Check:

```bash
ip addr show bond0
```

Configure:

```bash
nmcli connection modify bond0 \
ipv4.method manual \
ipv4.addresses 192.168.43.50/24 \
ipv4.gateway 192.168.43.1
```

Reactivate:

```bash
nmcli connection down bond0
nmcli connection up bond0
```

---

## 8. Problem: No Network Connectivity

Check:

```bash
ip route
```

Test the Gateway:

```bash
ping -c 4 192.168.43.1
```

Test external connectivity:

```bash
ping -c 4 8.8.8.8
```

Test DNS:

```bash
ping -c 4 google.com
```

---

## 9. Troubleshooting Workflow

```text
1. Check bond status
        ↓
2. Check active slave
        ↓
3. Check slave MII status
        ↓
4. Check NetworkManager
        ↓
5. Check connection profiles
        ↓
6. Check IP address
        ↓
7. Check routing table
        ↓
8. Test gateway
        ↓
9. Test external connectivity
```

---

## Important Commands

```bash
cat /proc/net/bonding/bond0
nmcli device status
nmcli connection show
nmcli connection show bond0
ip addr
ip route
ip link
ping
systemctl status NetworkManager
```
