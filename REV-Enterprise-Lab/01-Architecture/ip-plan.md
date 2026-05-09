# IP Address Plan

## 📋 Description

This document provides a comprehensive IP addressing plan for the REV-Enterprise-Lab environment, including detailed allocation schemes, subnet calculations, and management procedures for efficient network resource utilization.

## 🎯 Objectives

- Establish a scalable IP addressing scheme
- Ensure efficient utilization of address space
- Provide clear documentation for all IP assignments
- Support network segmentation and security policies
- Facilitate future network expansion

## 📊 IP Address Space Allocation

### Primary Network: 192.168.1.0/24

#### Network Summary
| Network | CIDR | Total Addresses | Usable Addresses | Broadcast |
|---------|------|-----------------|------------------|-----------|
| 192.168.1.0 | /24 | 256 | 254 | 192.168.1.255 |

#### Subnet Breakdown

| Subnet | CIDR | Range | Gateway | Purpose | Reserved |
|--------|------|-------|---------|---------|----------|
| 192.168.1.0/26 | /26 | 192.168.1.1-62 | 192.168.1.1 | Server Infrastructure | 192.168.1.10-20 |
| 192.168.1.64/26 | /26 | 192.168.1.65-126 | 192.168.1.65 | Client Workstations | 192.168.1.80-85 |
| 192.168.1.128/25 | /25 | 192.168.1.129-254 | 192.168.1.129 | Guest Network | 192.168.1.240-254 |

## 🖥️ Server Infrastructure (192.168.1.0/26)

### Static IP Assignments

| Server | IP Address | Subnet Mask | Gateway | DNS | Role | MAC Address |
|--------|------------|-------------|---------|-----|------|-------------|
| PDC | 192.168.1.10 | 255.255.255.192 | 192.168.1.1 | 192.168.1.10 | Primary DC | 00:15:5D:01:01:10 |
| FS01 | 192.168.1.15 | 255.255.255.192 | 192.168.1.1 | 192.168.1.10 | File Server | 00:15:5D:01:01:15 |
| APP01 | 192.168.1.20 | 255.255.255.192 | 192.168.1.1 | 192.168.1.10 | Application Server | 00:15:5D:01:01:20 |
| WEB01 | 192.168.1.8 | 255.255.255.192 | 192.168.1.1 | 192.168.1.10 | Web Server | 00:15:5D:01:01:08 |
| WEB02 | 192.168.1.9 | 255.255.255.192 | 192.168.1.1 | 192.168.1.10 | Web Server | 00:15:5D:01:01:09 |
| GW-SRV | 192.168.1.1 | 255.255.255.192 | N/A | 192.168.1.10 | Router Interface | 00:15:5D:01:01:01 |

### Reserved IP Range
- **192.168.1.21-30**: Reserved for future servers
- **192.168.1.31-40**: Reserved for network devices
- **192.168.1.41-50**: Reserved for virtual infrastructure

## 💻 Client Workstations (192.168.1.64/26)

### DHCP Scope Configuration

| Parameter | Value | Description |
|-----------|-------|-------------|
| Scope Range | 192.168.1.66-126 | Available for clients |
| Subnet Mask | 255.255.255.192 | /26 network |
| Default Gateway | 192.168.1.65 | Client gateway |
| Primary DNS | 192.168.1.10 | Internal DNS server |
| Secondary DNS | 8.8.8.8 | External DNS |
| Domain Name | rev.local | Active Directory domain |
| Lease Duration | 8 hours | Standard lease time |

### DHCP Exclusions
| Range | Purpose | Notes |
|-------|---------|-------|
| 192.168.1.80-85 | Printers | Static assignment for network printers |
| 192.168.1.90-95 | Network Devices | Switches, APs, cameras |
| 192.168.1.100-110 | Special Equipment | Lab equipment, testing devices |

### DHCP Reservations

| Device | IP Address | MAC Address | User | Department |
|--------|------------|-------------|------|------------|
| HRPC01 | 192.168.1.200 | 00:15:5D:01:02:01 | HR Manager | Human Resources |
| HKPC01 | 192.168.1.201 | 00:15:5D:01:02:02 | HK Supervisor | Housekeeping |
| SALESPC01 | 192.168.1.202 | 00:15:5D:01:02:03 | Sales Rep | Sales |
| ITPC01 | 192.168.1.203 | 00:15:5D:01:02:04 | IT Admin | IT Department |
| CONFPC01 | 192.168.1.204 | 00:15:5D:01:02:05 | Conference Room | General |

