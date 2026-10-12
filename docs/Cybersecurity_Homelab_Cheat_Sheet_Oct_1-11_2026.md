> Public sanitized copy. Substitute example placeholders with the addresses and names in your private operational notes. Previous Git commits may still show older versions.

# CYBERSECURITY HOME LAB — MASTER COMMAND CHEAT SHEET
**Project window:** October 1–11, 2026  
**Prepared:** October 12, 2026  
**Scope:** Proxmox, pfSense, Ubuntu, Kali, Wazuh, Windows Server 2022, Active Directory, Outlook/Microsoft Graph, Postfix, TP-Link AP, portfolio and GitHub.

> **How to use this guide:** `[VERIFIED]` = action or configuration supported by our project history; `[REFERENCE]` = helpful diagnostic or repeatable command (not necessarily previously run); `[CAUTION]` = changes state, may cause outages, or should be used only in a controlled lab. Commands are examples unless the specific execution was confirmed. Execute commands on the device named in the heading; never paste an entire troubleshooting section blindly. No passwords, tokens, client IDs, or app passwords are reproduced.

## 1. Project timeline and accomplishments

| Date | Work | Outcome / evidence |
|---|---|---|
| Oct 1 | ASUS laptop repurposed as Proxmox host; Ubuntu VM | Proxmox VE 9.x; Ubuntu VM 100 provisioned with 2 vCPU, 4 GB RAM, 60 GB disk, VirtIO bridge; Windows/Kali/Wazuh VMs followed. |
| Oct 1–3 | Proxmox network recovery; pfSense isolation; Wazuh integration | Repaired USB Ethernet bridge; Wazuh server and Linux/Windows endpoints communicating; restored lab access. |
| Oct 3–4 | Windows Server 2022, AD DS, agents | Domain services checked; Windows security telemetry appeared in Wazuh; snapshots and time settings addressed. |
| Oct 4–5 | Windows account and privileged-group testing | Security event 4720 and logon failures observed; `LabTestUser` Domain Admins change test captured IDs 4728 and 4729; Wazuh custom rule 100100, level 14. |
| Oct 5 | Wazuh alert email infrastructure | Postfix configured/tested for alert delivery; service status reviewed; Google app password used (secret not included). |
| Oct 5–8 | Separate Microsoft 365 Email Security Monitoring project | Personal Outlook mailbox, Microsoft Graph device-code auth, Python JSONL collector, Wazuh integration, rule 100500; deduplication tested; timer scheduled. |
| Oct 8 | Windows Server login and keyboard recovery | Regained domain Administrator session; Windows Server IP confirmed `.40`. |
| Oct 9 | Security portfolio / LinkedIn evidence | Wazuh privileged-group detection + alert-email screenshots selected for professional presentation. |
| Oct 10–11 | TP-Link Archer access point and home-lab network recovery | AP management login restored; config backup saved; white Internet LED; Windows client again on home-lab subnet; README troubleshooting notes updated. |

**Status distinction:** The Wazuh server at `<LAB_IP>` was active as of Oct 10. The Proxmox host's intended `<LAB_IP>` requires a fresh verification; it was previously unreachable. `<LAB_IP>` was formerly the iMac and is only *candidate free* pending DHCP mapping and conflict checks. Statuses are historical, not live network tests.

## 2. Network and device inventory

| Component | Address / detail | Notes |
|---|---|---|
| pfSense lab LAN | `<LAB_IP>` | Gateway; lab subnet `<LAB_IP>`. |
| TP-Link Archer AP | `<LAB_IP>` | AP mode; management access restored Oct 10. |
| Wazuh Server VM | `<LAB_IP>` | Active Oct 10. |
| Windows Server 2022 | `<LAB_IP>` | AD DS; confirmed Oct 8. |
| Primary Windows 11 laptop | `<LAB_IP>` on lab Wi-Fi | Also obtained `.117` in one Oct 10 wired test; verify interface before use. |
| Kali Linux VM | `<LAB_IP>` | Wazuh agent. |
| Former iMac reservation | `<LAB_IP>` | iMac removed; validate lease/static mapping before reuse. |
| Proxmox VE host | `<LAB_IP>` intended | Last reachability uncertain: check locally. |
| Ubuntu Server VM | `<LAB_IP>` | SSH/UFW and Wazuh agent. |
| Lab SSID | `LAB_WIFI` | Keep distinct from upstream Orbi Wi-Fi. |
| Upstream home Wi-Fi | Orbi | Separate upstream network. Do not assume the client is on lab Wi-Fi. |

