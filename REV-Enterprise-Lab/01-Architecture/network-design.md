# Network Design

## 📋 Description

This document outlines the comprehensive network design for the REV-Enterprise-Lab, including IP addressing schemes, VLAN segmentation, routing protocols, and security considerations for implementing a robust enterprise network infrastructure.

## 🎯 Objectives

- Design a scalable and secure network topology
- Implement proper IP addressing and subnetting
- Ensure network segmentation for security and performance
- Provide redundancy and high availability where required
- Support current and future business requirements

## 🌐 Network Topology

### Physical Topology

```
┌─────────────────────────────────────────────────────────────┐
│                    Enterprise Network Core                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐         ┌─────────────┐                   │
│  │   Core      │         │   Internet  │                   │
│  │   Switch    │◄────────┤   Gateway   │                   │
│  │  (L3 Switch)│         │  Firewall   │                   │
│  └─────────────┘         └─────────────┘                   │
│         │                       │                           │
│  ┌──────┴──────┐        ┌───────┴───────┐                   │
│  │             │        │               │                   │
│  │  ┌─────────┴────────┴─────────────┐ │                   │
│  │  │      Distribution Layer        │ │                   │
│  │  │  ┌─────┐ ┌─────┐ ┌─────┐     │ │                   │
│  │  │  │VLAN10│ │VLAN20│ │VLAN30│     │ │                   │
│  │  │  │Server│ │Client│ │Guest │     │ │                   │
│  │  │  └─────┘ └─────┘ └─────┘     │ │                   │
│  │  └─────────────────────────────────┘ │                   │
│  │                                     │                   │
│  │  ┌─────────────────────────────────┐ │                   │
│  │  │       Access Layer              │ │                   │
│  │  │  ┌─────┐ ┌─────┐ ┌─────┐     │ │                   │
│  │  │  │PDC  │ │HRPC1│ │AP1  │     │ │                   │
│  │  │  └─────┘ └─────┘ └─────┘     │ │                   │
│  │  └─────────────────────────────────┘ │                   │
│  └─────────────────────────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Logical Network Segmentation

| VLAN ID | Name | Purpose | IP Range | CIDR | Security Level |
|---------|------|---------|----------|------|----------------|
| 10 | SERVER_VLAN | Critical Infrastructure | 192.168.1.0/26 | /26 | Critical |
| 20 | CLIENT_VLAN | User Workstations | 192.168.1.64/26 | /26 | High |
| 30 | GUEST_VLAN | Guest/Temporary Access | 192.168.1.128/25 | /25 | Low |
| 40 | MANAGEMENT_VLAN | Administrative Access | 192.168.2.0/28 | /28 | Critical |
| 50 | VOIP_VLAN | Voice Communication | 192.168.2.16/28 | /28 | Medium |
| 60 | IOT_VLAN | IoT Devices | 192.168.2.32/27 | /27 | Low |

## 📊 IP Addressing Scheme

### Primary Network: 192.168.1.0/24

#### VLAN 10 - Server Network (192.168.1.0/26)
| Device | IP Address | Role | MAC Address | Notes |
|--------|------------|------|-------------|-------|
| PDC | 192.168.1.10 | Primary DC | 00:15:5D:01:01:10 | Static |
| FS01 | 192.168.1.15 | File Server | 00:15:5D:01:01:15 | Static |
| APP01 | 192.168.1.20 | Application Server | 00:15:5D:01:01:20 | Static |
| GW-SRV | 192.168.1.1 | Gateway | 00:15:5D:01:01:01 | Router Interface |

#### VLAN 20 - Client Network (192.168.1.64/26)
| Device | IP Address | Role | Assignment Method |
|--------|------------|------|-------------------|
| HRPC01 | 192.168.1.200 | HR Workstation | DHCP Reservation |
| HKPC01 | DHCP | Housekeeping PC | DHCP |
| SALESPC01 | DHCP | Sales Workstation | DHCP |
| ITPC01 | DHCP | IT Workstation | DHCP |
| GW-CLI | 192.168.1.65 | Gateway | Static |

#### VLAN 30 - Guest Network (192.168.1.128/25)
| Range | Purpose | Notes |
|-------|---------|-------|
| 192.168.1.129-254 | Guest Devices | Isolated from internal network |
| GW-GUEST | 192.168.1.129 | Gateway | Captive portal |

### Management Network: 192.168.2.0/24

#### VLAN 40 - Management (192.168.2.0/28)
| Device | IP Address | Role | Access Type |
|--------|------------|------|-------------|
| MGMT-PC | 192.168.2.1 | Management Station | Restricted |
| OOB-CONSOLE | 192.168.2.2 | Out-of-Band Management | Console Access |
| GW-MGMT | 192.168.2.14 | Gateway | Router Interface |

## 🔧 Routing Configuration

### Static Routes

```
# Default Route
0.0.0.0/0 → 192.168.1.1 (Internet Gateway)

