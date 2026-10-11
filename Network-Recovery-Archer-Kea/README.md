# pfSense Kea DHCP Migration and Archer BE230 Management Recovery

**Date:** October 10–11, 2026  
**Environment:** Isolated personal cybersecurity lab, pfSense CE 2.9.0, TP-Link Archer BE230 V1 (AP mode), Proxmox VE, Windows 11.

## Summary
Migrated pfSense's DHCP backend from end-of-life ISC DHCP to Kea DHCP and recovered the management interface of an Archer BE230 wireless access point **without factory resetting it**.

## Verified starting symptoms
- Archer continued broadcasting `Homelab-AP_5G` and `Homelab-AP_2.4G`, with a red Internet LED.
- Web management at `192.168.10.10` timed out from both Wi-Fi and a previously attempted Ethernet connection.
- Windows route, ping, ARP, TCP connectivity tests and Nmap showed that wireless connectivity and forwarding worked, but the management endpoint did not answer requests.
- Nmap reported filtered/no-response for TCP ports 80, 443 and 8080; the top-1,000-port scan did not find an open TCP port. The `-Pn` host-up message was an assumption, not proof of a response.

## pfSense migration and validation
1. Reviewed ISC-to-Kea compatibility: no DHCP failover or custom options in use, DNS Resolver registration of DHCP leases/static mappings disabled.
2. Saved pfSense configuration backup.
3. Switched DHCP backend to Kea; verified the Kea service was running.
4. Verified DHCP allocation/reuse events in system logs and restored wireless client's address `192.168.10.50`.
5. Tested connectivity to Wazuh (`192.168.10.30`), Windows Server 2022 (`192.168.10.40`) and Proxmox (`192.168.10.80`).
6. Removed stale DHCP reservations for the retired iMac (`.70`) and its old Ubuntu VM (`.20`), retaining six current static mappings.
7. Confirmed Proxmox uses a manually configured static address on `vmbr0`: `192.168.10.80/24`, gateway `192.168.10.1`.

## Root cause investigation: Archer management
The Archer was operating in **Access Point mode** but its local static management configuration was on **`192.168.0.2/24`**, with gateway and DNS set to `192.168.0.254`. Its DHCP Server option was **Auto**, and it supplied a `192.168.0.x` lease while isolated from the lab. Meanwhile, pfSense's DHCP mapping for Archer's MAC `68:7f:f0:18:5f:54` specified `192.168.10.10`. This inconsistent address configuration explained why the management page could not be reached at the expected lab IP.

## Troubleshooting and recovery
1. Verified Wi-Fi SSID/BSSID and signal with `netsh wlan show interfaces`; collected `ipconfig /all`, `arp -a`, `route print -4`, `Test-NetConnection` and Nmap evidence.
2. Isolated the Archer from the pfSense-connected switch; attached a Windows laptop by Ethernet to an Archer LAN port.
3. Enabled the Windows Ethernet adapter, disabled Wi-Fi to avoid confusing Orbi connectivity with the Archer connection.
4. Identified `192.168.0.2` via DHCP/ARP information and successfully opened its local web management page.
5. Confirmed Access Point mode. Set Archer DHCP Server to **Off** so pfSense remains the lab DHCP authority.
6. Set the Archer's static management address to `192.168.10.10/24`, gateway `192.168.10.1`, primary DNS `192.168.10.1`; retained secondary DNS `1.1.1.1`.
7. Reconnected Archer's blue Internet uplink to switch LAN 5 and rejoined `Homelab-AP_5G`.
8. Verified Windows received `192.168.10.50/24`, gateway `192.168.10.1`; TP-Link admin login loaded at `http://192.168.10.10`.
9. Internet LED changed to solid white. Successful ping to `1.1.1.1` and `nslookup google.com` confirmed network connectivity and DNS.
10. Exported a working Archer configuration backup.

## Outcome
**Recovered wireless AP management access and corrected the lab's DHCP ownership and management subnet without a factory reset.** Demonstrated systematic Layer 2/3 troubleshooting, ARP/MAC analysis, DHCP log interpretation, wired isolation testing, TCP port diagnostics, IP/gateway/DNS correction, change verification and backup discipline.

## Portfolio-ready achievement
Diagnosed and recovered a TP-Link Archer BE230 access point whose management interface was unreachable despite functional Wi-Fi. Isolated inconsistent static IP/gateway/DNS settings and DHCP behavior, restored centralized pfSense Kea DHCP operation, and validated browser management, wireless connectivity, Internet access and DNS—without a factory reset.

*Scope note:* All testing took place on equipment owned and administered within a personal home lab.
