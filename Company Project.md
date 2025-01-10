# Company Network Project

This repository documents the setup of a company's internal and external network connectivity. The network includes both local circuits and an edge router for internet access. Below is a description of the topology, configurations, and routing rules implemented.

## Network Topology

The network is structured as follows:

- **Edge Router** (`S1C1-EDGE-01`): Connected to the internet with the IP `192.168.1.1/24`.
- **Core Routers**:
  - `S1C2-CORE-1` with interfaces connected to `10.203.11.0/24` and `10.203.12.0/24`.
  - `S1C3-CORE-2` and `S1C4-CORE-3` interconnected locally.
- **Routing Protocol**: OSPF is used for dynamic routing between the routers.
- **Internet Access**: All local traffic is routed through the edge router using OSPF neighbor IDs and `iptables` rules.

## Configurations

### Edge Router Configuration (`S1C1-EDGE-01`)

#### Enable IP Forwarding
```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Make it persistent
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

#### Configure OSPF
```bash
# Install Quagga or FRR for OSPF
sudo apt update && sudo apt install frr -y

# OSPF configuration
vtysh
conf t
router ospf
 network 10.203.11.0/24 area 0
 network 10.203.12.0/24 area 0
 exit
write
```

#### Add iptables Rules for Internet Traffic
```bash
# NAT rule for outgoing traffic
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Forward traffic from local subnets to the internet
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Save the iptables rules
sudo sh -c 'iptables-save > /etc/iptables/rules.v4'
```

### Core Router Configuration (`S1C2-CORE-1`, `S1C3-CORE-2`, `S1C4-CORE-3`)

#### Configure OSPF
```bash
# Example for CORE-1
vtysh
conf t
router ospf
 network 10.203.11.0/24 area 0
 network 10.203.12.0/24 area 0
 exit
write

# Repeat similar configurations for CORE-2 and CORE-3 with appropriate networks
```

### Testing the Configuration

#### Check OSPF Neighbors
```bash
# Verify OSPF neighbor relationships
vtysh -c "show ip ospf neighbor"
```

#### Test Internet Connectivity
```bash
# Ping an external IP from a local device
ping 8.8.8.8
```

#### Verify iptables Rules
```bash
# List active iptables rules
sudo iptables -L -v -n
```

## Notes
- Ensure OSPF is correctly configured on all devices for dynamic routing.
- Use secure and private IP ranges for internal subnets.
- The IP addresses and network structure provided here are placeholders and can be adjusted for specific use cases.
