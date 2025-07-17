# Mellanox-Config-56gbe
Guide to enable or re-enable 56GBE negotiated speeds on Mellanox Connect-X3 Cards and Mellanox Switches with Ethernet Capability. 


# Mellanox 56GbE Link Setup Guide (Infiniband Over Ethernet)

This guide details how to configure Mellanox ConnectX-3 (or similar) NICs and a Mellanox switch to establish a 56GbE Ethernet link. This assumes you're running on Ubuntu/Debian servers.

---

## 🛠️ Prerequisites

* Mellanox ConnectX-3 or higher cards
* Mellanox switch with Ethernet firmware
* DAC cables or optical modules rated for 56Gbps (FDR)
* (Optional) mstflint tools installed on servers
* Firmware should support Ethernet 56G mode

---

## 📌 Switch Configuration (MLNX-OS)

1. Connect to the switch CLI.
2. Enter configuration mode:

```bash
configure terminal
```

3. Select interface:

```bash
interface ethernet 1/3
```

4. Disable interface before changing speed:

```bash
shutdown
```

5. Set port speed to 56G:

```bash
speed 56G
```

6. Re-enable interface:

```bash
no shutdown
```

7. (Optional) Set MTU:

```bash
mtu 9216
```

8. Confirm port status:

```bash
show interfaces ethernet 1/3
```

Ensure **Actual speed: 56 Gbps**.

---

## 💻 Server-Side Setup (Ubuntu/Debian)

### 1. Ensure Proper Cabling

* Use FDR-rated DAC cables or matching optical transceivers.

### 2. Stop NetworkManager (Optional, if interfering)

```bash
sudo nmcli device set enp6s27 managed no
```

### 3. Bring Down Interface

```bash
sudo ip link set enp6s27 down
```

### 4. Force Link Settings

```bash
sudo ethtool -s enp6s27 speed 56000 duplex full autoneg off
sudo ip link set enp6s27 mtu 9216
sudo ip link set enp6s27 up
```

### 5. Confirm Link Status

```bash
ethtool enp6s27
```

Expect:

* **Speed: 56000Mb/s**
* **Duplex: Full**
* **Link detected: yes**

### 6. Assign IPv4 Address

```bash
sudo ip addr add 192.168.40.150/24 dev enp6s27
sudo ip link set enp6s27 up
```

### 7. Verify IP and Connectivity

```bash
ip addr show enp6s27
ping 192.168.40.1
```

### 8. Make Configuration Persistent (Optional)

Using Netplan (`/etc/netplan/99-mellanox.yaml`):

```yaml
network:
  version: 2
  ethernets:
    enp6s27:
      dhcp4: no
      addresses: [192.168.40.150/24]
      gateway4: 192.168.40.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

Apply:

```bash
sudo netplan apply
```

---

## 🔧 (Optional) Firmware and mlxconfig Steps

If needed:

1. Install mstflint:

```bash
sudo apt install mstflint
```

2. Start MST:

```bash
sudo mst start
```

3. Query Configuration:

```bash
sudo mlxconfig -d <PCI_BUS_ID> q
```

4. Set Link Types to Ethernet (if not already):

```bash
sudo mlxconfig -d <PCI_BUS_ID> set LINK_TYPE_P1=2 LINK_TYPE_P2=2
```

5. Reboot NIC or server to apply.

---

## 📊 Performance Testing

* Use `iperf3` or production workloads to verify link speeds and stability.
* Monitor traffic in switch UI to confirm active TX/RX.

---

# Success! You now have a 56GbE Ethernet link active.