### Windows — network diagnostics [REFERENCE, with commands previously used]
```powershell
ipconfig /all
ipconfig
ping <LAB_IP>
ping <LAB_IP>
ping <LAB_IP>
ping <LAB_IP>
arp -a
tracert <LAB_IP>
Get-NetIPConfiguration
Get-NetAdapter
Get-NetConnectionProfile
Test-NetConnection <LAB_IP> -Port 443
```
**Interpretation:** First confirm the laptop is on `192.168.10.x`, gateway `<LAB_IP>`, and the intended SSID. Being connected to Orbi rather than `LAB_WIFI` can explain failed access to lab devices. `ping` failure alone does not prove the host is down because ICMP might be blocked.

### Linux — network diagnostics [REFERENCE]
```bash
ip -br address
ip route
ip link show
ping -c 4 <LAB_IP>
ping -c 4 <LAB_IP>
ip neigh
ss -tulpen
resolvectl status
sudo journalctl -b -p warning --no-pager
```

## 3. Proxmox VE — host and VM operations

### Host checks [REFERENCE]
```bash
pveversion
hostname -I
ip -br address
ip route
cat /etc/network/interfaces
ip link show vmbr0
pvesm status
pvesh get /nodes/$(hostname)/status
qm list
```
`qm` is run **on the Proxmox host**, not on Ubuntu/Kali guests.

### Bridge repair [VERIFIED history; CAUTION: live network modification]
These commands were documented during recovery of the Plugable USB 2.5 GbE NIC; the interface name is machine-specific:
```bash
ip link set enx8cae4cb99fee up
ip link set enx8cae4cb99fee master vmbr0
ifreload -a
```
**Before repeating:** confirm the NIC name with `ip -br link`, confirm bridge membership with `bridge link`, and have console access. Reapplying a network configuration remotely can disconnect Proxmox.

### VM lifecycle [REFERENCE; CAUTION if stopping/rebooting]
```bash
qm list
qm status 100
qm config 100
qm start 100
qm shutdown 100
qm reboot 100
qm agent 100 ping
```
VM `100` was documented as `ubuntu-server` when originally created; check `qm list` because numeric IDs and guest-agent availability can change. Avoid forcibly stopping a domain controller or Wazuh during active writes.

### Proxmox web interface and guest troubleshooting
- In a browser: `https://<verified-proxmox-IP>:8006/`
- **NoVNC:** useful for console login; if it freezes, try SPICE console or reconnect.
- **SPICE:** used while fixing Windows Server keyboard/login.
- **Snapshots:** taken during key Windows Server and lab configuration stages. Confirm storage space before creating more.
- **Capacity note:** 16 GB host RAM was nearly saturated with Windows Server + Kali + Ubuntu running; avoid running every VM without checking memory.

## 4. pfSense firewall and TP-Link Archer AP

### pfSense — intended policy and recovery workflow
- Lab gateway `<LAB_IP>`; lab `<LAB_IP>`.
- Segment lab from upstream `<LAB_IP>` (earlier WAN design; confirm current upstream addressing).
- Specific lab policy documented: Ubuntu ICMP permission; Windows SSH permission; Kali SSH blocked by a rule.
- In pfSense, confirm **Interfaces**, **Firewall > Rules > LAN**, **Status > DHCP Leases**, and **Services > DHCP Server** for static mappings.
- Confirm the AP is **access point** (not router) mode and attached to the correct subnet; avoid accidental double NAT.
- Back up pfSense configuration before risky changes.

### Oct 10 AP recovery checklist [VERIFIED actions, no shell command needed]
1. Verify Windows is on lab SSID or correctly attached by Ethernet, rather than auto-switching to Orbi.
2. Verify `ipconfig /all` shows `192.168.10.x`, gateway `.1`.
3. Check ping/ARP reachability for AP `<LAB_IP>`.
4. Open the TP-Link management page; successful login was recovered.
5. Save a **configuration backup**; backup was saved.
6. Check client addressing, wireless internet access, and LEDs; AP Internet LED returned to solid white.
7. Document outcome in GitHub README.

**Beware:** `arp -a` showing an address is useful but does **not** guarantee the device's web interface is listening. Use browser/protocol tests separately.

## 5. Ubuntu / Kali — Linux administration

### System and service checks [REFERENCE]
```bash
hostnamectl
lsb_release -a
uname -a
free -h
df -h
lsblk
uptime
sudo apt update
sudo systemctl --failed
sudo journalctl -u ssh --no-pager -n 50
```
### SSH and firewall [REFERENCE; CAUTION on rule changes]
```bash
sudo systemctl status ssh --no-pager
sudo ufw status verbose
sudo ufw status numbered
ss -tln | grep ':22'
ssh <username>@<LAB_IP>
```
Do not enable or restrict UFW remotely unless you have explicitly allowed the management connection first.

