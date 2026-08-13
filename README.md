# Small-Business-Enterprise-Network-Cisco-Packet-Tracer
Designed and configured a small business network in Cisco Packet Tracer using VLANs, router-on-a-stick, DHCP, SSH, NAT/PAT, firewall ACLs, and simulated Internet connectivity. The network separates management, CIS (department), and server traffic while allowing controlled Internet access.

## Network Topology

The network contains:

- 1 Cisco switch
- 1 internal/company router
- 1 firewall/edge router
- 1 ISP router
- Management/HR PC
- CIS/Employee PC
- Internal company server
- Simulated Internet server

The traffic path to the simulated Internet is:

CIS PC → Switch → Company Router → Firewall Router → ISP Router → Internet Server

### Topology Screenshot

Add the complete Packet Tracer topology screenshot here later.

---

## VLAN Configuration

| VLAN | Name | Network | Default Gateway |
|------|------|---------|-----------------|
| 10 | Management | 192.168.10.0/24 | 192.168.10.1 |
| 20 | CIS / Employees | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Server | 192.168.30.0/24 | 192.168.30.1 |

Switch port assignments:

| Port | Purpose | VLAN |
|------|---------|------|
| Fa0/1 | Trunk to Router | Trunk |
| Fa0/2 | Management / HR | VLAN 10 |
| Fa0/3 | CIS / Employee PC | VLAN 20 |
| Fa0/4 | Company Server | VLAN 30 |

---

## Switch Configuration

The switch was configured with separate VLANs for network segmentation.
vlan 10
 name MANAGEMENT

vlan 20
 name EMPLOYEES

vlan 30
 name SERVER
