# Cybersecurity Home Lab

This repository documents my hands-on cybersecurity home lab, including firewall administration, network segmentation, virtualization, Windows and Linux systems, centralized security monitoring, Active Directory security events, and structured troubleshooting.

The lab is designed to strengthen practical skills in:

- Network security and firewall administration
- TCP/IP networking and segmentation
- Virtualization with Proxmox VE
- Windows Server and Active Directory
- Linux administration
- Wazuh SIEM/XDR monitoring
- File Integrity Monitoring (FIM)
- Authentication and account-management monitoring
- SOC-style event investigation
- Troubleshooting and documentation

## Current Lab Focus

The environment includes five major documented projects:

1. **pfSense Firewall and Network Segmentation**
2. **Proxmox Cybersecurity Home Lab with Wazuh and Active Directory Monitoring**
3. **Microsoft 365 Email Security Monitoring**
4. **Kali–Wazuh SSH Detection and Defense**
5. **Archer BE230 Network Recovery and Kea DHCP Migration**

The Proxmox project now hosts Windows and Linux virtual machines and a dedicated Wazuh monitoring server on the segmented lab network.

## Current Lab Architecture

```text
Internet / Home Network
        |
        v
   pfSense Firewall
 WAN: 192.168.0.15
 LAN: 192.168.10.1
        |
        v
Cybersecurity Lab - 192.168.10.0/24
        |
        +-- Proxmox VE .............. 192.168.10.80
        |    +-- Wazuh Server ....... 192.168.10.30
        |    +-- Kali Linux ......... 192.168.10.60
        |    +-- Ubuntu Server ...... 192.168.10.90
        |    +-- Windows Server 2022
        |    +-- Client VM
        |
        +-- Windows management systems
        +-- Wireless lab access point
```

## Technologies

- pfSense
- Proxmox VE
- Wazuh
- Windows Server 2022
- Active Directory Domain Services (AD DS)
- DNS
- Windows 11
- Ubuntu Server
- Kali Linux
- PowerShell
- Linux command line
- SSH
- UFW
- TCP/IP
- DHCP
- NAT
- Firewall Rules
- Network Segmentation
- File Integrity Monitoring
- Windows Security Event Logs
- Splunk
- Microsoft Sentinel
- Azure Arc
- VMware Fusion

## Project 1 - pfSense Firewall and Network Segmentation

The pfSense project documents firewall deployment, WAN/LAN configuration, isolation rules, troubleshooting, and connectivity validation.

### Documentation

- [`pfSense/README.md`](pfSense/README.md) - project overview
- [`pfSense/installation.md`](pfSense/installation.md) - installation process
- [`pfSense/wan-lan-setup.md`](pfSense/wan-lan-setup.md) - WAN and LAN configuration
- [`pfSense/Firewall-rules.md`](pfSense/Firewall-rules.md) - firewall rules and testing
- [`pfSense/troubleshooting.md`](pfSense/troubleshooting.md) - troubleshooting and lessons learned

### Verified pfSense Skills

- Installed and configured pfSense
- Assigned WAN and LAN interfaces
- Configured lab addressing and gateway settings
- Implemented network isolation rules
- Tested allowed and blocked traffic
- Verified routing and gateway connectivity
- Troubleshot interface and connectivity issues
- Used structured troubleshooting instead of changing multiple settings at once

## Project 2 - Proxmox Cybersecurity Home Lab

The Proxmox project centralizes several cybersecurity systems on one virtualization host and integrates them with the existing pfSense-segmented lab network.

Full documentation is available here:

- [`Proxmox/README.md`](Proxmox/README.md)

### Verified Proxmox and Wazuh Milestones

- Deployed Proxmox VE and configured the host at `192.168.10.80`
- Created Ubuntu Server, Kali Linux, Wazuh Server, Windows Server 2022, and client VMs
- Restored a failed Proxmox bridge by identifying that the physical NIC was missing from the live `vmbr0` bridge
- Reattached the physical interface, reloaded networking, and verified the fix survived a reboot
- Validated gateway, Internet, and VM-to-VM communication
- Verified Wazuh Manager, Indexer, and Dashboard services
- Enrolled Windows, Windows Server, Ubuntu, and Kali agents in Wazuh
- Validated Windows failed and successful logon monitoring
- Configured and tested real-time Windows File Integrity Monitoring
- Validated Active Directory user creation, enable/disable, password reset, lockout/unlock, deletion, and global security-group membership changes
- Validated privileged **Domain Admins** membership monitoring using a controlled test account
- Verified Event ID `4728` when `PrivTest` was added to Domain Admins and Event ID `4729` when it was removed
- Confirmed Wazuh captured the member, target group, actor (`Administrator`), domain (`HOMELAB`), and domain controller (`DC01.homelab.local`)
- Built a reusable Wazuh view for Windows Server / Active Directory security events
- Performed a mini SOC investigation correlating repeated failed logons with an account lockout

## Project 3 - Microsoft 365 Email Security Monitoring

Connected a personal Outlook mailbox to Microsoft Graph with a Python/MSAL monitor, delegated Mail.Read, local token reuse, message-ID deduplication, and five-minute systemd scheduling. Wazuh ingested JSONL email metadata and applied custom rules 100500 (Microsoft account security notifications, level 8), 100501 (high importance, level 6), and 100502 (phishing-style subjects, level 7).

A real Gmail-to-Outlook lab email matched rule 100502 and was verified in Threat Hunting. This is hands-on home-lab work; the keyword rules are triage heuristics.

- [Full documentation, sanitized examples, validation steps, and screenshots](Microsoft-365-Email-Security/README.md)

### Achievement

