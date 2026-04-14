# 🏗️ SOC Home Lab — Architecture

> **A complete visual breakdown of the Wazuh SOC Home Lab — how each machine is connected, what role it plays, and how attacks are detected in real-time.**

---

## 📌 Lab at a Glance

| Component        | OS                     | Role                      |
|------------------|------------------------|---------------------------|
| 🐉 Kali Linux    | Kali Linux (Rolling)   | Attacker Machine          |
| 🪟 Windows       | Windows 10/11          | Victim + Wazuh Agent      |
| 🐧 Wazuh Server  | Ubuntu Server 22.04    | SIEM — Manager + Indexer  |
| 📊 Dashboard     | Runs on Ubuntu Server  | Alert Monitoring UI       |

---

## 🖼️ Lab Diagram

![SOC Lab Architecture](./lab-diagram.png)

---

## 🗺️ Full Architecture Diagram

```
+==============================================================================+
|                      🏠  SOC HOME LAB ENVIRONMENT                           |
+==============================================================================+
|                                                                              |
|  +---------------------------+        +----------------------------------+   |
|  |                           |        |                                  |   |
|  |   🐉  KALI LINUX          |        |   🪟  WINDOWS MACHINE            |   |
|  |   Role : Attacker         |        |   Role : Victim + Wazuh Agent    |   |
|  |   OS   : Kali Rolling     |        |   OS   : Windows 10/11           |   |
|  |                           |        |                                  |   |
|  |   Tools Used:             | -----> |   Wazuh Agent (Installed):       |   |
|  |   * Nmap  (Port Scan)     | ATTACK |   * Collects Windows Event Logs  |   |
|  |   * Metasploit            |        |   * File Integrity Monitoring    |   |
|  |   * Hydra (Brute Force)   |        |   * Registry Change Detection    |   |
|  |   * Netcat                |        |   * Real-time Log Forwarding     |   |
|  |   * Evil-WinRM            |        |   * Sysmon Log Collection        |   |
|  |                           |        |                                  |   |
|  +---------------------------+        +------------------+---------------+   |
|                                                          |                  |
|                                        Encrypted Log Forwarding             |
|                                         TCP Port 1514 / 1515                |
|                                                          |                  |
|                                                          v                  |
|                         +------------------------------------+              |
|                         |                                    |              |
|                         |  🐧  WAZUH SERVER (Ubuntu 22.04)   |              |
|                         |  Role : SIEM Core Security Engine  |              |
|                         |                                    |              |
|                         |  +------------------------------+  |              |
|                         |  |  Wazuh Manager               |  |              |
|                         |  |  * Receives logs from agents |  |              |
|                         |  |  * Applies detection rules   |  |              |
|                         |  |  * Triggers alerts           |  |              |
|                         |  +------------------------------+  |              |
|                         |                                    |              |
|                         |  +------------------------------+  |              |
|                         |  |  Wazuh Indexer (OpenSearch)  |  |              |
|                         |  |  * Stores all ingested logs  |  |              |
|                         |  |  * Enables fast search       |  |              |
|                         |  +------------------------------+  |              |
|                         |                                    |              |
|                         |  +------------------------------+  |              |
|                         |  |  Filebeat                    |  |              |
|                         |  |  * Ships logs to Indexer     |  |              |
|                         |  |  * Manager -> Indexer link   |  |              |
|                         |  +------------------------------+  |              |
|                         |                                    |              |
|                         |  Detects:                          |              |
|                         |  [+] Brute Force Attacks           |              |
|                         |  [+] Port Scanning & Recon         |              |
|                         |  [+] Privilege Escalation          |              |
|                         |  [+] Malware / Suspicious Process  |              |
|                         |  [+] Unauthorized File Changes     |              |
|                         |  [+] Failed Logins (EventID 4625)  |              |
|                         |                                    |              |
|                         +------------------+-----------------+              |
|                                            |                                |
|                                   HTTPS Port 443                            |
|                                            |                                |
|                                            v                                |
|                         +------------------------------------+              |
|                         |                                    |              |
|                         |  📊  WAZUH DASHBOARD               |              |
|                         |  Role : Monitoring & Alert Center  |              |
|                         |  URL  : https://<wazuh-server-ip>  |              |
|                         |                                    |              |
|                         |  * Real-time Alert Visualization   |              |
|                         |  * MITRE ATT&CK Framework Mapping  |              |
|                         |  * Threat Hunting & Investigation  |              |
|                         |  * Agent Health Monitoring         |              |
|                         |  * Compliance Dashboards           |              |
|                         |  * Incident Timeline View          |              |
|                         |                                    |              |
|                         +------------------------------------+              |
|                                                                              |
+==============================================================================+
```