## 6. Wazuh SIEM — server, agents, alerts, hunting

### Server health (run on Wazuh Server) [REFERENCE]
```bash
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
sudo systemctl status wazuh-logcollector --no-pager
sudo /var/ossec/bin/agent_control -l
sudo tail -n 80 /var/ossec/logs/ossec.log
sudo tail -n 80 /var/ossec/logs/alerts/alerts.json
sudo du -sh /var/ossec/queue/vd
df -h /
```
Some installations do not expose `wazuh-logcollector` as a standalone systemd unit; the logcollector is typically a component of the Wazuh manager. If `systemctl` says the unit was not found, inspect `wazuh-manager` status and `/var/ossec/logs/ossec.log` instead.

### Wazuh configuration paths [VERIFIED relevant files]
```bash
sudo nano /var/ossec/etc/ossec.conf
sudo nano /var/ossec/etc/rules/local_rules.xml
sudo /var/ossec/bin/wazuh-logtest
sudo systemctl restart wazuh-manager
```
`restart` is a **CAUTION** operation: perform after XML checks and preferably when the dashboard can tolerate a temporary interruption. The interactive `wazuh-logtest` allows pasting a sample event to check decoder and rule behavior.

### Windows agent status (Windows PowerShell as administrator) [VERIFIED commands]
```powershell
Get-Service WazuhSvc
NET START WazuhSvc
Get-Content 'C:\Program Files (x86)\ossec-agent\ossec.log' -Tail 50
```
Agent enrollment pointed to Wazuh manager `<LAB_IP>`. We saw Windows workstation `LAB_WORKSTATION`, Ubuntu, Kali, and Windows Server 2022 agents active during project work. Exact assigned agent IDs may change after reinstall/re-enrollment.

### Hunting in Wazuh Dashboard [VERIFIED detections]
- **Windows new local/domain user:** Event ID `4720`.
- **Failed sign-in:** Event ID `4625` (Windows raw log), Wazuh rules vary; lab observed failed-logon analysis and rule `60122` in earlier investigation.
- **Domain Admins member added:** Event ID `4728` for security-enabled global group.
- **Domain Admins member removed:** Event ID `4729`.
- **Custom privileged-group alert:** Wazuh rule `100100`, level `14`.
- **Email monitor custom notification:** Wazuh rule `100500`, level `8`.

Dashboard useful searches (field names depend on indexed event schema):
```text
rule.id:100100
rule.id:100500
win.system.eventID:4720
win.system.eventID:4728
win.system.eventID:4729
```
**Note:** The custom rule IDs `100100` and `100500` are from our own lab configuration, not universal built-in Wazuh rules.

## 7. Windows Server 2022 and Active Directory

### Domain and services [VERIFIED domain checks, REFERENCE commands]
```powershell
whoami
hostname
ipconfig /all
Get-Service NTDS
Get-ADDomain
Get-ADDomainController
Get-ADGroupMember -Identity 'Domain Admins'
Get-ADUser -Identity 'LabTestUser' -Properties Enabled,MemberOf
```
Run AD cmdlets on the domain controller with the AD PowerShell module and authorized account. The domain login `HOMELAB\Administrator` worked Oct 8; the lab evidence referenced `DC01.homelab.local`.

