# 🛡️ Home SOC Lab — Attack Simulation & Threat Detection

> A hands-on cybersecurity home lab simulating real-world attacks and detecting them with a SIEM, built to demonstrate practical SOC Analyst skills.

---

## 📌 Project Overview

This project involves building a 3-VM isolated lab environment where:
- An **attacker machine (Kali Linux)** launches simulated cyberattacks
- A **target machine (Ubuntu Server)** receives the attacks
- A **SIEM (Wazuh)** monitors, detects, and alerts on suspicious activity in real time

The goal is to replicate the core workflow of a SOC Analyst — from attack detection to incident documentation.

---

## 🧱 Lab Architecture

```
┌─────────────────────────────────────────────────────┐
│              VirtualBox Host-Only Network            │
│                  192.168.56.0/24                     │
│                                                      │
│  ┌──────────────┐      ┌──────────────────────────┐  │
│  │  Kali Linux  │─────▶│    target-ubuntu         │  │
│  │  (Attacker)  │      │    (Victim/Target)        │  │
│  │192.168.56.102│      │    192.168.56.10          │  │
│  └──────────────┘      └────────────┬─────────────┘  │
│                                     │ Wazuh Agent     │
│                              ┌──────▼──────────────┐  │
│                              │   Wazuh Manager     │  │
│                              │   (SIEM/Detection)  │  │
│                              │   192.168.56.101    │  │
│                              └─────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

| VM | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | 192.168.56.102 | Kali Linux 2026.1 |
| target-ubuntu | Victim/Target | 192.168.56.10 | Ubuntu Server 26.04 LTS |
| Wazuh Manager | SIEM / Detection | 192.168.56.101 | Wazuh OVA v4.14.5 |

**Networking:** Each VM uses dual adapters — NAT (internet access) + Host-Only (isolated lab traffic). The host-only network allows the Windows host to access the Wazuh dashboard via browser without exposing the lab to external networks.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| VirtualBox | Hypervisor — running all 3 VMs |
| Wazuh v4.14.5 | SIEM — log collection, correlation, alerting |
| Kali Linux | Attacker platform |
| Ubuntu Server 26.04 | Target/victim machine |
| Hydra | SSH brute-force attack tool |
| Nmap | Network reconnaissance / port scanning |
| OpenSSH | Target service (port 22) |

---

## ⚔️ Attack Scenarios

### Scenario 1 — SSH Brute Force Attack

**Tool:** Hydra  
**Technique:** MITRE ATT&CK T1110.001 — Brute Force: Password Guessing  
**Tactic:** Credential Access (TA0006)

**What happened:**  
Hydra systematically tried thousands of passwords from the rockyou.txt wordlist against the SSH service on target-ubuntu. After hundreds of failed attempts, the correct password was discovered and the attacker gained unauthorized shell access.

**Command used:**
```bash
hydra -l testuser -P /usr/share/wordlists/rockyou.txt 192.168.56.10 ssh -t 4
```

**Result:** Password cracked (`abc123`), unauthorized SSH login achieved as `testuser`.

**Wazuh Detection:**
- Rule ID **40112** (Level 12 — Critical): *"Multiple authentication failures followed by a success"*
- **725 authentication failures** logged
- MITRE ATT&CK categories flagged: Password Guessing, SSH, Brute Force, Valid Accounts

---

### Scenario 2 — Nmap Network Reconnaissance Scan

**Tool:** Nmap  
**Technique:** MITRE ATT&CK T1046 — Network Service Discovery  
**Tactic:** Discovery (TA0007)

**What happened:**  
The attacker ran a service version detection scan to enumerate open ports and identify what software was running on the target machine. This reconnaissance was the first step before launching the brute force attack.

**Command used:**
```bash
nmap -sV 192.168.56.10
```

**Result discovered by attacker:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.2
```

**Wazuh Detection:**  
Connection and syslog events logged on target-ubuntu from source IP 192.168.56.102 during the scan window.

---

## 📊 Wazuh Detection Results

| Metric | Value |
|---|---|
| Total alerts generated | 1,109 |
| Authentication failures detected | 725 |
| Critical alerts (Level 12) | 2 |
| MITRE ATT&CK tactics detected | Credential Access, Discovery, Lateral Movement |
| Wazuh agent status | Active (Agent ID 001) |

---

## 📄 Incident Reports

Two formal incident reports were written for this project, following real SOC documentation standards:

- **[IR-001 — SSH Brute Force Attack](./incident-reports/IR-001-SSH-Brute-Force.md)**
- **[IR-002 — Nmap Network Reconnaissance](./incident-reports/IR-002-Nmap-Recon.md)**

Each report includes: incident summary table, attack timeline, detection method (rule ID + description), impact assessment, and recommended remediation steps.

---

## 🔧 Key Skills Demonstrated

- **SIEM Deployment** — Installed and configured Wazuh manager + enrolled a remote agent
- **Network Segmentation** — Designed isolated lab network with dual-adapter VMs
- **Attack Simulation** — Executed SSH brute force and network reconnaissance attacks
- **Log Analysis** — Interpreted Wazuh alerts, rule IDs, MITRE ATT&CK mappings
- **Incident Documentation** — Wrote professional incident reports in SOC analyst format
- **Linux Administration** — Static IP config (netplan), SSH hardening, user management, service management (systemctl)
- **Troubleshooting** — Diagnosed and resolved Hyper-V virtualization conflict, DNS resolution failure, SSH config issues

---

## 🗂️ Repository Structure

```
home-soc-lab/
├── README.md                          # This file
├── incident-reports/
│   ├── IR-001-SSH-Brute-Force.md      # Brute force incident report
│   └── IR-002-Nmap-Recon.md           # Reconnaissance incident report
└── screenshots/
    ├── wazuh-endpoints-active.png     # target-ubuntu showing Active in Wazuh
    ├── wazuh-threat-dashboard.png     # 725 auth failures + MITRE chart
    ├── wazuh-events-spike.png         # Alert spike during attack
    ├── wazuh-alert-detail.png         # Rule 40112 expanded alert details
    ├── hydra-attack-output.png        # Hydra cracking the password
    └── nmap-scan-output.png           # Nmap service discovery results
```

---

## 🚀 How to Replicate This Lab

1. Install VirtualBox on a Windows/Linux host (min. 16GB RAM recommended)
2. Download Wazuh OVA from https://packages.wazuh.com/4.x/vm/wazuh-4.14.5.ova and import into VirtualBox
3. Create Ubuntu Server VM — configure static IP via netplan, install OpenSSH
4. Create Kali Linux VM
5. Set all VMs to use Host-Only Adapter (192.168.56.0/24 range)
6. Install Wazuh agent on Ubuntu, enroll it to the Wazuh manager
7. Run attack simulations from Kali, monitor detections in Wazuh dashboard

---

## 👤 About

**Dharani Raj M**  
B.Tech Information Technology — Panimalar Engineering College (Anna University), 2026  
📧 itsdharani26@gmail.com  
🔗 [LinkedIn](https://www.linkedin.com/in/dharanirajm)  
🐙 [GitHub](https://github.com/Dharani26M)

**Certifications:**
- Cisco Introduction to Cybersecurity (Cisco Networking Academy)
- OverTheWire Bandit — Linux/command-line fundamentals

---

*This project was built as a portfolio demonstration of practical SOC analyst skills. All attacks were performed in an isolated lab environment with no connection to production systems or the public internet.*
