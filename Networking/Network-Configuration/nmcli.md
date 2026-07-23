# Network Configuration with nmcli

## 1. NetworkManager

NetworkManager is the service responsible for managing network configuration on Linux.

Check its status:

```bash
systemctl status NetworkManager
```

Start the service:

```bash
systemctl start NetworkManager
```

Enable it at boot:

```bash
systemctl enable NetworkManager
```

Restart the service:

```bash
systemctl restart NetworkManager
```

---
#  Add a New Network Profile

To create a new NetworkManager connection profile:

```bash
nmcli connection add type ethernet \
con-name lab-network \
ifname ens160 \
ipv4.method manual \
ipv4.addresses 192.168.43.50/24 \
ipv4.gateway 192.168.43.1 \
ipv4.dns "8.8.8.8 8.8.4.4"
```

This command creates a new profile with:

```text
Connection Name: lab-network
Interface:       ens160
IP Address:      192.168.43.50/24
Gateway:         192.168.43.1
DNS:             8.8.8.8, 8.8.4.4
```

## Verify the New Profile

```bash
nmcli connection show
```

## Activate the New Profile

```bash
nmcli connection up lab-network
```

## Verify the Configuration

```bash
ip addr show ens160
ip route
```

## Delete a Network Profile

```bash
nmcli connection delete lab-network
```

> Note: `nmcli connection add` creates a new NetworkManager profile, while `nmcli connection modify` changes an existing profile.


# 2. Network Connections

## Show all connections

```bash
nmcli connection show
```

Example:

```text
NAME   UUID                                  TYPE      DEVICE
home   2d3f8f23-bdfd-4720-919d-8bad6dc57cc1  ethernet  ens160
```

## Show a specific connection

```bash
nmcli connection show home
```

## Show active connections

```bash
nmcli connection show --active
```

---

# 3. Network Devices

## Show device status

```bash
nmcli device status
```

## Show detailed device information

```bash
nmcli device show ens160
```

## Show IP information

```bash
nmcli device show ens160 | grep IP4
```

---

# 4. Configure a Static IP Address

Current configuration:

```text
Connection: home
Interface:  ens160
IP Address: 192.168.43.43/24
Gateway:    192.168.43.1
DNS:        8.8.8.8, 8.8.5.3
```

Configure a static IP:

```bash
nmcli connection modify home ipv4.method manual
nmcli connection modify home ipv4.addresses 192.168.43.43/24
nmcli connection modify home ipv4.gateway 192.168.43.1
nmcli connection modify home ipv4.dns "8.8.8.8 8.8.5.3"
```

Apply the configuration:

```bash
nmcli connection down home
nmcli connection up home
```

---

# 5. Verify the Configuration

Check the IP address:

```bash
ip addr show ens160
```

Check the routing table:

```bash
ip route
```

Check the DNS:

```bash
nmcli device show ens160 | grep DNS
```

Check the configuration:

```bash
nmcli connection show home | grep -E 'ipv4.method|ipv4.addresses|ipv4.gateway|ipv4.dns'
```

---

# 6. Network Connectivity Testing

## Test the Default Gateway

```bash
ping -c 4 192.168.43.1
```

## Test Internet Connectivity by IP

```bash
ping -c 4 8.8.8.8
```

## Test DNS Resolution

```bash
ping -c 4 google.com
```

---

# 7. Change the IP Address

```bash
nmcli connection modify home ipv4.addresses 192.168.43.50/24
```

Apply:

```bash
nmcli connection up home
```

Verify:

```bash
ip addr show ens160
```

---

# 8. Change the Default Gateway

```bash
nmcli connection modify home ipv4.gateway 192.168.43.1
```

Verify:

```bash
ip route
```

Expected:

```text
default via 192.168.43.1 dev ens160
```

---

# 9. Change DNS

```bash
nmcli connection modify home ipv4.dns "8.8.8.8 8.8.4.4"
```

Apply:

```bash
nmcli connection up home
```

Verify:

```bash
nmcli device show ens160 | grep DNS
```

---

# 10. Disable Automatic DNS

```bash
nmcli connection modify home ipv4.ignore-auto-dns yes
```

---

# 11. Activate and Deactivate a Connection

Deactivate:

```bash
nmcli connection down home
```

Activate:

```bash
nmcli connection up home
```

---

# 12. Troubleshooting

## Problem: No IP Address

Check:

```bash
ip addr
```

Check:

```bash
nmcli device status
```

Solution:

```bash
nmcli connection up home
```

---

## Problem: No Internet Access

Check the routing table:

```bash
ip route
```

Test the Gateway:

```bash
ping -c 4 192.168.43.1
```

Test Internet by IP:

```bash
ping -c 4 8.8.8.8
```

If the Gateway works but 8.8.8.8 fails, check:

* Default Gateway
* Network connection
* Router
* Firewall

---

## Problem: DNS Resolution Failure

Test:

```bash
ping -c 4 8.8.8.8
```

If it works, test:

```bash
ping -c 4 google.com
```

If the IP works but the hostname fails, the problem is probably DNS.

Check:

```bash
cat /etc/resolv.conf
```

or:

```bash
nmcli device show ens160 | grep DNS
```

Fix:

```bash
nmcli connection modify home ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection up home
```

---

## Problem: NetworkManager Is Not Running

Check:

```bash
systemctl status NetworkManager
```

Start:

```bash
systemctl start NetworkManager
```

Enable at boot:

```bash
systemctl enable NetworkManager
```

---

# Important Commands

```bash
nmcli connection show
nmcli connection show --active
nmcli device status
nmcli device show
ip addr
ip route
ping
ss -tulnp
```

---

# Troubleshooting Methodology

Always troubleshoot in this order:

1. Check the Network Interface
2. Check the IP Address
3. Check the Default Gateway
4. Test the Gateway
5. Test Internet Connectivity by IP
6. Check DNS Resolution
7. Check Firewall
8. Check Listening Services and Ports