Built and validated an automated Outlook → Graph → Python/MSAL → Wazuh pipeline with deduplication, five-minute monitoring, three custom detection rules, and real mailbox-to-dashboard test evidence.

## Project 4 - Kali–Wazuh SSH Detection and Defense

Validated controlled SSH authentication failures from Kali to Ubuntu on a dedicated Proxmox test bridge, traced journald events through Wazuh, and confirmed custom rule 100510 in both the manager alert file and dashboard. Disabled password authentication for the dedicated lab account, verified rejection without a password prompt, preserved management access, and removed the temporary firewall allowance.

- [Full investigation, configuration examples, validation, and limitations](Kali-Wazuh-SSH-Lab/README.md)

The historical architecture above describes the management network. During this scenario Kali moved to a dedicated test bridge; Ubuntu retained a management interface with forwarding disabled. Redacted custom-alert and SSH before/after screenshots are included in the project documentation.

## Project 5 - Archer BE230 Recovery and pfSense Kea DHCP Migration

Diagnosed an unreachable Archer BE230 V1 management interface and restored access without a factory reset. An isolated Ethernet test showed its static management IP remained on `192.168.0.2` while the pfSense lab network used `192.168.10.0/24`. Corrected Archer management IP, gateway and DNS; disabled Archer DHCP Auto; verified management access, Wi-Fi, Internet and DNS. Migrated pfSense CE 2.9.0 from ISC to Kea DHCP, validated logs and existing connectivity, and removed obsolete device reservations.

- [Verified recovery procedure and technical evidence](Network-Recovery-Archer-Kea/README.md)
- [Portfolio case study](https://cybersecurity-portfolio-bcu.pages.dev/archer-kea-network-recovery)

## Active Directory Security Monitoring Evidence

The Windows Server 2022 domain controller was monitored through Wazuh and validated against real Windows Security events.

| Event ID | Activity |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4720 | User account created |
| 4722 | User account enabled |
| 4724 | Password reset attempt |
| 4725 | User account disabled |
| 4726 | User account deleted |
| 4728 | Member added to global security group |
| 4729 | Member removed from global security group |
| 4740 | User account locked out |
| 4767 | User account unlocked |

### Privileged Group Monitoring - Domain Admins

A controlled account named `PrivTest` was created and temporarily added to the **Domain Admins** group. Wazuh detected Event ID `4728` and showed the target group as `Domain Admins`, the added member as `PrivTest`, the actor as `Administrator`, and the source system as `DC01.homelab.local`.

The account was then removed from **Domain Admins**. Wazuh detected Event ID `4729` with the same privileged group context, confirming that both privileged membership additions and removals were visible through centralized monitoring.

This test demonstrates practical monitoring of a high-impact Active Directory privilege change while keeping the activity controlled and reversible.

## File Integrity Monitoring

A dedicated Windows test directory was added to the Wazuh `syscheck` configuration for real-time monitoring.

Validated events included:

- File created - Wazuh Rule ID `554`
- File modified / checksum changed - Wazuh Rule ID `550`
- MD5, SHA1, and SHA256 integrity data captured

## SOC-Style Investigation

A controlled account-lockout scenario was used to practice event correlation.

Investigation chain:

```text
4625 Failed Logon
      |
4625 Failed Logon
      |
4625 Failed Logon
      |
4740 Account Lockout
```

The investigation confirmed that repeated failed authentication attempts against the test account caused the lockout. Wazuh provided centralized visibility into both the authentication failures and the resulting lockout event.

Assessment: authorized lab activity; no malicious activity identified.

## Resource Management

The Proxmox host has approximately 16 GB RAM. Running all VMs simultaneously pushed host memory above 90%, so VM usage was adjusted to keep the environment stable.

A practical working combination is:

- Wazuh Server
- Ubuntu Server
- Windows Server 2022

Kali is started when needed for testing and stopped afterward to preserve memory headroom.

## Troubleshooting Approach

A major goal of this lab is developing a repeatable troubleshooting process:

1. Check physical connectivity and interface state
2. Verify IP addressing and subnet configuration
3. Confirm routing and gateway connectivity
4. Validate bridge/interface membership
5. Review firewall and access-control rules
6. Check service status and logs
7. Test host-to-host and VM-to-VM connectivity
8. Make one change at a time
9. Retest after each change
10. Document the final cause and resolution

## Skills Practiced

- Proxmox VE administration
- Virtual machine deployment and resource management
- Linux bridge troubleshooting
- Network segmentation and pfSense administration
- Windows Server administration
- Active Directory Domain Services
- Privileged Active Directory group monitoring
- DNS
- PowerShell
- Linux administration
- Wazuh SIEM/XDR administration
- Endpoint enrollment and monitoring
- Windows Security Event analysis
- File Integrity Monitoring
- Authentication monitoring
- Active Directory account-management monitoring
- Event correlation
- SOC-style incident triage
- Troubleshooting and technical documentation

## Current Status

The lab is operational with pfSense segmentation, Proxmox virtualization, Windows and Linux systems, centralized Wazuh monitoring, Windows Server / Active Directory event collection, real-time FIM, privileged Domain Admins membership monitoring, and a reusable AD security event view.

## Next Phase

Planned next work includes:

- Additional SOC-style investigations
- Custom Wazuh detection rules
- Expanded Linux monitoring
- Additional attack/defense scenarios
- More screenshots and evidence
- Azure Arc and Microsoft Sentinel integration notes

## Purpose

This repository is a technical record of my ongoing cybersecurity learning and home-lab development. It demonstrates practical work with network defense, firewalls, virtualization, Windows and Linux administration, centralized monitoring, Active Directory security, and structured troubleshooting.

## Portfolio

Cybersecurity portfolio:

https://cybersecurity-portfolio-bcu.pages.dev/
