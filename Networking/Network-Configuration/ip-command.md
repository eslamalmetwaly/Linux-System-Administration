# Linux `ip` Command

The `ip` command is used to manage and troubleshoot network interfaces, IP addresses, and routing tables.

---

# 1. Display Network Interfaces

Show all interfaces:

```bash
ip link show
```

Short form:

```bash
ip l
```

Show IP addresses:

```bash
ip addr show
```

Short form:

```bash
ip a
```

Show a specific interface:

```bash
ip addr show ens160
```

---

# 2. Enable and Disable a Network Interface

Bring an interface UP:

```bash
ip link set ens160 up
```

Bring an interface DOWN:

```bash
ip link set ens160 down
```

Check the status:

```bash
ip link show ens160
```

---

# 3. Add an IP Address

Add an IP address temporarily:

```bash
ip addr add 192.168.43.50/24 dev ens160
```

Verify:

```bash
ip addr show ens160
```

> This configuration is temporary and will be lost after reboot or NetworkManager reconnection.

---

# 4. Delete an IP Address

Delete a specific IP address:

```bash
ip addr del 192.168.43.50/24 dev ens160
```

---

# 5. Remove All IP Addresses

```bash
ip addr flush dev ens160
```

> ⚠️ Be careful. This removes all IP addresses from the interface.

---

# 6. Display the Routing Table

```bash
ip route
```

Short form:

```bash
ip r
```

Example:

```text
default via 192.168.43.1 dev ens160
192.168.43.0/24 dev ens160 src 192.168.43.43
```

---

# 7. Add a Default Gateway

Add a default route:

```bash
ip route add default via 192.168.43.1 dev ens160
```

---

# 8. Delete a Default Gateway

```bash
ip route del default via 192.168.43.1 dev ens160
```

---

# 9. Add a Static Route

Example:

```bash
ip route add 10.10.10.0/24 via 192.168.43.1 dev ens160
```

This means:

```text
To reach 10.10.10.0/24
Use Gateway 192.168.43.1
Through Interface ens160
```

Verify:

```bash
ip route
```

---

# 10. Delete a Static Route

```bash
ip route del 10.10.10.0/24 via 192.168.43.1
```

---

# 11. Display the ARP / Neighbor Table

```bash
ip neigh
```

Example:

```text
192.168.43.1 dev ens160 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

The neighbor table shows the relationship between:

```text
IP Address → MAC Address
```

---

# 12. Delete an ARP Entry

```bash
ip neigh del 192.168.43.1 dev ens160
```

---

# 13. Troubleshooting with the `ip` Command

## Problem: Interface is DOWN

Check:

```bash
ip link show ens160
```

If the interface is DOWN:

```bash
ip link set ens160 up
```

Verify:

```bash
ip link show ens160
```

---

## Problem: No IP Address

Check:

```bash
ip addr show ens160
```

If no `inet` address exists, the interface has no IPv4 address.

Check NetworkManager:

```bash
nmcli device status
```

Activate the connection:

```bash
nmcli connection up home
```

---

## Problem: No Default Gateway

Check:

```bash
ip route
```

If no line starts with:

```text
default via
```

there is no default gateway.

Temporary fix:

```bash
ip route add default via 192.168.43.1 dev ens160
```

Permanent fix using NetworkManager:

```bash
nmcli connection modify home ipv4.gateway 192.168.43.1
nmcli connection up home
```

---

## Problem: Duplicate IP Address

Check:

```bash
ip addr show ens160
```

If multiple unexpected IP addresses exist:

```bash
ip addr del <IP>/<PREFIX> dev ens160
```

Example:

```bash
ip addr del 192.168.1.6/24 dev ens160
```

---

# 14. Temporary vs Permanent Configuration

## Temporary Configuration

Using `ip`:

```bash
ip addr add 192.168.43.50/24 dev ens160
```

The change may disappear after:

* Reboot
* NetworkManager restart
* Connection restart

## Permanent Configuration

Using `nmcli`:

```bash
nmcli connection modify home ipv4.addresses 192.168.43.50/24
nmcli connection up home
```

---

# Troubleshooting Workflow

When a Linux system has a network problem:

## Step 1: Check the Interface

```bash
ip link
```

## Step 2: Check the IP Address

```bash
ip addr
```

## Step 3: Check the Routing Table

```bash
ip route
```

## Step 4: Check the Neighbor Table

```bash
ip neigh
```

## Step 5: Test the Gateway

```bash
ping -c 4 192.168.43.1
```

## Step 6: Test Internet by IP

```bash
ping -c 4 8.8.8.8
```

## Step 7: Test DNS

```bash
ping -c 4 google.com
```

---

# Important Difference

| Tool    | Purpose                                 |
| ------- | --------------------------------------- |
| `ip`    | Temporary runtime network configuration |
| `nmcli` | Persistent NetworkManager configuration |

---

# Quick Reference

```bash
ip a
ip link
ip route
ip neigh

ip link set ens160 up
ip link set ens160 down

ip addr add IP/PREFIX dev ens160
ip addr del IP/PREFIX dev ens160

ip route add default via GATEWAY dev ens160
ip route del default via GATEWAY dev ens160

ip route add NETWORK/PREFIX via GATEWAY dev ens160
ip route del NETWORK/PREFIX via GATEWAY
```