### Security event log investigation [REFERENCE]
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4720; StartTime=(Get-Date).AddDays(-2)} -MaxEvents 20
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4728,4729; StartTime=(Get-Date).AddDays(-7)} -MaxEvents 30
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625; StartTime=(Get-Date).AddDays(-2)} -MaxEvents 30
```
**Why:** `4720` account creation; `4728/4729` group membership; `4625` failed logon. Wazuh's ingestion pipeline should be checked against the Windows Security log when events are missing from a dashboard query.

### Controlled lab account test [VERIFIED test account, CAUTION: changes domain]
The `LabTestUser` account was created and used in a tightly controlled group-membership test. The commands below are **reproduction examples**; check first whether `LabTestUser` already exists.
```powershell
# Run ONLY in the isolated HOMELAB domain after checking account existence:
Get-ADUser -Filter 'SamAccountName -eq "LabTestUser"'
New-ADUser -Name 'LabTestUser' -SamAccountName 'LabTestUser' -AccountPassword (Read-Host -AsSecureString) -Enabled $true
Add-ADGroupMember -Identity 'Domain Admins' -Members 'LabTestUser'
Get-ADGroupMember -Identity 'Domain Admins'
Remove-ADGroupMember -Identity 'Domain Admins' -Members 'LabTestUser' -Confirm:$true
```
**Security:** Never leave a test principal in Domain Admins. Check for events `4728` and `4729` in Wazuh and use the least privilege needed for normal lab operation. `wazuhtest` was a prior temporary test account that was deleted; do not recreate it unless required for a new test.

### Login / console recovery [VERIFIED]
- Windows Server 2022 keyboard issues were isolated using Notepad and SPICE/noVNC console.
- Login eventually succeeded as `HOMELAB\Administrator`.
- Server address `<LAB_IP>` confirmed using `ipconfig /all`.
- Avoid password resets or repeated failed attempts before validating console keyboard behavior and account context.

## 8. Wazuh email alerts / Postfix

### Postfix status [VERIFIED]
```bash
sudo systemctl status postfix --no-pager
```
Oct 5 output reported `active (exited)` for the wrapper service; **that alone is not proof that outbound email delivery is working**. Inspect logs and actual receipt:
```bash
sudo journalctl -u postfix --no-pager -n 100
sudo postqueue -p
sudo postconf -n
```
**CAUTION:** `postconf -n` can reveal SMTP identities or configuration; redact credentials before sharing screenshots. A Gmail app password was created for the relay but is deliberately not recorded in this guide.

### Wazuh alert parameters [VERIFIED settings discussed]
```xml
<alerts>
  <log_alert_level>3</log_alert_level>
  <email_alert_level>12</email_alert_level>
