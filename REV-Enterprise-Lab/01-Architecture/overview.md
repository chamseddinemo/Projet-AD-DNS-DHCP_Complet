# Architecture Overview

## 📋 Description

This document provides a comprehensive overview of the REV-Enterprise-Lab architecture, including the infrastructure design, component relationships, and deployment strategy for simulating a real enterprise Windows Server environment.

## 🎯 Objectives

- Design a scalable and secure enterprise network architecture
- Implement proper separation of concerns across different service layers
- Ensure high availability and redundancy where applicable
- Create a maintainable and documented infrastructure
- Support future growth and expansion requirements

## 🏗️ Infrastructure Components

### Core Server Infrastructure

| Component | Role | IP Address | OS Version | Purpose |
|-----------|------|------------|-------------|---------|
| PDC | Primary Domain Controller | 192.168.1.10 | Windows Server 2019/2022 | AD DS, DHCP, DNS, File Services |
| FS01 | File Server | 192.168.1.15 | Windows Server 2019/2022 | Centralized File Storage |
| APP01 | Application Server | 192.168.1.20 | Windows Server 2019/2022 | Business Application Hosting |

### Client Infrastructure

| Component | Role | IP Address | OS Version | Department |
|-----------|------|------------|-------------|------------|
| HRPC01 | HR Workstation | 192.168.1.200 | Windows 10 Enterprise | Human Resources |
| HKPC01 | Housekeeping Workstation | DHCP | Windows 10 Enterprise | Housekeeping |
| SALESPC01 | Sales Workstation | DHCP | Windows 10 Enterprise | Sales |
| ITPC01 | IT Workstation | DHCP | Windows 10 Enterprise | IT Department |

## 🌐 Network Architecture

### Physical Network Design

