# Small Business Enterprise Network - Cisco Packet Tracer

## Project Overview

This project demonstrates the design, configuration, security, testing, and troubleshooting of a small business network using Cisco Packet Tracer.

The network was designed to simulate a small company environment containing separate Management/HR, CIS/Employee, and Server networks. VLANs were used to logically separate departments, while a Cisco router provides inter-VLAN routing using Router-on-a-Stick.

The project also includes DHCP for automatic IP addressing, SSH for secure remote management, static routing, NAT/PAT, an edge/firewall router, an ISP simulation, an internal web server, an external Internet test server, and extended Access Control Lists (ACLs).

Security policies were implemented so that employees cannot access the Management VLAN and can access the internal company server only through approved web services while maintaining Internet connectivity.

The project was tested from end to end and several connectivity problems were intentionally or naturally encountered and troubleshot using Cisco IOS verification commands.

---

## Project Objectives

The main objectives of this project were to:

- Create separate networks for company departments using VLANs
- Configure access and trunk ports on a Cisco switch
- Configure Router-on-a-Stick for inter-VLAN routing
- Configure DHCP for automatic client addressing
- Configure SSH for secure remote administration
- Connect the internal company network to an edge/firewall router
- Configure static and default routing
- Configure NAT/PAT for Internet access
- Simulate an ISP network
- Simulate an external Internet server
- Configure ACLs for internal network security
- Prevent employees from accessing the Management network
- Allow only HTTP/HTTPS access from employees to the internal server
- Verify routing, NAT, ACLs, DHCP, VLANs, and SSH
- Troubleshoot connectivity failures using Cisco IOS commands
- Document the completed network for a technical portfolio

---

# Network Topology

The completed network contains:

- Cisco 2960 access switch
- Internal/company router
- Edge/firewall router
- ISP router
- Management/HR workstation
- CIS/Employee workstation
- Internal company server
- Simulated Internet test server

The main traffic path from an employee workstation to the simulated Internet is:

```text
CIS / Employee PC
        |
        v
   Cisco Switch
        |
        v
  Company Router
        |
        v
 Firewall Router
        |
        v
    ISP Router
        |
        v
Internet Test Server
   198.18.0.10
```

The company router performs inter-VLAN routing.

The firewall router separates the internal company network from the ISP and performs NAT/PAT.

The ISP router represents an external service provider.

The Internet test server represents an external destination used to verify end-to-end connectivity.

---

## Network Topology Screenshot

<img width="1320" height="530" alt="image" src="https://github.com/user-attachments/assets/a2826fab-1fa8-4db4-81ef-f1839d8a72ed" />

---

# IP Addressing Plan

The network uses multiple IPv4 subnets for internal VLANs and external point-to-point connections.

| Network | Purpose | Gateway / Important Address |
|---|---|---|
| 192.168.10.0/24 | Management VLAN | 192.168.10.1 |
| 192.168.20.0/24 | CIS / Employee VLAN | 192.168.20.1 |
| 192.168.30.0/24 | Internal Server VLAN | 192.168.30.1 |
| 10.0.0.0/30 | Company Router ↔ Firewall | 10.0.0.2 / 10.0.0.1 |
| 203.0.113.0/30 | Firewall ↔ ISP | 203.0.113.2 / 203.0.113.1 |
| 198.18.0.0/24 | Simulated Internet Network | 198.18.0.1 |
| 198.18.0.10 | Internet Test Server | Gateway 198.18.0.1 |

---

# VLAN Design

VLANs were created to logically separate company departments and services.

| VLAN | Name | Network | Default Gateway | Purpose |
|---|---|---|---|---|
| 10 | MANAGEMENT | 192.168.10.0/24 | 192.168.10.1 | Management / HR |
| 20 | EMPLOYEES | 192.168.20.0/24 | 192.168.20.1 | CIS / Employee Workstations |
| 30 | SERVER | 192.168.30.0/24 | 192.168.30.1 | Internal Company Server |

VLANs create separate Layer 2 broadcast domains.

This improves organization and security because devices in different VLANs cannot communicate directly without passing through a Layer 3 routing device.

---

# Switch Port Assignments

The Cisco 2960 switch was configured with the following port assignments.

| Switch Port | Connected Device | VLAN |
|---|---|---|
| Fa0/1 | Company Router | Trunk |
| Fa0/2 | Management / HR PC | VLAN 10 |
| Fa0/3 | CIS / Employee PC | VLAN 20 |
| Fa0/4 | Internal Company Server | VLAN 30 |

