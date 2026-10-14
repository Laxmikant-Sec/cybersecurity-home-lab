# Proxmox Cybersecurity Home Lab

## Overview

This project documents the build, troubleshooting, and security-monitoring validation of a Proxmox VE home lab integrated with pfSense and Wazuh.

The goal is to create a realistic multi-system environment for practicing virtualization, network defense, Windows and Linux administration, Active Directory monitoring, endpoint security, detection engineering, and SOC-style investigations.

## Architecture

The lab includes Proxmox VE, pfSense, Wazuh, Windows Server 2022, Ubuntu Server, Kali Linux, and Windows endpoints on the segmented cybersecurity network.

## Network Recovery and Wazuh Integration — October 1–3, 2026

### Proxmox build and VM foundation

- Deployed Proxmox VE as the main virtualization platform on the isolated `192.168.10.0/24` lab network.
- Configured the Proxmox host with static address `192.168.10.80/24` and gateway `192.168.10.1`.
- Built Ubuntu Server, Wazuh Server, Kali Linux, and Windows Server 2022 virtual machines.
- Confirmed the lab architecture could support Windows, Linux, SIEM, Active Directory, and security-testing workloads.

### Network outage and troubleshooting

A connectivity failure made the Proxmox host unreachable from other lab systems even though the host still showed its static IP configuration.

Troubleshooting included:

- Checking `ip route` and confirming the default route via `192.168.10.1` on `vmbr0`.
- Reviewing `/etc/network/interfaces` and confirming `vmbr0` was configured for `192.168.10.80/24` with gateway `192.168.10.1`.
- Inspecting the Linux bridge and physical NIC state with `bridge link show` and `ip addr`.
- Identifying the physical NIC as down / unavailable in the live bridge path during the failure.
- Bringing the physical interface up and validating its relationship to `vmbr0`.
- Re-testing communication from Proxmox to pfSense and from the Windows management laptop to Proxmox.

### Recovery validation

The repair was verified by successful ICMP communication to the Proxmox host from the Windows management system with `0%` packet loss. The Proxmox web interface at `https://192.168.10.80:8006` became reachable again.

Additional validation included:

- pfSense gateway reachability at `192.168.10.1`.
- pfSense DHCP/static-mapping review for lab systems.
- Windows-to-Proxmox connectivity testing.
- Proxmox host route and bridge verification.
- Confirmation that the VM environment was available again after network recovery.

### Wazuh integration

After network recovery, Wazuh services and endpoint connectivity were validated across the Proxmox lab. Windows Server 2022, Ubuntu, Kali Linux, and Windows endpoints were used for centralized monitoring and subsequent security-event testing.

This work demonstrates a complete troubleshooting cycle: observe symptoms, inspect Layer 2/Layer 3 configuration, isolate the bridge/NIC issue, restore communication, and verify application-level access.

## Active Directory Security Monitoring

Windows Server 2022 was validated as an Active Directory Domain Controller with AD DS and DNS running. Wazuh was used to validate key Windows Security events, including successful and failed logons, account creation and deletion, password and lockout activity, and group-membership changes.

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

### Privileged Domain Admins Monitoring

A controlled test account named `PrivTest` was temporarily added to the **Domain Admins** group and then removed.

Wazuh captured the corresponding privileged-group membership events:

- Event ID `4728` for the addition to Domain Admins
- Event ID `4729` for the removal from Domain Admins

The event details confirmed the changed member, the `Domain Admins` target group, the administrative actor, and the domain controller that recorded the change.

This validated centralized monitoring of high-impact Active Directory privilege changes through Wazuh Threat Hunting.

## Custom Wazuh Detection and Email Alerting — October 5, 2026

The privileged-group monitoring workflow was extended with a custom Wazuh child rule and email notification path.