### Workstation Naming Convention
- **Format**: [DEPT]-PC-[NN]
- **Examples**: HR-PC-01, IT-PC-01, SALES-PC-01
- **Maximum**: 99 workstations per department

## 🌐 Guest Network (192.168.1.128/25)

### Guest DHCP Configuration

| Parameter | Value | Description |
|-----------|-------|-------------|
| Scope Range | 192.168.1.130-239 | Available for guests |
| Subnet Mask | 255.255.255.128 | /25 network |
| Default Gateway | 192.168.1.129 | Guest gateway |
| Primary DNS | 8.8.8.8 | Public DNS |
| Secondary DNS | 8.8.4.4 | Public DNS |
| Lease Duration | 2 hours | Short lease for guests |

### Guest Network Restrictions
- **Internet Only**: No access to internal resources
- **Captive Portal**: Authentication required
- **Bandwidth Limited**: 1 Mbps per device
- **Session Timeout**: 4 hours maximum

## 📱 Management Network (192.168.2.0/24)

### Management Subnets

| Subnet | CIDR | Range | Purpose |
|--------|------|-------|---------|
| 192.168.2.0/28 | /28 | 192.168.2.1-14 | Out-of-Band Management |
| 192.168.2.16/28 | /28 | 192.168.2.17-30 | Monitoring Systems |
| 192.168.2.32/27 | /27 | 192.168.2.33-62 | Backup Infrastructure |
| 192.168.2.64/26 | /26 | 192.168.2.65-126 | Virtualization Management |

### Management Device Assignments

| Device | IP Address | Purpose | Access Method |
|--------|------------|---------|---------------|
| MGMT-CONSOLE | 192.168.2.1 | Primary Management Station | RDP, SSH |
| OOB-SWITCH | 192.168.2.2 | Out-of-Band Switch | Console, SSH |
| MON-SERVER | 192.168.2.17 | Monitoring System | Web UI |
| BACKUP-SRV | 192.168.2.33 | Backup Server | RDP |
| VCENTER | 192.168.2.65 | vCenter Server | Web UI |

## 🔧 DNS Records Configuration

### Forward Lookup Zone: rev.local

| Name | Type | TTL | Data | Description |
|------|------|-----|------|-------------|
| @ | SOA | 3600 | PDC.rev.local admin.rev.local 1 3600 900 604800 86400 | Start of Authority |
| @ | NS | 3600 | PDC.rev.local | Name Server |
| PDC | A | 3600 | 192.168.1.10 | Primary DC |
| FS01 | A | 3600 | 192.168.1.15 | File Server |
| APP01 | A | 3600 | 192.168.1.20 | Application Server |
| www | A | 3600 | 192.168.1.8 | Web Server 1 |
| www | A | 3600 | 192.168.1.9 | Web Server 2 |
| mail | CNAME | 3600 | APP01.rev.local | Mail Server |
| files | CNAME | 3600 | FS01.rev.local | File Server |
| _ldap._tcp | SRV | 3600 | 0 100 389 PDC.rev.local | LDAP Service |
| _kerberos._tcp | SRV | 3600 | 0 100 88 PDC.rev.local | Kerberos Service |

### Reverse Lookup Zone: 1.168.192.in-addr.arpa

| IP | PTR Record | TTL | Description |
|----|------------|-----|-------------|
| 1 | GW-SRV.rev.local | 3600 | Gateway |
| 8 | WEB01.rev.local | 3600 | Web Server 1 |
| 9 | WEB02.rev.local | 3600 | Web Server 2 |
| 10 | PDC.rev.local | 3600 | Primary DC |
| 15 | FS01.rev.local | 3600 | File Server |
| 20 | APP01.rev.local | 3600 | Application Server |

## 📊 IP Address Utilization

### Current Utilization

| Network | Total | Used | Available | Utilization |
|---------|-------|------|-----------|-------------|
| 192.168.1.0/26 | 62 | 8 | 54 | 12.9% |
| 192.168.1.64/26 | 62 | 5 | 57 | 8.1% |
| 192.168.1.128/25 | 126 | 1 | 125 | 0.8% |
| 192.168.2.0/24 | 254 | 5 | 249 | 2.0% |

### Projected Growth (12 months)

| Network | Current | Projected | Available | Status |
|---------|---------|-----------|-----------|--------|
| Server Network | 8 | 15 | 54 | ✅ Healthy |
| Client Network | 5 | 50 | 57 | ✅ Healthy |
| Guest Network | 1 | 20 | 125 | ✅ Healthy |
| Management | 5 | 10 | 249 | ✅ Healthy |