Fa0/1 operates as an 802.1Q trunk because it must carry traffic belonging to multiple VLANs between the switch and company router.

---

# VLAN Configuration

The following VLANs were created on the switch.

```cisco
vlan 10
 name MANAGEMENT
 exit

vlan 20
 name EMPLOYEES
 exit

vlan 30
 name SERVER
 exit
```

The access ports were then assigned to their appropriate VLANs.

```cisco
interface fa0/2
 switchport mode access
 switchport access vlan 10
 exit

interface fa0/3
 switchport mode access
 switchport access vlan 20
 exit

interface fa0/4
 switchport mode access
 switchport access vlan 30
 exit
```

---

# Trunk Configuration

The connection between the switch and company router was configured as a trunk.

```cisco
interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
 exit
```

A trunk allows traffic from VLAN 10, VLAN 20, and VLAN 30 to travel across one physical connection.

The router determines which VLAN the traffic belongs to using 802.1Q VLAN tags.

---

# Switch Verification

The following commands were used to verify the switch configuration:

```cisco
show vlan brief
show interfaces trunk
show interfaces status
show mac address-table
```

`show vlan brief` verifies VLAN creation and access-port assignments.

`show interfaces trunk` verifies that the router-facing port is operating as a trunk.

`show interfaces status` shows connected interfaces and their current VLAN assignments.

`show mac address-table` was useful during troubleshooting to determine which physical switch port a device was connected to.

### VLAN Verification

<img width="1399" height="544" alt="image" src="https://github.com/user-attachments/assets/63d99b9e-c6a6-44d3-9e8d-cd3240d59705" />


### Trunk Verification