```
┌─────────────────────────────────────────────────────────────┐
│                    REV Enterprise Network                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐         ┌─────────────┐                   │
│  │   Switch    │─────────│    Router    │                   │
│  │ 192.168.1.0 │         │ 192.168.1.1 │                   │
│  └─────────────┘         └─────────────┘                   │
│         │                       │                           │
│  ┌──────┴──────┐        ┌───────┴───────┐                   │
│  │             │        │               │                   │
│  │  ┌─────────┴────────┴─────────────┐ │                   │
│  │  │        Server Infrastructure   │ │                   │
│  │  │  ┌─────┐ ┌─────┐ ┌─────┐     │ │                   │
│  │  │  │ PDC │ │FS01 │ │APP01│     │ │                   │
│  │  │  └─────┘ └─────┘ └─────┘     │ │                   │
│  │  └─────────────────────────────────┘ │                   │
│  │                                     │                   │
│  │  ┌─────────────────────────────────┐ │                   │
│  │  │       Client Workstations       │ │                   │
│  │  │  ┌─────┐ ┌─────┐ ┌─────┐     │ │                   │
│  │  │  │HRPC1│ │HKPC1│ │SLS1 │     │ │                   │
│  │  │  └─────┘ └─────┘ └─────┘     │ │                   │
│  │  └─────────────────────────────────┘ │                   │
│  └─────────────────────────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Logical Network Segmentation

| Network Segment | IP Range | Purpose | Security Level |
|-----------------|----------|---------|----------------|
| Server Network | 192.168.1.10-30 | Critical Infrastructure | High |
| Client Network | 192.168.1.40-230 | User Workstations | Medium |
| Management | 192.168.1.240-254 | Administrative Access | Critical |
| Guest/DMZ | 192.168.2.0/24 | Temporary Access | Low |

## 🔧 Service Architecture

### Active Directory Architecture

```
rev.local Forest
├── rev.local Domain
│   ├── Domain Controllers OU
│   │   └── PDC
│   ├── Servers OU
│   │   ├── File Servers
│   │   └── Application Servers
│   ├── Workstations OU
│   │   ├── HR
│   │   ├── Housekeeping
│   │   ├── Sales
│   │   └── IT
│   ├── Groups OU
│   │   ├── Security Groups
│   │   └── Distribution Groups
│   └── Service Accounts OU
```

### DNS Architecture

| Zone Type | Zone Name | Purpose | Records |
|-----------|-----------|---------|---------|
| Primary | rev.local | Internal domain resolution | A, CNAME, SRV |
| Reverse | 1.168.192.in-addr.arpa | IP to name resolution | PTR |
| Forwarder | Various | Internet resolution | Root hints |

### DHCP Architecture

| Scope | Range | Exclusions | Reservations | Options |
|-------|-------|------------|--------------|---------|
| Primary | 192.168.1.40-230 | 192.168.1.80-85 | 192.168.1.200 (HRPC01) | DNS, Gateway, Domain |

## 🔒 Security Architecture

### Defense in Depth Strategy

1. **Network Layer**
   - Firewall rules and segmentation
   - VLAN separation
   - Network access control

2. **Host Layer**
   - Windows Firewall
   - Antivirus protection
   - Host-based intrusion detection

3. **Application Layer**
   - Application whitelisting
   - Code signing requirements
   - Software restriction policies

4. **Data Layer**
   - BitLocker encryption
   - NTFS permissions
   - Data classification

### Trust Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                    Trust Model                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    Transitive    ┌─────────────┐           │
│  │ rev.local   │◄─────────────────►│ Child Domain│           │
│  │ (Parent)    │                  │ (Future)    │           │
│  └─────────────┘                  └─────────────┘           │
│         │                                   │               │
│  ┌─────────────┐    One-Way Trust    ┌─────────────┐       │
│  │ External    │◄───────────────────►│ rev.local   │       │
│  │ Partner     │                    │ (Resource)  │       │
│  └─────────────┘                    └─────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 📊 Performance Considerations

### Server Resource Planning

| Server | CPU | RAM | Storage | Network |
|--------|-----|-----|---------|---------|
| PDC | 4 cores | 8GB | 100GB SSD | 1Gbps |
| FS01 | 4 cores | 16GB | 500GB SSD + 2TB HDD | 1Gbps |
| APP01 | 8 cores | 32GB | 200GB SSD | 1Gbps |

### Network Bandwidth Requirements

| Service | Required Bandwidth | Concurrent Users | Priority |
|---------|-------------------|------------------|----------|
| File Services | 100 Mbps | 50 | High |
| Active Directory | 10 Mbps | 200 | Critical |
| Application Access | 50 Mbps | 30 | Medium |
| Internet Access | 200 Mbps | 100 | Medium |

## 🚀 Deployment Strategy

### Phase 1: Core Infrastructure
1. Server hardware preparation
2. Windows Server installation
3. Network configuration
4. Active Directory deployment

### Phase 2: Service Implementation
1. DHCP and DNS configuration
2. File server deployment
3. Group Policy implementation
4. Security baseline configuration

### Phase 3: Client Integration
1. Workstation preparation
2. Domain join procedures
3. Application deployment
4. User training and documentation

### Phase 4: Validation and Optimization
1. Comprehensive testing
2. Performance tuning
3. Security validation
4. Documentation completion

## 📈 Scalability Considerations

### Growth Planning

| Metric | Current Capacity | Target Capacity | Scaling Strategy |
|--------|------------------|-----------------|------------------|
| Users | 50 | 500 | Additional DCs |
| Storage | 2TB | 10TB | Storage expansion |
| Applications | 5 | 25 | Load balancing |
| Network Bandwidth | 1Gbps | 10Gbps | Network upgrade |

### High Availability Roadmap

1. **Short Term (3-6 months)**
   - Additional domain controller
   - DHCP failover
   - DNS round robin

2. **Medium Term (6-12 months)**
   - File server clustering
   - Network load balancing
   - Backup domain controller

3. **Long Term (12+ months)**
   - Multi-site replication
   - Disaster recovery site
   - Cloud backup integration

## 📋 Configuration Standards

### Naming Conventions

| Object Type | Format | Example |
|-------------|--------|---------|
| Servers | [DEPT]-[ROLE]-[NN] | IT-DC-01, HR-FS-01 |
| Workstations | [DEPT]-PC-[NN] | HR-PC-01, IT-PC-01 |
| Users | firstname.lastname | john.smith |
| Groups | [DEPT]-[TYPE] | HR-Group, IT-Admins |
| OUs | [Department] | Human Resources, IT |

### IP Address Allocation

| Range | Purpose | Assignment Method |
|-------|---------|-------------------|
| 192.168.1.1-9 | Infrastructure | Static |
| 192.168.1.10-39 | Servers | Static |
| 192.168.1.40-230 | Clients | DHCP |
| 192.168.1.231-239 | Printers | Static |
| 192.168.1.240-254 | Management | Static |

## 🖼️ Screenshots

### Network Diagram
![Network Architecture Diagram](../screenshots/01-architecture/network-diagram.png)

### Server Infrastructure
![Server Rack Layout](../screenshots/01-architecture/server-infrastructure.png)

### Active Directory Structure
![AD Architecture](../screenshots/01-architecture/ad-architecture.png)

## 🔍 Validation Procedures

### Network Connectivity Tests
- Ping gateway from all segments
- DNS resolution tests
- DHCP lease verification
- Inter-segment routing tests

### Service Availability Tests
- Domain controller authentication
- File server access
- Application connectivity
- Group Policy application

### Performance Benchmarks
- Network latency measurements
- File transfer speeds
- Authentication response times
- Resource utilization monitoring

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