- Created custom Rule ID `100100` as a child of built-in rule `60159`.
- Matched `Domain Admins` membership changes using `win.eventdata.targetUserName` with PCRE2.
- Raised matching activity to **Wazuh Level 14**.
- Used the alert description `CRITICAL: Privileged Domain Admins membership changed.`
- Validated the rule with controlled Event IDs `4728` and `4729`.
- Configured Postfix on the Wazuh server as a local mail relay.
- Configured Gmail SMTP relay with TLS/SASL and a Google App Password.
- Used `mailq` to diagnose an initial SASL authentication failure.
- Confirmed a standalone Postfix test email before enabling Wazuh email notifications.
- Triggered another controlled Domain Admins membership change and received a Wazuh email showing **Alert level 14**, Rule `100100`, Event ID `4728`, and the custom critical description.

End-to-end workflow:

```text
Active Directory membership change
        |
Windows Security Event 4728 / 4729
        |
Wazuh built-in rule 60159
        |
Custom rule 100100 - Level 14
        |
Postfix relay
        |
Gmail notification
```

Detailed documentation:

- [`custom-wazuh-domain-admin-alerting.md`](custom-wazuh-domain-admin-alerting.md)

## File Integrity Monitoring

Real-time Windows File Integrity Monitoring was configured and validated. Wazuh detected file creation and file modification events and captured integrity hash data.

## SOC-Style Investigation

A controlled account-lockout scenario was used to practice event correlation. Repeated failed authentication events were correlated with the resulting account-lockout event in Wazuh.

## Reusable AD Security View

A Wazuh Threat Hunting view was saved to group important Windows Server and Active Directory event IDs into a reusable monitoring workflow.

## Skills Demonstrated

- Proxmox VE administration
- Linux bridge and NIC troubleshooting
- TCP/IP and static IP configuration
- pfSense integration
- Connectivity testing and recovery validation
- Windows Server administration
- Active Directory Domain Services
- Privileged Active Directory group monitoring
- Wazuh SIEM/XDR administration
- Wazuh custom rule development
- Windows Security Event analysis
- File Integrity Monitoring
- Authentication and account-management monitoring
- Event correlation
- Postfix SMTP relay configuration
- SASL/TLS troubleshooting
- Alert notification workflow validation
- SOC-style incident triage
- Structured troubleshooting
- Technical documentation

## Current Status

The lab is operational with centralized Wazuh monitoring, Active Directory event collection, real-time FIM, privileged Domain Admins membership monitoring, a custom Level 14 Wazuh detection rule, Gmail email alerting through Postfix, and a reusable AD security-event workflow.

## Next Phase

- Additional SOC-style investigations
- Expanded custom Wazuh detections
- Expanded Linux monitoring
- Additional attack/defense scenarios


## Wazuh SOC Security Monitoring Dashboard — October 2026

Built and validated a custom SOC-style security monitoring dashboard using Wazuh SIEM and Windows Security event data collected from an isolated Proxmox home lab.

### Dashboard Visualizations

- **Security Alerts by Severity Level:** Bar chart showing Wazuh alert counts grouped by rule severity.
- **Windows Failed Login Attempts Over Time:** Time-series visualization of Windows Event ID 4625.
- **Group Membership Changes by SID:** Table tracking Windows Event ID 4728 and affected group security identifiers.
- **Critical Domain Admins Security Alerts:** Metric tracking custom Wazuh Rule 100100 (Level 14), with two matching privileged group membership alerts verified on October 5, 2026.

### Validation and Skills Demonstrated

- Investigated Windows Security events and validated custom Wazuh detection rules using Discover and Threat Hunting.
- Confirmed Domain Admins membership-change detection from Windows Server 2022, including the group SID ending in -512.
- Configured event filters, aggregations, a 30-day search window, and a four-panel dashboard.
- Practiced SOC-style monitoring, event correlation, detection validation, and security visualization.

### Dashboard Screenshot

![Wazuh SOC Security Monitoring Dashboard](Wazuh-SOC-Security-Monitoring-Dashboard-Oct-2026.png)

*Screenshot captured October 2026. Counts reflect the selected historical monitoring period and may change with the dashboard time range.*
