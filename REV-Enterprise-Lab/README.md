# REV-Enterprise-Lab

## 🏢 Project Description

**REV-Enterprise-Lab** is a comprehensive Windows Server infrastructure laboratory environment designed to simulate real-world enterprise IT operations. This project demonstrates the implementation and management of core Windows Server services including Active Directory, Group Policy, DHCP, DNS, File Server management, client configuration, and security hardening.

## 🎯 Objectives

- Design and implement a complete enterprise Windows Server environment
- Deploy and configure Active Directory with proper organizational structure
- Implement Group Policy Objects for security and user management
- Configure network services (DHCP, DNS) for optimal performance
- Establish secure file server infrastructure with proper permissions
- Validate all configurations through comprehensive testing procedures
- Document all processes for enterprise-grade IT operations

## 🛠️ Technologies Used

| Technology | Version | Purpose |
|------------|---------|---------|
| Windows Server | 2019/2022 | Primary Server OS |
| Windows 10 | Enterprise | Client Workstation |
| Active Directory | Domain Services | Identity Management |
| Group Policy | Management Console | Policy Enforcement |
| DHCP Server | Role | IP Address Management |
| DNS Server | Role | Name Resolution |
| File Server | Role | Centralized Storage |
| PowerShell | 5.1+ | Automation & Management |

## 🏗️ Environment Details

### Network Configuration
- **Domain Name**: `rev.local`
- **Primary Domain Controller**: `PDC.rev.local`
- **Server IP Address**: `192.168.1.10`
- **Client Workstation**: `HRPC01.rev.local`
- **Reserved Client IP**: `192.168.1.200`

### IP Address Scheme
| Component | IP Address | Subnet Mask | Gateway |
|-----------|-------------|-------------|---------|
| PDC Server | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| HRPC01 Client | 192.168.1.200 | 255.255.255.0 | 192.168.1.1 |
| DHCP Scope | 192.168.1.40-230 | 255.255.255.0 | 192.168.1.1 |

## 📋 Architecture Overview

The REV-Enterprise-Lab implements a hierarchical enterprise architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                    REV.ENTERPRISE.LAB                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   PDC       │  │  File Server│  │   Client    │         │
│  │192.168.1.10 │  │192.168.1.15 │  │192.168.1.200│         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│                    Core Services                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │Active Dir   │ │   DHCP      │ │    DNS      │            │
│  │Domain Svc   │ │   Server    │ │   Server    │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
├─────────────────────────────────────────────────────────────┤
│                  Organizational Units                       │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐            │
│  │   HR    │ │    HK   │ │  Sales  │ │    IT   │            │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘            │
└─────────────────────────────────────────────────────────────┘
```

## ✨ Features Implemented

### 🔐 Identity & Access Management
- Active Directory Domain Services deployment
- Organizational Unit structure for departmental organization
- User and group management with proper naming conventions
- Computer account management for domain-joined devices

### 📋 Policy Management
- Group Policy Objects for security enforcement
- User restrictions (Command Prompt, Control Panel)
- Removable storage access controls
- Software restriction policies
- Password and account lockout policies

### 🌐 Network Services
- DHCP server with scope configuration and reservations
- DNS server with A records and round-robin load balancing
- Network topology design and IP planning

### 💾 File Services
- Shared folder creation with NTFS permissions
- Drive mapping via Group Policy
- Disk quota management
- Department-specific access controls

### 🔒 Security Hardening
- Password complexity requirements
- Account lockout policies
- Endpoint security restrictions
- Removable media controls

### 🖥️ Client Management
- Windows 10 domain join procedures
- Local administrator delegation
- Group Policy validation
- Application deployment via GPO

## 📸 Screenshots

### Architecture Overview
![Architecture Diagram](screenshots/01-architecture/architecture-overview.png)

### Active Directory Structure
![AD Users and Computers](screenshots/02-active-directory/ad-structure.png)

### Group Policy Management
![Group Policy Console](screenshots/03-group-policy/gpo-management.png)

### Network Services Configuration
![DHCP DNS Configuration](screenshots/04-network-services/network-services.png)

### File Server Setup
![Shared Folders](screenshots/05-file-server/file-server.png)

### Client Configuration
![Domain Join](screenshots/06-client-configuration/domain-join.png)

### Security Policies
![Security Settings](screenshots/07-security/security-policies.png)

## 🎓 Skills Demonstrated

| Category | Skills |
|----------|--------|
| **Server Administration** | Windows Server deployment, Role configuration, Service management |
| **Active Directory** | Domain controller setup, OU design, User/Group management |
| **Group Policy** | GPO creation, Policy linking, Security restrictions |
| **Network Services** | DHCP/DNS configuration, IP planning, Load balancing |
| **File Services** | Share permissions, NTFS rights, Quota management |
| **Security** | Policy enforcement, Access controls, Endpoint protection |
| **Client Management** | Domain joins, Policy validation, Troubleshooting |
| **Documentation** | Technical writing, Process documentation, Validation procedures |

## 🚀 Future Improvements

### Planned Enhancements
- **High Availability**: Implement additional domain controllers
- **Certificate Services**: Deploy AD CS for certificate management
- **Remote Access**: Configure VPN and DirectAccess solutions
- **Monitoring**: Implement System Center Operations Manager
- **Backup Solutions**: Deploy Windows Server Backup strategies
- **Automation**: Create PowerShell automation scripts
- **Cloud Integration**: Hybrid cloud scenarios with Azure AD

### Advanced Features
- **Read-Only Domain Controller**: For branch office scenarios
- **Fine-Grained Password Policies**: Department-specific requirements
- **Dynamic Access Control**: Claims-based access management
- **Workplace Join**: Modern device registration
- **Windows Server Update Services**: Centralized patch management

## ✅ Validation Summary

The project includes comprehensive validation procedures:

| Component | Validation Method | Status |
|-----------|-------------------|---------|
| Active Directory | User login, Group membership | ✅ Validated |
| Group Policy | gpupdate, Policy testing | ✅ Validated |
| DHCP | IP assignment, Reservation testing | ✅ Validated |
| DNS | Name resolution, Load balancing | ✅ Validated |
| File Server | Access testing, Permission verification | ✅ Validated |
| Security | Password policy, Lockout testing | ✅ Validated |

## 📚 Documentation Structure

```
REV-Enterprise-Lab/
├── 01-Architecture/          # Network design and planning
├── 02-Active-Directory/      # AD DS implementation
├── 03-Group-Policy/         # GPO configuration
├── 04-Network-Services/     # DHCP/DNS setup
├── 05-File-Server/          # File services implementation
├── 06-Client-Configuration/ # Workstation management
├── 07-Security/             # Security policies
├── 08-Applications/         # Application deployment
├── 09-Validation/           # Testing and troubleshooting
├── docs/                    # Project documentation
└── screenshots/             # Visual documentation
```

## 🤝 Contributing

This project serves as a comprehensive learning resource for Windows Server administration. Feel free to use it as a reference for your own enterprise infrastructure projects.

## 📄 License

This project is educational and intended for learning purposes. Please ensure compliance with Microsoft licensing terms when using in production environments.

---

**Project Status**: ✅ Complete  
**Last Updated**: May 2026  
**Version**: 1.0.0