---

## 🔄 Attack & Detection Flow

```
  STEP 1            STEP 2             STEP 3             STEP 4
+----------+      +----------+      +----------+      +----------+
|          |      |          |      |          |      |          |
| Attacker | ---> | Windows  | ---> |  Wazuh   | ---> |Dashboard |
|  Kali    |      | Machine  |      |  Server  |      |  Alert   |
|  Linux   |      | (Victim) |      | (Ubuntu) |      |Generated |
|          |      |          |      |          |      |          |
| Nmap     |      | Agent    |      | Rules    |      | SOC      |
| Hydra    |      | captures |      | match -> |      | Analyst  |
| Exploit  |      | the logs |      | Alert!   |      | responds |
|          |      |          |      |          |      |          |
+----------+      +----------+      +----------+      +----------+
```

---

## 🌐 Network Configuration

```
+---------------------------------------------+
|           NETWORK TOPOLOGY                  |
|                                             |
|   All VMs on same Host-Only / NAT Network   |
|                                             |
|   Kali Linux      -->  192.168.x.x          |
|   Windows Machine -->  192.168.x.x          |
|   Ubuntu (Wazuh)  -->  192.168.x.x          |
|                                             |
|   Key Ports:                                |
|   * 1514 / 1515  --> Agent <-> Manager      |
|   * 443          --> Dashboard (HTTPS)      |
|   * 9200         --> Wazuh Indexer          |
|   * 55000        --> Wazuh API              |
+---------------------------------------------+
```

---

## 🧩 Component Deep Dive

### 🐉 Kali Linux — Attacker Machine

- Acts as the **red team / threat actor** in this lab
- Runs simulated attacks to generate real security events

| Tool         | Purpose                           |
|--------------|-----------------------------------|
| `nmap`       | Port scanning & service discovery |
| `hydra`      | SSH / RDP brute force attacks     |
| `metasploit` | Exploitation framework            |
| `netcat`     | Reverse shell / listener          |
| `evil-winrm` | Windows remote management exploit |

---

### 🪟 Windows Machine — Victim + Wazuh Agent

- Acts as the **target / blue team endpoint**
- Wazuh Agent installed → sends logs to Wazuh Server

| Log Source              | What it captures                        |
|-------------------------|-----------------------------------------|
| Windows Event Logs      | Login attempts, process creation        |
| Sysmon                  | Deep process, network, file activity    |
| File Integrity Monitor  | Detects unauthorized file modifications |
| Registry Monitor        | Tracks registry changes by malware      |

---

### 🐧 Wazuh Server — Ubuntu 22.04 LTS (SIEM Core)

- The **brain of the SOC lab**
- Three core components running together:

| Component         | Function                                   |
|-------------------|--------------------------------------------|
| **Wazuh Manager** | Receives logs, applies rules, fires alerts |
| **Wazuh Indexer** | Stores and indexes all logs (OpenSearch)   |
| **Filebeat**      | Ships data between Manager and Indexer     |

---

### 📊 Wazuh Dashboard — Alert Monitoring UI

- Web-based UI accessed via browser
- Shows everything the SOC analyst needs:
  - 🔴 Critical / High / Medium / Low severity alerts
  - 🗺️ MITRE ATT&CK technique mapping
  - 📈 Event trend graphs
  - 🕵️ Agent status and health
  - 📋 Detailed log drill-down per alert

---

## 📁 Folder Structure

```
Architecture/
├── Architecture.md       <-- This file (you are here)
└── lab-diagram.png       <-- Visual PNG diagram
```

---

## 🎯 Purpose of This Lab

| Goal                   | Description                                       |
|------------------------|---------------------------------------------------|
| 🔵 Blue Team Skills    | Learn how a SIEM detects real attacks             |
| 🔴 Red Team Awareness  | Understand attacker techniques via logs           |
| 🧠 SOC Analyst Training| Practice reading alerts, investigating incidents  |
| 📋 Rule Writing        | Create custom Wazuh detection rules               |
| 🏆 Portfolio Project   | Showcase hands-on cybersecurity skills            |

---

<div align="center">

**Built with 🖥️ VirtualBox | 🐧 Ubuntu Server 22.04 | 🪟 Windows 10/11 | 🐉 Kali Linux | 🛡️ Wazuh SIEM**

</div>