## 🔄 IP Address Management Procedures

### Static IP Assignment Process

1. **Request Submission**
   - Submit IP request to IT department
   - Include device type, location, and purpose
   - Specify required duration (permanent/temporary)

2. **IP Allocation**
   - Review available IP ranges
   - Assign IP from appropriate subnet
   - Update IP address database

3. **Configuration**
   - Configure device with assigned IP
   - Update DNS records if required
   - Test connectivity and name resolution

4. **Documentation**
   - Update IP address spreadsheet
   - Create device record in inventory system
   - Notify network team of changes

### DHCP Management Process

1. **Scope Monitoring**
   - Review DHCP utilization weekly
   - Monitor for address exhaustion
   - Analyze lease patterns

2. **Reservation Management**
   - Process reservation requests
   - Update DHCP reservations
   - Document all changes

3. **Scope Adjustments**
   - Modify scope parameters as needed
   - Add/remove exclusions
   - Adjust lease durations

### IP Reclamation Process

1. **Inactive Device Identification**
   - Scan for inactive IP addresses
   - Review DHCP lease history
   - Check DNS records

2. **Verification**
   - Ping test IP addresses
   - Check switch MAC address tables
   - Verify with device owners

3. **Reclamation**
   - Remove static IP assignments
   - Delete DHCP reservations
   - Update documentation

## 📈 Future Expansion Planning

### Additional Subnet Requirements

| Purpose | Required IPs | Proposed Subnet | Timeline |
|---------|--------------|-----------------|----------|
| VoIP Network | 100 | 192.168.3.0/25 | Q3 2026 |
| IoT Devices | 200 | 192.168.3.128/25 | Q4 2026 |
| DMZ | 50 | 192.168.4.0/26 | Q1 2027 |
| Branch Office | 254 | 192.168.5.0/24 | Q2 2027 |

### IPv6 Transition Plan

| Phase | Objective | Implementation |
|-------|-----------|----------------|
| Phase 1 | IPv6 Infrastructure | Configure router interfaces |
| Phase 2 | DNS IPv6 | Add AAAA records |
| Phase 3 | Client IPv6 | Enable IPv6 on workstations |
| Phase 4 | Dual Stack | Full IPv4/IPv6 operation |

## 🔍 Monitoring and Reporting

### IP Address Monitoring

```powershell
# DHCP Scope Monitoring
Get-DhcpServerv4Scope -ComputerName PDC.rev.local | 
    Select-Object ScopeId, Name, State, AddressRange

# DNS Record Monitoring
Get-DnsServerResourceRecord -ZoneName rev.local -ComputerName PDC.rev.local |
    Where-Object {$_.RecordType -eq "A"}

# IP Utilization Report
$scopes = Get-DhcpServerv4Scope -ComputerName PDC.rev.local
foreach ($scope in $scopes) {
    $stats = Get-DhcpServerv4ScopeStatistics -ComputerName PDC.rev.local -ScopeId $scope.ScopeId
    Write-Output "Scope: $($scope.ScopeId) - Utilization: $($stats.PercentageInUse)%"
}
```

### Automated Alerts

| Alert Type | Threshold | Action |
|------------|-----------|--------|
| DHCP Scope Utilization | >80% | Email notification |
| DNS Record Changes | Any | Log event |
| Static IP Conflicts | Detection | Alert IT team |
| Unused IP Addresses | 30 days | Review for reclamation |

## 🖼️ Screenshots

### IP Address Plan
![IP Address Allocation](../screenshots/01-architecture/ip-allocation.png)

### DHCP Configuration
![DHCP Scope Setup](../screenshots/01-architecture/dhcp-setup.png)

### DNS Records
![DNS Zone Configuration](../screenshots/01-architecture/dns-records.png)

### Network Utilization
![IP Utilization Report](../screenshots/01-architecture/utilization-report.png)

## 📋 IP Address Database Template

| IP Address | Device Name | MAC Address | User | Department | Assignment Date | Expiry Date | Notes |
|-------------|-------------|-------------|------|------------|-----------------|-------------|-------|
| 192.168.1.10 | PDC | 00:15:5D:01:01:10 | System | IT | 01/05/2026 | Permanent | Primary DC |
| 192.168.1.15 | FS01 | 00:15:5D:01:01:15 | System | IT | 01/05/2026 | Permanent | File Server |
| 192.168.1.200 | HR-PC-01 | 00:15:5D:01:02:01 | John.Doe | HR | 02/05/2026 | Permanent | HR Manager |

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: August 2026