# Inter-VLAN Routing
192.168.1.0/26 → VLAN 10 (Direct)
192.168.1.64/26 → VLAN 20 (Direct)
192.168.1.128/25 → VLAN 30 (Direct)
192.168.2.0/28 → VLAN 40 (Direct)

# Management Routes
192.168.2.0/24 → 192.168.2.14 (Management Gateway)
```

### Routing Protocol Configuration

```
# OSPF Configuration (Future Enhancement)
Router OSPF 1
  Network 192.168.1.0 0.0.0.255 area 0
  Network 192.168.2.0 0.0.0.255 area 0
  Passive-interface default
  No passive-interface GigabitEthernet0/0
```

## 🔒 Security Design

### Firewall Rules

#### Inbound Rules
| Source | Destination | Port | Protocol | Action | Description |
|--------|-------------|------|----------|--------|-------------|
| ANY | 192.168.1.10 | 53 | TCP/UDP | ALLOW | DNS Server |
| ANY | 192.168.1.10 | 88 | TCP/UDP | ALLOW | Kerberos |
| ANY | 192.168.1.10 | 389 | TCP/UDP | ALLOW | LDAP |
| ANY | 192.168.1.10 | 445 | TCP | ALLOW | SMB |
| VLAN20 | VLAN10 | ANY | ANY | ALLOW | Client to Server |
| VLAN30 | ANY | 80,443 | TCP | ALLOW | Guest Internet |
| VLAN30 | VLAN10,20 | ANY | ANY | DENY | Guest Isolation |

#### Outbound Rules
| Source | Destination | Port | Protocol | Action | Description |
|--------|-------------|------|----------|--------|-------------|
| ANY | ANY | 53 | TCP/UDP | ALLOW | DNS Resolution |
| ANY | ANY | 80,443 | TCP | ALLOW | HTTP/HTTPS |
| ANY | ANY | 25,587 | TCP | ALLOW | Email (Specific Servers) |
| VLAN30 | INTERNAL | ANY | ANY | DENY | Guest Internet Only |

### Network Access Control (NAC)

| Device Type | Authentication Method | Authorization | Quarantine |
|-------------|----------------------|---------------|------------|
| Servers | Certificate | Server VLAN | N/A |
| Workstations | 802.1X + AD | Department VLAN | Remediation VLAN |
| Guest Devices | Captive Portal | Guest VLAN | Isolated VLAN |
| IoT Devices | MAC Address | IoT VLAN | Monitoring VLAN |

## 📈 Performance Optimization

### Quality of Service (QoS)

| Traffic Class | Priority | Bandwidth Guarantee | Applications |
|---------------|----------|---------------------|--------------|
| Voice | Highest | 25% | VoIP, Video Conferencing |
| Critical Business | High | 40% | AD, Database, ERP |
| Business Applications | Medium | 25% | File Services, Email |
| Best Effort | Low | 10% | Web Browsing, Updates |

### Load Balancing

#### DNS Round Robin
```
www.rev.local:
  192.168.1.8  (Web Server 1)
  192.168.1.9  (Web Server 2)
  192.168.1.10 (Web Server 3)
```

#### Network Load Balancing (Future)
```
Application Load Balancer:
  VIP: 192.168.1.100
  Pool: 192.168.1.20-22
  Algorithm: Round Robin
  Health Check: TCP 8080
```

## 🔧 Network Services Configuration

### DHCP Configuration

#### VLAN 20 - Client Scope
```
Scope: 192.168.1.66-126
Subnet Mask: 255.255.255.192
Default Gateway: 192.168.1.65
DNS Servers: 192.168.1.10, 8.8.8.8
Domain Name: rev.local
Lease Duration: 8 hours

Exclusions:
  192.168.1.80-85 (Reserved for printers)

Reservations:
  192.168.1.200 (HRPC01 - 00:15:5D:01:02:01)