</alerts>
```
A sample configuration in the project at one point showed `<email_notification>no</email_notification>` with placeholder SMTP values, *before* alert-delivery work. Do not assume that snapshot represents the final working configuration. Check current `ossec.conf`, Postfix, and inbox delivery before asserting email notifications are operational.

## 9. Microsoft 365 Email Security Monitoring (separate project)

**Architecture [VERIFIED]:** Personal Outlook mailbox → Microsoft Graph delegated `Mail.Read`, `User.Read` → MSAL device-code login (`/consumers`) → Python monitor → JSONL file → Wazuh local logcollector → decoder/custom rule `100500` → Threat Hunting. This is **not** the enterprise Microsoft 365 Office 365 audit-log integration.

### Files and locations [VERIFIED]
```text
/home/USERNAME/m365_monitor.py
/home/USERNAME/m365_email_events.jsonl
/home/USERNAME/.m365_seen_ids.json
/home/USERNAME/.m365_token_cache.bin
/home/USERNAME/.m365_monitor.conf
/home/USERNAME/m365-monitor-venv/bin/python
/etc/systemd/system/m365-email-monitor.service
/etc/systemd/system/m365-email-monitor.timer
```
Some initial authentication setup also used `~/m365_device_auth.py`. No application secret is embedded in the guide.

### Python setup and protected files [REFERENCE based on verified stack]
```bash
python3.12 --version
python3.12 -m venv ~/m365-monitor-venv
source ~/m365-monitor-venv/bin/activate
python -m pip show msal requests
ls -l ~/.m365_seen_ids.json ~/.m365_token_cache.bin ~/.m365_monitor.conf
chmod 600 ~/.m365_seen_ids.json ~/.m365_token_cache.bin ~/.m365_monitor.conf
```
Permissions require ownership of the files. Never paste contents of token caches or app configuration into public GitHub/LinkedIn posts.

### Authentication configuration [VERIFIED]
```text
Authority: https://login.microsoftonline.com/consumers
Delegated permissions: Mail.Read, User.Read
Flow: device-code authentication (MSAL Python)
Token cache: ~/.m365_token_cache.bin
Client ID config: ~/.m365_monitor.conf
```
`offline_access` was removed from the explicit requested scope list after a reserved-scope error. MSAL may handle standard OpenID scopes internally.

### systemd service [VERIFIED configuration]
```ini
[Service]
ExecStart=/home/USERNAME/m365-monitor-venv/bin/python /home/USERNAME/m365_monitor.py
```
This is a **relevant excerpt**, not a full replacement for the service file; keep the existing `[Unit]`, user, working directory, and other service settings intact.

### Five-minute timer [VERIFIED configuration]
```ini
[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
Persistent=true
```
The timer was enabled with:
```bash
sudo systemctl enable --now m365-email-monitor.timer
```
### Inspect the running pipeline [VERIFIED names; REFERENCE commands]
```bash
sudo systemctl status m365-email-monitor.timer --no-pager
sudo systemctl status m365-email-monitor.service --no-pager
systemctl list-timers --all | grep m365
sudo journalctl -u m365-email-monitor.service -n 80 --no-pager
wc -l /home/USERNAME/m365_email_events.jsonl
tail -n 3 /home/USERNAME/m365_email_events.jsonl
stat /home/USERNAME/.m365_seen_ids.json
```
**Result achieved:** The JSONL feed was parsed into Wazuh and rule `100500` matched. Deduplication was checked with `wc -l`: file remained **51 lines → 51 lines** on an unchanged fetch. The state file had `600` permissions. Wazuh manager-generated local logs may appear under agent ID `000`.

### Wazuh JSON integration [VERIFIED concept]
The Wazuh logcollector tails `/home/USERNAME/m365_email_events.jsonl` with a JSON decoder, and the lab's custom detection rule is `100500`. Check the existing Wazuh `ossec.conf` and `local_rules.xml` for the precise active stanza rather than replacing them from a simplified cheat-sheet snippet.

## 10. Security operations quick-reference

| Need | Device | Command or place to start |
|---|---|---|
| Am I on the correct network? | Windows | `ipconfig /all`; inspect Wi-Fi SSID |
| Can I reach pfSense? | Windows/Linux | `ping <LAB_IP>` |
| Can I reach Wazuh? | Windows/Linux | `ping <LAB_IP>` then browser/service check |
| Is Proxmox's IP correct? | Proxmox console | `ip -br address`, `ip route`, `qm list` |
| Is Proxmox bridge attached? | Proxmox console | `bridge link`, `ip link show vmbr0` |
| Is Wazuh alive? | Wazuh | `sudo systemctl status wazuh-manager --no-pager` |
| Are agents enrolled? | Wazuh | `sudo /var/ossec/bin/agent_control -l` |
| Is Windows agent running? | Windows | `Get-Service WazuhSvc` |
| Check domain users | DC PowerShell | `Get-ADUser -Identity 'LabTestUser'` |
| See group changes | DC PowerShell | `Get-WinEvent -FilterHashtable @{LogName='Security';Id=4728,4729}` |
| Check Outlook monitor timer | Wazuh | `systemctl list-timers --all \| grep m365` |
| Check JSONL deduplication | Wazuh | `wc -l /home/USERNAME/m365_email_events.jsonl` |
| Find specific alerts | Wazuh UI | `rule.id:100100` or `rule.id:100500` |
| Check relay queue | Wazuh | `sudo postqueue -p` |

## 11. GitHub, Cloudflare portfolio, and evidence

**Verified public project locations (historically supplied):**
- Portfolio: https://cybersecurity-portfolio-bcu.pages.dev/
- GitHub: https://github.com/shivansh2589/cybersecurity-home-lab

**Strong evidence to preserve:**
- Proxmox topology and bridge recovery notes, before/after networking tests.
- pfSense segmented LAN policies; Oct 10 AP recovery, configuration backup and solid-white LED.
- Wazuh agent status across Linux and Windows endpoints.
- Wazuh Threat Hunting: Rule **100100**, severity **14**, Domain Admins membership change.
- Email-alert receipt screenshot (ensure no sensitive recipients are exposed).
- Outlook/Microsoft Graph JSONL collector, five-minute systemd timer, and Wazuh Rule **100500**; redact mail subjects and sensitive identifiers in public images.
- Windows event evidence `4720`, `4728`, `4729`, `4625`.

### Git commands [REFERENCE; run inside your local cloned repo]
```bash
git status
git pull --ff-only
git add README.md
git commit -m 'Document October home lab recovery and security monitoring'
git push
```
Do **not** commit `.m365_token_cache.bin`, `.m365_monitor.conf`, `.m365_seen_ids.json`, mail contents, certificates, private keys, or passwords. Prefer sanitized screenshots and redacted snippets.

## 12. Next verification checklist

- [ ] From the Proxmox physical console, verify current host IP and USB NIC / `vmbr0` status (planned `.80` unconfirmed).
- [ ] Check pfSense DHCP leases and static mappings before reusing `.70`.
- [ ] Confirm Windows laptop sticks to the intended lab network rather than Orbi.
- [ ] Check Wazuh manager/indexer/dashboard and four agents.
- [ ] Check AD `Domain Admins` membership and remove any lingering experimental admin permissions.
- [ ] Confirm actual email delivery, not just Postfix service state.
- [ ] Confirm `m365-email-monitor.timer` active, expected run interval and JSONL deduplication.
- [ ] Redact screenshots and keep GitHub README / portfolio synchronized.

---
*This is an October 1–11 historical runbook reconstructed from our documented work, not an automated inventory of current running systems. Some commands are reference examples, not a verbatim exhaustive terminal history.*