![Switch Trunk Status]([switch-trunk-status.png](https://github.com/gurveer6123-coder/Small-Business-Enterprise-Network-Cisco-Packet-Tracer/blob/main/switch-trunk%20status.png?raw=true))

### Interface Verification

![Switch Interface Status](<img width="1474" height="412" alt="image" src="https://github.com/user-attachments/assets/a6c88573-be7c-4ac3-9893-41f194149905" />
)

---

# Router-on-a-Stick Configuration

Router-on-a-Stick was used to provide inter-VLAN routing.

Instead of dedicating one physical router interface to every VLAN, logical subinterfaces were created on one router interface.

Each subinterface represents one VLAN.

```cisco
interface g0/0/0
 no ip address
 no shutdown
 exit

interface g0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface g0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

interface g0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit
```

The subinterface numbers `.10`, `.20`, and `.30` were selected to correspond with VLAN IDs 10, 20, and 30.

The router addresses act as the default gateways for each VLAN:

```text
VLAN 10 → 192.168.10.1
VLAN 20 → 192.168.20.1
VLAN 30 → 192.168.30.1
```

This allows devices in different VLANs to communicate only after their traffic passes through the company router.

---

# Company Router Interface Verification

The following command was used:

```cisco
show ip interface brief
```

The router showed active subinterfaces for all three VLANs.

![Router IP Interfaces](https://github.com/gurveer6123-coder/Small-Business-Enterprise-Network-Cisco-Packet-Tracer/blob/main/router-ip-interfaces.png?raw=true)

---

# DHCP Configuration

DHCP was configured on the company router so that client computers could automatically receive their IPv4 configuration.

An example DHCP configuration for VLAN 20 is shown below.

```cisco
ip dhcp excluded-address 192.168.20.1 192.168.20.10

ip dhcp pool EMPLOYEES
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
 exit
```

The first addresses in the subnet were excluded from DHCP so they could be reserved for infrastructure such as routers, switches, servers, or printers.

The employee workstation successfully received an address from the `192.168.20.0/24` network.

The following command was used to verify active DHCP leases:

```cisco
show ip dhcp binding
```

### DHCP Verification

![Router DHCP Bindings](screenshots/router-dhcp-bindings.png)

---

# SSH Remote Management

SSH was configured so the router and switch could be managed remotely from a workstation.

SSH was selected instead of Telnet because SSH encrypts management traffic and login credentials.

Example SSH configuration:

```cisco
ip domain-name company.local
username admin privilege 15 secret Admin123

crypto key generate rsa

ip ssh version 2

line vty 0 4
 login local
 transport input ssh
 exit
```

The RSA key size used in the Packet Tracer lab was 1024 bits.

SSH was successfully tested from a company workstation.

Example:

```text
ssh -l admin 192.168.10.1
```

The switch was also successfully accessed remotely through SSH using its management address.

This demonstrated secure remote administration of network infrastructure.

> Note: The credentials used in this Packet Tracer project are lab credentials only and should not be used in a production environment.

---

# Company Router to Firewall Connection

The company router connects to the edge/firewall router using a small `/30` point-to-point subnet.

```text
Network: 10.0.0.0/30

Company Router: 10.0.0.2
Firewall Router: 10.0.0.1
```

The company router interface was configured with:

```cisco
interface g0/0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit
```

A default route was configured on the company router:

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

This tells the company router:

> If the destination is not one of my directly connected networks, forward the traffic to the firewall at 10.0.0.1.

This is how internal traffic begins its path toward the simulated Internet.

---

# Company Router Routing Table

Routing was verified using:

```cisco
show ip route
```

The company router contained connected routes for the internal VLAN networks and the point-to-point firewall network.

It also contained a default route toward `10.0.0.1`.

### Routing Table

![Company Router Routing Table](screenshots/router-routing-table.png)

---

# Firewall / Edge Router Configuration

The edge router acts as the boundary between the private company network and simulated external ISP network.

Its inside interface connects to the company router.

```cisco
interface g0/0
 ip address 10.0.0.1 255.255.255.252
 ip nat inside
 no shutdown
 exit
```

Its outside interface connects to the ISP router.

```cisco
interface g0/1
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown
 exit
```

The firewall router therefore has:

```text
G0/0 → Company Network
10.0.0.1/30

G0/1 → ISP
203.0.113.2/30
```

### Firewall Interfaces

![Firewall IP Interfaces](screenshots/firewall-ip-interfaces.png)

---

# Firewall Default Route

The firewall uses the ISP router as its default gateway.

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

This means traffic destined for external networks is forwarded to the ISP router at `203.0.113.1`.

---

# Firewall Return Route

During troubleshooting, an important route was discovered to be missing.

The firewall knew how to send traffic toward the Internet but initially did not have a route back toward the internal company VLANs.

The following route was added:

```cisco
ip route 192.168.0.0 255.255.0.0 10.0.0.2
```

This tells the firewall:

> To reach any internal 192.168.x.x company network, forward the traffic to the company router at 10.0.0.2.

This route was required for successful end-to-end communication between employee PCs and the simulated Internet.

---

# Firewall Routing Verification

The following command was used:

```cisco
show ip route
```

The firewall routing table contained:

```text
10.0.0.0/30        → directly connected
203.0.113.0/30     → directly connected
192.168.0.0/16     → via 10.0.0.2
0.0.0.0/0          → via 203.0.113.1
```

### Firewall Routing Table

![Firewall Routing Table](screenshots/firewall-routing-table.png)

---

# NAT / PAT Configuration

The company uses private IPv4 addresses such as `192.168.20.2`.

These private addresses are not used directly on the simulated Internet side.

NAT/PAT was therefore configured on the firewall.

First, an ACL identified which internal addresses were eligible for translation.

```cisco
access-list 1 permit 192.168.0.0 0.0.255.255
access-list 1 permit 10.0.0.0 0.0.0.3
```

PAT was then configured:

```cisco
ip nat inside source list 1 interface g0/1 overload
```

The `overload` keyword enables Port Address Translation.

This allows multiple internal devices to share the firewall's outside address:

```text
203.0.113.2
```

For example:

```text
Employee PC private address:
192.168.20.2

Firewall outside address:
203.0.113.2

Internet Server:
198.18.0.10
```

When the employee PC accesses the Internet server, the firewall translates the internal source address from `192.168.20.2` to `203.0.113.2`.

---

# NAT Verification

The following command was used on the firewall:

```cisco
show ip nat translations
```

A working translation appeared similar to:

```text
Pro  Inside global       Inside local       Outside local      Outside global
icmp 203.0.113.2         192.168.20.2       198.18.0.10        198.18.0.10
```

This confirmed that NAT/PAT was operating correctly.

### NAT Translation Screenshot

![Firewall NAT Translations](screenshots/firewall-nat-translations.png)

---

# ISP Router Configuration

An additional router was configured to simulate an Internet Service Provider.

The ISP-facing network between the firewall and ISP uses:

```text
203.0.113.0/30
```

The ISP router interface connected to the firewall uses:

```cisco
interface g0/0
 ip address 203.0.113.1 255.255.255.252
 no shutdown
 exit
```

The ISP interface connected to the external test server uses:

```cisco
interface g0/1
 ip address 198.18.0.1 255.255.255.0
 no shutdown
 exit
```

Therefore:

```text
ISP G0/0 = 203.0.113.1/30
ISP G0/1 = 198.18.0.1/24
```

### ISP Interface Verification

![ISP Interface Status](screenshots/isp-interface-status.png)

---

# Internet Test Server

A Packet Tracer server was configured to represent an external Internet destination.

The server configuration is:

```text
IP Address:      198.18.0.10
Subnet Mask:     255.255.255.0
Default Gateway: 198.18.0.1
```

The ISP router successfully pinged the server:

```text
ping 198.18.0.10
```

This verified connectivity between the ISP network and the simulated Internet server.

### ISP-to-Server Verification

![ISP Server Ping Success](screenshots/isp-server-ping-success.png)

---

# Internal Company Server

The internal company server is located in VLAN 30.

Its network is:

```text
192.168.30.0/24
```

The server uses:

```text
IP Address:      192.168.30.2
Default Gateway: 192.168.30.1
```

The server provides web services to employees.

Security controls were implemented so that the Employee VLAN can access the server using approved web protocols while other traffic is blocked.

---

# Employee Security ACL

An extended Access Control List was configured on the company router.

The ACL controls traffic originating from the Employee/CIS VLAN.

```cisco
ip access-list extended EMPLOYEE_FILTER

 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255

 permit tcp 192.168.20.0 0.0.0.255 host 192.168.30.2 eq 80

 permit tcp 192.168.20.0 0.0.0.255 host 192.168.30.2 eq 443

 deny ip 192.168.20.0 0.0.0.255 host 192.168.30.2

 permit ip 192.168.20.0 0.0.0.255 any
```

The ACL was applied inbound on the Employee VLAN subinterface.

```cisco
interface g0/0/0.20
 ip access-group EMPLOYEE_FILTER in
 exit
```

Applying the ACL inbound means traffic is checked when it enters the router from VLAN 20.

---

# ACL Rule Explanation

The first ACL rule is:

```cisco
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
```

This blocks Employee VLAN devices from accessing the Management VLAN.

The next rule is:

```cisco
permit tcp 192.168.20.0 0.0.0.255 host 192.168.30.2 eq 80
```

This allows employees to access the internal server using HTTP on TCP port 80.

The next rule is:

```cisco
permit tcp 192.168.20.0 0.0.0.255 host 192.168.30.2 eq 443
```

This permits HTTPS on TCP port 443.

The next rule is:

```cisco
deny ip 192.168.20.0 0.0.0.255 host 192.168.30.2
```

This blocks other Employee VLAN traffic to the internal server.

For example, ICMP ping to the server is blocked.

The final rule is:

```cisco
permit ip 192.168.20.0 0.0.0.255 any
```

This permits Employee VLAN traffic to other destinations, including the simulated Internet.

---

# Security Policy

The completed ACL implements the following policy:

| Source | Destination | Service | Result |
|---|---|---|---|
| Employee VLAN | Management VLAN | Any IP | Blocked |
| Employee VLAN | Internal Server | HTTP 80 | Allowed |
| Employee VLAN | Internal Server | HTTPS 443 | Allowed |
| Employee VLAN | Internal Server | Other traffic / ICMP | Blocked |
| Employee VLAN | Simulated Internet | IP | Allowed |

This demonstrates network segmentation and the principle of allowing only the access that users require.

---

# ACL Verification

The ACL was verified using:

```cisco
show access-lists
```

The output showed match counters such as:

```text
10 deny ip 192.168.20.0 ... 192.168.10.0 ... (8 matches)

20 permit tcp 192.168.20.0 ... host 192.168.30.2 eq www

30 permit tcp 192.168.20.0 ... host 192.168.30.2 eq 443

40 deny ip 192.168.20.0 ... host 192.168.30.2 (4 matches)

50 permit ip 192.168.20.0 ... any (8 matches)
```

The counters proved that the ACL was actively processing real test traffic.

### ACL Verification Screenshot

![Router ACL Rules](screenshots/router-acl-rules.png)

---

# Security and Connectivity Testing

Multiple tests were performed from the CIS / Employee workstation.

---

## Test 1 - Employee Access to Management VLAN

The Employee PC attempted to reach the Management VLAN gateway.

```text
ping 192.168.10.1
```

The connection failed as expected.

This confirmed that the ACL successfully prevents employees from accessing the Management network.

### Result

```text
Employee VLAN → Management VLAN = BLOCKED
```

![Employee Management Access Blocked](screenshots/employee-management-blocked.png)

---

## Test 2 - Employee Ping to Internal Server

The employee workstation attempted to ping the internal company server:

```text
ping 192.168.30.2
```

The result was:

```text
Reply from 192.168.20.1: Destination host unreachable.
```

The ping failed because the ACL allows HTTP and HTTPS to the server but blocks other IP traffic.

### Result

```text
Employee VLAN → Server ICMP = BLOCKED
```

![Employee Server Ping Blocked](screenshots/employee-server-ping-blocked.png)

---

## Test 3 - Employee HTTP Access to Internal Server

Although ping traffic was blocked, the employee workstation was allowed to access the web service.

The following URL was opened from the Packet Tracer web browser:

```text
http://192.168.30.2
```

The Cisco Packet Tracer web page loaded successfully.

This confirmed that TCP port 80 was allowed while unwanted traffic remained blocked.

### Result

```text
Employee VLAN → Server HTTP = ALLOWED
```

![Server HTTP Access Success](screenshots/server-http-access-success.png)

---

## Test 4 - Internet Connectivity

The Employee PC tested connectivity to the external simulated Internet server.

```text
ping 198.18.0.10
```

The result was:

```text
Packets: Sent = 4
Received = 4
Lost = 0

0% packet loss
```

This confirmed successful end-to-end communication through the entire network.

### Traffic Path

```text
192.168.20.2
Employee PC
     |
     v
Switch
     |
     v
192.168.20.1
Company Router
     |
     v
10.0.0.1
Firewall
     |
     v
203.0.113.1
ISP
     |
     v
198.18.0.10
Internet Server
```

### Result

```text
Employee VLAN → Internet Server = SUCCESS
```

![Employee Internet Ping Success](screenshots/employee-internet-ping-success.png)

---

# End-to-End Connectivity

After configuration and troubleshooting, the complete communication path worked successfully:

```text
Employee PC
192.168.20.2

        ↓

Cisco 2960 Switch

        ↓

Company Router
192.168.20.1 / 10.0.0.2

        ↓

Firewall Router
10.0.0.1 / 203.0.113.2

        ↓

ISP Router
203.0.113.1 / 198.18.0.1

        ↓

Internet Server
198.18.0.10
```

This test validated VLAN configuration, trunking, Router-on-a-Stick, routing, ACL processing, NAT/PAT, ISP connectivity, and return routing.

---

# Troubleshooting Experience

A major part of this project involved identifying and resolving network problems.

Rather than only configuring devices, the network was tested step-by-step to determine exactly where connectivity stopped.

---

## Troubleshooting Issue 1 - Incorrect VLAN Assignment

During an earlier stage of the project, a client had a valid IP address but could not communicate with its default gateway.

The following commands were used:

```cisco
show interfaces status
show vlan brief
show mac address-table
```

The MAC address table revealed the physical switch port used by the client.

The port was found to be assigned to the wrong VLAN.

After correcting the access VLAN on the switch port, communication with the gateway worked successfully.

This demonstrated that having a correct IP address does not guarantee connectivity if the Layer 2 VLAN configuration is incorrect.

---

## Troubleshooting Issue 2 - Company Router Could Reach Internet but PC Could Not

Another important problem occurred when the company router could successfully ping:

```text
198.18.0.10
```

but the Employee PC could not.

Testing was performed hop-by-hop.

The Company Router could reach the firewall:

```text
ping 10.0.0.1
```

The firewall could reach the ISP:

```text
ping 203.0.113.1
```

The ISP could reach the Internet server:

```text
ping 198.18.0.10
```

This confirmed that the forward path was working.

The issue was eventually identified as a missing return route on the firewall.

The firewall did not know how to route traffic back to the internal `192.168.x.x` company networks.

The problem was fixed using:

```cisco
ip route 192.168.0.0 255.255.0.0 10.0.0.2
```

After adding the route:

```text
Employee PC → Internet Server = SUCCESS
```

NAT translations also appeared correctly on the firewall.

This troubleshooting exercise demonstrated the importance of checking both the forward path and the return path.

---

## Troubleshooting Issue 3 - NAT Verification

When testing NAT, the command:

```cisco
show ip nat translations
```

was used on the firewall.

Initially, no translations appeared.

The firewall interfaces were checked to verify that:

```text
G0/0 = ip nat inside
G0/1 = ip nat outside
```

The NAT ACL and PAT configuration were then verified.

After the Employee PC generated Internet traffic, translations appeared correctly.

This confirmed that NAT/PAT was functioning.

---

# Useful Verification Commands

The following commands were used throughout the project.

### Switch

```cisco
show vlan brief
show interfaces trunk
show interfaces status
show mac address-table
```

### Company Router

```cisco
show ip interface brief
show ip route
show ip dhcp binding
show access-lists
show running-config
```

### Firewall Router

```cisco
show ip interface brief
show ip route
show ip nat translations
show running-config
```

### ISP Router

```cisco
show ip interface brief
show ip route
ping 198.18.0.10
```

These commands were essential for configuration verification and troubleshooting.

---

# Configuration Persistence

After successful configuration, device configurations were saved using:

```cisco
copy running-config startup-config
```

The running configuration exists in RAM and can be lost after a restart.

Saving it to startup-config ensures that the Cisco device reloads the configuration during the next boot.

The complete Cisco Packet Tracer `.pkt` project file was also saved.

---

# Technologies and Concepts Demonstrated

This project demonstrates hands-on knowledge of:

- Cisco Packet Tracer
- Cisco IOS CLI
- IPv4 addressing
- Subnetting
- VLANs
- Access ports
- 802.1Q trunking
- Router-on-a-Stick
- Inter-VLAN routing
- DHCP
- Default gateways
- Static routing
- Default routes
- NAT
- PAT
- Standard ACL concepts
- Extended ACLs
- Network segmentation
- Server access control
- SSH remote administration
- Firewall / edge routing concepts
- ISP simulation
- HTTP and HTTPS access control
- ICMP testing
- Cisco show commands
- Layer 2 troubleshooting
- Layer 3 troubleshooting
- Routing troubleshooting
- NAT troubleshooting
- ACL troubleshooting
- End-to-end connectivity testing
- Technical documentation

---

# Project Outcome

The completed network successfully provides a segmented and secured small business environment.

Management, employee, and server devices operate in separate VLANs.

Router-on-a-Stick provides controlled communication between VLANs.

DHCP automatically provides network configuration to client devices.

SSH provides secure remote administration of network devices.

The company router forwards external traffic to an edge/firewall router.

The firewall provides NAT/PAT and connects the organization to a simulated ISP.

Employees can access the simulated Internet while private internal addresses are translated through PAT.

ACLs protect the Management VLAN and restrict access to the internal company server.

Employees cannot directly access Management resources.

Employees cannot ping the internal server but are allowed to access approved HTTP and HTTPS services.

The final end-to-end connection:

```text
Employee PC
→ Switch
→ Company Router
→ Firewall
→ ISP
→ Internet Server
```

was tested successfully.

The project demonstrates not only network configuration skills but also the ability to test, diagnose, and resolve real networking problems using a structured troubleshooting process.

---

# Repository Structure

```text
Small-Business-Enterprise-Network-Cisco-Packet-Tracer/
│
├── README.md
│
├── Small-Business-Enterprise-Network.pkt
│
└── screenshots/
    ├── network-topology.png
    ├── switch-interface-status.png
    ├── switch-vlan-brief.png
    ├── switch-trunk-status.png
    ├── router-acl-rules.png
    ├── router-dhcp-bindings.png
    ├── router-ip-interfaces.png
    ├── router-routing-table.png
    ├── firewall-ip-interfaces.png
    ├── firewall-nat-translations.png
    ├── firewall-routing-table.png
    ├── isp-interface-status.png
    ├── isp-server-ping-success.png
    ├── employee-management-blocked.png
    ├── employee-server-ping-blocked.png
    ├── employee-internet-ping-success.png
    └── server-http-access-success.png
```

---

# Conclusion

This project provided practical experience designing and securing a small enterprise-style network from the ground up.

It covered switching, routing, addressing, DHCP, SSH, VLAN segmentation, ACL security, NAT/PAT, firewall routing, simulated ISP connectivity, internal server access, troubleshooting, verification, and documentation.

The final network successfully demonstrates how multiple Cisco networking technologies work together to provide secure communication between internal users, business resources, and external networks.