```

#### VLAN 30 - Guest Scope
```
Scope: 192.168.1.130-254
Subnet Mask: 255.255.255.128
Default Gateway: 192.168.1.129
DNS Servers: 8.8.8.8, 8.8.4.4
Lease Duration: 2 hours
```

### DNS Configuration

#### Forward Lookup Zones
```
rev.local:
  @       NS  PDC.rev.local
  PDC     A   192.168.1.10
  FS01    A   192.168.1.15
  APP01   A   192.168.1.20
  www     A   192.168.1.8
  www     A   192.168.1.9
  _ldap   SRV 0 100 389 PDC.rev.local
  _kerberos SRV 0 100 88 PDC.rev.local
```

#### Reverse Lookup Zones
```
1.168.192.in-addr.arpa:
  10      PTR PDC.rev.local
  15      PTR FS01.rev.local
  20      PTR APP01.rev.local
```

## 📊 Network Monitoring

### SNMP Configuration
```
SNMP Community: REV-MONITOR (Read-Only)
SNMP Version: v2c
Trap Receivers: 192.168.2.1 (Management Station)

Monitored Interfaces:
  - GigabitEthernet0/0 (Core Switch)
  - GigabitEthernet0/1-24 (Access Ports)
  - VLAN Interfaces 10,20,30,40
```

### Syslog Configuration
```
Syslog Server: 192.168.2.1
Facility: Local7
Severity Level: Informational
Log Retention: 90 days
```

## 🚀 High Availability Design

### Redundancy Strategies

#### Network Redundancy
- **Core Switch**: Dual power supplies, redundant supervisors
- **Distribution Layer**: Stack switches with cross-connects
- **Access Layer**: Link aggregation (LACP) where possible
- **Uplinks**: Multiple diverse paths to core

#### Service Redundancy
- **Domain Controllers**: Additional DC in future phases
- **DHCP**: Split scope configuration planned
- **DNS**: Secondary DNS server deployment
- **File Services**: DFS replication for redundancy

### Convergence Planning

```
# Spanning Tree Configuration
Spanning-Tree mode Rapid-PVST+
Root Bridge: Core Switch (Priority: 28672)
Backup Root: Distribution Switch (Priority: 32768)

# Link Aggregation
Port-Channel1: Core-Distribution (4x1Gbps LACP)
Port-Channel2: Distribution-Access (2x1Gbps LACP)
```

## 📋 Implementation Timeline

### Phase 1: Core Network (Week 1-2)
- [ ] Core switch configuration
- [ ] VLAN creation and assignment
- [ ] Inter-VLAN routing setup
- [ ] Basic firewall rules

### Phase 2: Service Deployment (Week 3-4)
- [ ] DHCP scope configuration
- [ ] DNS zone setup
- [ ] NAC implementation
- [ ] QoS configuration

### Phase 3: Client Integration (Week 5-6)
- [ ] Workstation network configuration
- [ ] Wireless network setup
- [ ] Guest network implementation
- [ ] Monitoring deployment

### Phase 4: Optimization (Week 7-8)
- [ ] Performance tuning
- [ ] Redundancy implementation
- [ ] Security hardening
- [ ] Documentation completion

## 🖼️ Screenshots

### Network Diagram
![Network Topology](../screenshots/01-architecture/network-topology.png)

### VLAN Configuration
![VLAN Setup](../screenshots/01-architecture/vlan-configuration.png)

### Firewall Rules
![Firewall Configuration](../screenshots/01-architecture/firewall-rules.png)

### DHCP Scope
![DHCP Configuration](../screenshots/01-architecture/dhcp-scope.png)

## 🔍 Validation Procedures

### Connectivity Tests
```powershell
# Test inter-VLAN routing
Test-NetConnection -ComputerName 192.168.1.10 -Port 389
Test-NetConnection -ComputerName 192.168.1.15 -Port 445

# Test DNS resolution
nslookup PDC.rev.local
nslookup www.rev.local

# Test DHCP functionality
ipconfig /release
ipconfig /renew
ipconfig /all
```

### Performance Tests
```powershell
# Bandwidth tests
Test-NetConnection -ComputerName 192.168.1.15 -Port 445
Measure-Command { Copy-Item testfile.dat \\192.168.1.15\share }

# Latency tests
Test-Connection 192.168.1.10 -Count 100
```

### Security Validation
```powershell
# Firewall rule testing
Test-NetConnection -ComputerName 192.168.1.10 -Port 3389
Test-NetConnection -ComputerName 192.168.1.10 -Port 53

# VLAN isolation testing
Test-NetConnection -ComputerName 192.168.1.200 -Port 445
```

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
