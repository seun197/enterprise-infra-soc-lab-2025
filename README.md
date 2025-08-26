# Enterprise Infrastructure SOC Lab (2025)

**Firewall, Identity, VPN, Access Control, Visibility — designed, implemented, and proven in a controlled lab.**

This project simulates a corporate environment to demonstrate **end-to-end security engineering fundamentals**:  
designing, implementing, breaking/fixing, and proving the core controls a security engineer or SOC analyst depends on.

---

## 🌍 Architecture
![Topology](docs/diagram_lab_topology.png)

- **pfSense** → Firewall, NAT, VPN Gateway (`192.168.100.1`)  
- **SRV-CORE** → Windows Server 2019 DC/DNS/DHCP/File (`192.168.100.10`)  
- **WIN10-CLI** → Domain workstation + Wireshark endpoint  
- **UBU-CLI** → Linux client (network/DNS drills)  
- **KALI-ATT** → Attacker VM (nmap, brute-force, tcpdump)  
- **VPN** → OpenVPN (`10.8.0.0/24` → push route to 192.168.100.0/24)  

---

## 🎯 Security Objectives
1. **Perimeter Control** → pfSense firewall enforces rule order, logs blocked traffic.  
2. **Identity & Authentication** → Active Directory with Kerberos tickets validated at client + DC.  
3. **Access Control** → Group-based NTFS permissions on hidden share (`Finance$`).  
4. **Remote Access** → VPN tunnel provides controlled access to corporate subnet.  
5. **Visibility & Detection** → Sysmon telemetry, Windows audit policy, Wireshark captures.  

---

## 📊 Implementation Proofs

### 1. Networking – DHCP & DNS
- **Problem:** Clients fall back to APIPA (`169.254.x.x`) if DHCP/DNS fail.  
- **Action:** Captured **DORA handshake** in Wireshark; ran `nslookup srv.lab.local`.  
- **Solution:** Verified DC issued leases and resolved names correctly.  
- **Evidence:**  
  - `pcaps/dhcp_dora_wireshark.pcapng`  
  - `screenshots/dns_nslookup_srv_lab_local.png`

### 2. Perimeter Security – Firewall
- **Problem:** Needed to prove firewall enforcement beyond defaults.  
- **Action:** Added pfSense rule to **block ICMP** above allow-any.  
- **Solution:** Client pings dropped; pfSense firewall log confirmed.  
- **Evidence:** `screenshots/pfsense_firewall_log_icmp_blocks.png`

### 3. Identity – Active Directory OUs
- **Problem:** Group Policy doesn’t apply in default *Computers* container.  
- **Action:** Created **Workstations OU**, moved WIN10-CLI object.  
- **Solution:** Policies scoped cleanly for workstations.  
- **Evidence:** `screenshots/aduc_ous_with_win10-cli_in_workstations.png`

### 4. Access Control – Group-based NTFS
- **Problem:** Finance share required least privilege, auditable access.  
- **Action:** Created hidden share `Finance$`; applied **GG_Finance_RW** group permissions.  
- **Solution:**  
  - Alice (in group) → write access  
  - Bob (not in group) → denied  
  - Bob (added later) → access granted  
- **Evidence:**  
  - `screenshots/financeshare_alice_access.png`  
  - `screenshots/financeshare_bob_access_denied.png`  
  - `screenshots/financeshare_bob_access_after_group_add.png`

### 5. Authentication – Kerberos
- **Problem:** Needed proof of Kerberos, not NTLM.  
- **Action:** On client → ran `klist` before/after accessing share. On DC → filtered logs.  
- **Solution:**  
  - Client showed **TGT + TGS (`cifs/srv.lab.local`)**  
  - DC logged **4768/4769/4624** events  
- **Evidence:**  
  - `screenshots/win10-cli_klist_tgt_tgs.png`  
  - `screenshots/eventviewer_security_4768_4769_4624.png`

### 6. Remote Access – VPN
- **Problem:** Off-LAN users needed secure route into corporate subnet.  
- **Action:** Configured pfSense OpenVPN (tun/Wintun, pushed LAN route).  
- **Solution:** Share only reachable while VPN connected.  
- **Evidence:** `screenshots/pfsense_openvpn_status_connected.png`

### 7. Visibility – Sysmon & Brute-Force
- **Problem:** Needed detection visibility on endpoints.  
- **Action:** Deployed **Sysmon**; simulated brute-force with Hydra.  
- **Solution:**  
  - Sysmon logged **Event ID 1 (process creation)**  
  - DC Security log filled with **4625 failed logons**  
- **Evidence:**  
  - `screenshots/sysmon_eventid1_process_creation.png`  
  - `screenshots/eventviewer_security_4625_failed_logons.png`

---

## 🧯 Break → Symptom → Fix
- DHCP off → APIPA lease → restart DHCP → renew = valid lease.  
- DNS mispointed → `srv.lab.local` fails → reset DNS to `192.168.100.10`.  
- No gateway → LAN OK, Internet dead → restore pfSense `.1`.  
- Kerberos fail → time skew >5 mins → resync NTP, fix DNS.  
- Firewall rule order → block below allow ineffective → move rule above allow-any.  

---

## 🛡️ Detection Scenarios
- **Baseline:** Normal Kerberos ticket flow captured in packets + logs.  
- **Attack:** Brute-force from Kali → 4625 storm generated.  
- **Response:** Correlated Sysmon Event ID 1 (powershell.exe) with failed logon events.  
- **Outcome:** Validated detection pipeline from endpoint → DC → firewall.  

---

## 📚 Lessons Learned
- DNS is the backbone of AD security.  
- Firewall rule **order** = security enforcement.  
- Group-based permissions (AGDLP) scale; per-user ACLs fail.  
- Packet capture validates what logs only imply.  
- Visibility (Sysmon + audit) is mandatory to catch brute-force noise.  

---

## 🎓 Skills Demonstrated
- **Identity & Access:** AD DS, Kerberos, NTFS, OUs, Groups  
- **Networking:** DHCP, DNS, NAT, routing, VPN  
- **Perimeter:** pfSense firewall, rule design, logging  
- **Endpoint Visibility:** Sysmon, Windows audit policies  
- **Detection Engineering:** Brute-force & Kerberos monitoring  
- **Troubleshooting:** Break/fix drills under pressure  

---

## ⚠️ Ethics
All offensive tools (nmap, brute-force) were executed only in an **isolated lab environment**.  
No production systems were targeted.

---

*© 2025 – Enterprise Infrastructure SOC Lab: proving design, implementation, and validation of security controls.*
