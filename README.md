# Contract-Grade AD Security Lab (2025)

**Stack:** pfSense (firewall+VPN), Windows Server (AD/DNS/DHCP), Windows 10 client, Ubuntu client, Kali attacker.  
**Proof-first:** every claim backed by packets, logs, and screenshots.

---

## 📌 Why this lab matters
- Networking: proved with DHCP DORA, DNS resolution, and pfSense firewall control.  
- Identity: built a real AD forest with OUs, users, groups, and group-based access.  
- Authentication: captured Kerberos tickets (TGT/TGS) + Event IDs (4768/4769/4624).  
- Remote Access: pfSense OpenVPN into the lab, verified with share access.  
- Visibility: deployed Sysmon + auditing, showed brute-force detection (4625).  


---

# 📂 Architecture
![Diagram](docs/diagram_lab_topology.png)

- pfSense router/NAT/VPN (LAN `192.168.100.0/24`, gateway `.1`)
- Windows Server DC/DNS/DHCP (`192.168.100.10`)
- Windows 10 client (joined to domain)
- Ubuntu client (Linux fundamentals & DNS/route tests)
- Kali attacker (nmap/tcpdump brute-force)
- SMB share: `\\srv.lab.local\Finance$` (hidden, group-based permissions)

---

# 🔑 Proof of Work (Problem → Action → Solution)

## 1) DHCP – proving leases
**Problem:** New clients sometimes got APIPA (`169.254.x.x`).  
**Action:** Captured DHCP traffic during `ipconfig /renew`.  
**Solution:** Full DORA (Discover/Offer/Request/Ack) from DC.  
**Evidence:** ![DHCP DORA](screenshots/01_dhcp_dora_wireshark.png)

---

## 2) DNS – the glue of AD
**Problem:** `\\srv.lab.local\Finance$` failed without DNS.  
**Action:** Set client DNS to DC; ran `nslookup`.  
**Solution:** `srv.lab.local → 192.168.100.10`.  
**Evidence:** ![DNS](screenshots/02_dns_nslookup_srv_lab_local.png)

---

## 3) Firewall – pfSense actually blocks
**Problem:** Needed proof that perimeter rules worked.  
**Action:** Added top rule to block ICMP; pinged from client.  
**Solution:** pfSense logs show ICMP blocked/allowed by rule order.  
**Evidence:** ![Firewall](screenshots/03_pfsense_firewall_log_icmp_blocks.png)

---

## 4) Active Directory – structured OUs
**Problem:** GPOs don’t apply in default Computers container.  
**Action:** Created `Workstations`, `Users`, `Groups`; moved WIN10-CLI to Workstations.  
**Solution:** Clean AD structure, scoped policies apply.  
**Evidence:** ![ADUC](screenshots/04_aduc_ous_with_win10-cli_in_workstations.png)

---

## 5) Access Control – group-based SMB
**Problem:** Needed least-privilege proof on share.  
**Action:** NTFS + hidden share `Finance$`; Alice in `GG_Finance_RW`, Bob not.  
**Solution:** Alice allowed, Bob denied → group membership flips access.  
**Evidence:**  
- ![Alice access](screenshots/05_financeshare_alice_access.png)  
- ![Bob denied](screenshots/06_financeshare_bob_access_denied.png)

---

## 6) Kerberos – tickets are real
**Problem:** Must prove AD auth wasn’t NTLM.  
**Action:** Ran `klist` before/after accessing share; checked DC logs.  
**Solution:** TGT + TGS (`cifs/srv.lab.local`) issued; DC logs show 4768/4769/4624.  
**Evidence:**  
- ![klist before](screenshots/07.1_klist_before.png)  
- ![klist after](screenshots/07.2_win10-cli_klist_tgt_tgs.png)  
- ![Event Logs](screenshots/08_eventviewer_security_4768_4769_4624.png)

---

## 7) VPN – remote access works
**Problem:** Needed secure off-LAN access.  
**Action:** pfSense OpenVPN with push route to `192.168.100.0/24`.  
**Solution:** Tunnel connected; share only accessible with VPN up.  
**Evidence:** ![VPN](screenshots/09.1_pfsense_openvpn_status_connected.png)

---

## 8) Visibility – Sysmon + brute force
**Problem:** Native audit logs miss attacker noise.  
**Action:** Installed Sysmon (SwiftOnSecurity config) + ran Hydra brute force from Kali.  
**Solution:**  
- Sysmon Event ID 1 captured `notepad.exe` run by domain user.  
- DC logs showed wave of 4625 failed logons.  
**Evidence:**  
- ![Sysmon](screenshots/10_sysmon_eventid1_process_creation.png)  
- ![4625 failures](screenshots/11.1_eventviewer_security_4625_failed_logons.png)  
- ![More 4625s](screenshots/11.2_eventviewer_security_4625_failed_logons.png)

---

# 🧾 Lessons Learned
- **DNS is life**: break it, and Kerberos + shares collapse.  
- **Groups > Users**: access control scales only through AGDLP.  
- **Firewall rule order matters**: block above allow or it’s useless.  
- **Packets don’t lie**: DHCP DORA + Kerberos TGS are indisputable proof.  

---

# 📂 Repo structure

