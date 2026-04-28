# 🔍 Nmap Port Scan Attack — Complete Documentation

> **Project:** Wazuh SOC Home Lab
> **Attack Type:** Network Reconnaissance using Nmap
> **Attacker Machine:** Kali Linux (VM)
> **Victim Machine:** Windows Host (Wazuh Agent Installed)
> **SIEM:** Wazuh on Ubuntu Server (VM)
> **Objective:** Perform a real-world network reconnaissance attack using Nmap and detect it using Wazuh SIEM

---

## 📑 Table of Contents

1. [What is Nmap & Why Attackers Use It](#1-what-is-nmap--why-attackers-use-it)
2. [Lab Environment Overview](#2-lab-environment-overview)
3. [Pre-requisites & Tools](#3-pre-requisites--tools)
4. [Understanding Nmap Scan Types](#4-understanding-nmap-scan-types)
5. [Phase 1 — Find Your Network Range](#5-phase-1--find-your-network-range)
6. [Phase 2 — Discover Live Hosts](#6-phase-2--discover-live-hosts)
7. [Phase 3 — Basic Port Scan](#7-phase-3--basic-port-scan)
8. [Phase 4 — Service & Version Detection](#8-phase-4--service--version-detection)
9. [Phase 5 — OS Detection](#9-phase-5--os-detection)
10. [Phase 6 — Aggressive Full Scan (Final Attack)](#10-phase-6--aggressive-full-scan-final-attack)
11. [Phase 7 — Save Scan Results](#11-phase-7--save-scan-results)
12. [Phase 8 — Observe Wazuh Alerts](#12-phase-8--observe-wazuh-alerts)
13. [Phase 9 — Alert Analysis](#13-phase-9--alert-analysis)
14. [What Attacker Does With This Information](#14-what-attacker-does-with-this-information)
15. [Blue Team — Detection & Defense](#15-blue-team--detection--defense)
16. [MITRE ATT&CK Mapping](#16-mitre-attck-mapping)
17. [Key Takeaways](#17-key-takeaways)
18. [Screenshots](#18-screenshots)

---

## 1. What is Nmap & Why Attackers Use It

**Nmap (Network Mapper)** is a free, open-source tool used to discover hosts and services on a network. It works by sending specially crafted packets to target hosts and analyzing the responses.

### 🌍 Real-World Usage

In **every single real penetration test and cyber attack**, reconnaissance is the **very first step**. Before an attacker does anything — brute force, exploit, phishing — they first answer these questions:

> *"What machines are on this network? What ports are open? What services are running? What OS is the target using?"*

Nmap answers all of these questions. This is exactly why:
- It is installed by default on **Kali Linux**
- It is used by **ethical hackers, red teamers, and malicious attackers alike**
- Security teams specifically monitor for Nmap-like scan patterns

### 🔴 Why This is Dangerous

| Risk | Explanation |
|------|-------------|
| **Information Gathering** | Attacker learns exactly what services to target |
| **Attack Planning** | Open ports reveal potential entry points |
| **Stealth Scanning** | Advanced Nmap scans can bypass basic firewalls |
| **Zero Interaction** | Victim doesn't need to click anything |
| **Pre-Attack Step** | Our SMB Brute Force attack used Nmap data first |

### 🎯 In This Lab

In our previous attack (SMB Brute Force), we **first used Nmap** to find the Windows machine's IP and confirm Port 445 was open — **this is that documentation.**

---

## 2. Lab Environment Overview

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           HOME LAB NETWORK                                   ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║   ┌───────────────────────────┐          ┌───────────────────────────┐       ║
║   │      KALI LINUX  (VM)     │          │    UBUNTU SERVER  (VM)    │       ║
║   ├───────────────────────────┤          ├───────────────────────────┤       ║
║   │  Role  : Attacker         │          │  Role  : SIEM  (Wazuh)    │       ║
║   │  Tool  : Nmap             │          │  Port  : 55000 / 1514     │       ║
║   │  Type  : Port Scanner     │          │  Dash  : HTTPS / 443      │       ║
║   └─────────────┬─────────────┘          └─────────────▲─────────────┘       ║
║                 │                                      │                     ║
║                 │  ====  Nmap Port Scan  ===>          │                     ║
║                 │     (Multiple Ports / Protocols)     │                     ║
║                 │                      Logs & Alerts   │                     ║
║                 │                          (via TLS)   │                     ║
║                 ▼                                      │                     ║
║   ┌─────────────────────────────────────────────────────────────────┐        ║
║   │                    WINDOWS HOST  (Victim)                       │        ║
║   ├─────────────────────────────────────────────────────────────────┤        ║
║   │   Role    :  Victim Machine                                     │        ║
║   │   Ports   :  All ports being scanned by Nmap                    │        ║
║   │   Agent   :  Wazuh Agent  -->  Forwarding Logs to SIEM          │        ║
║   └─────────────────────────────────────────────────────────────────┘        ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

| Component | Details |
|-----------|---------|
| 🔴 **Attacker** | Kali Linux (VirtualBox/VMware VM) |
| 🛡️ **SIEM** | Ubuntu Server + Wazuh Manager |
| 🖥️ **Victim** | Windows 10/11 Host Machine |
| 🔌 **Attack Method** | TCP/UDP Port Scanning |
| ⚔️ **Attack Tool** | Nmap 7.x |
| 📊 **Detection Tool** | Wazuh Dashboard |

---

## 3. Pre-requisites & Tools

### On Kali Linux (Attacker):

| Tool | Purpose | Pre-installed? |
|------|---------|----------------|
| `nmap` | Network scanner & port mapper | ✅ Yes (Kali built-in) |
| `terminal` | Command execution | ✅ Yes |

### Verify Nmap is Installed:
```bash
nmap --version
```

Expected Output:
```
Nmap version 7.94 ( https://nmap.org )
Platform: x86_64-pc-linux-gnu
Compiled with: liblua-5.4.4 libpcre2-10.42 ...
```

### On Windows (Victim):
- Wazuh Agent installed and connected to Wazuh Manager
- Windows Firewall ON (default) — Nmap still detects most things

### On Ubuntu Server (SIEM):
- Wazuh Manager running
- Wazuh Dashboard accessible via browser

### Network Requirement:
- Kali Linux and Windows must be on **same network** (or same VirtualBox/VMware Host-Only / NAT network)
- They should be able to ping each other

### Verify Connectivity from Kali:
```bash
ping <Windows-IP> -c 4
```

If ping replies come — you're ready to scan. ✅

---

## 4. Understanding Nmap Scan Types

Before jumping into commands, understand **what each scan does** — this knowledge is what impresses interviewers.

### Types of Scans We Will Use:

| Scan Type | Flag | How It Works | When Used |
|-----------|------|-------------|-----------|
| **Ping Scan** | `-sn` | Just checks if host is alive, no port scan | Find live hosts |
| **SYN Scan** | `-sS` | Sends SYN, doesn't complete handshake (stealth) | Most common, default |
| **TCP Connect** | `-sT` | Completes full TCP handshake | When SYN needs root |
| **UDP Scan** | `-sU` | Scans UDP ports | Slower, finds DNS/SNMP |
| **Version Scan** | `-sV` | Detects service versions | Know exact software |
| **OS Detection** | `-O` | Guesses operating system | Know the target OS |
| **Aggressive** | `-A` | All of above combined | Full reconnaissance |
| **Script Scan** | `-sC` | Runs default NSE scripts | Extra vulnerability info |

### Understanding Port States:

| State | Meaning |
|-------|---------|
| **open** | Port is accepting connections — service is running |
| **closed** | Port is accessible but no service is listening |
| **filtered** | Firewall is blocking — Nmap can't determine state |
| **open\|filtered** | Nmap can't tell if open or filtered |

---

## 5. Phase 1 — Find Your Network Range

### Step 1: Check Kali's IP Address

```bash
ip a
```

**Sample Output:**
```
1: lo: <LOOPBACK,UP,LOWER_UP>
   inet 127.0.0.1/8

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
   inet 192.168.1.10/24 brd 192.168.1.255
```

Note your IP — here it is `192.168.1.10`
This means your network range is `192.168.1.0/24`

> 📝 **Your network range = first 3 numbers of your IP + .0/24**
> Example: IP is `192.168.56.101` → Range is `192.168.56.0/24`

### Step 2: Confirm Network Interface

```bash
ifconfig
```

Or specifically:
```bash
ip route
```

Output shows your default gateway — confirms your subnet.

---

## 6. Phase 2 — Discover Live Hosts

Now find all devices that are online on your network.

### Step 3: Ping Scan (Host Discovery)

```bash
nmap -sn 192.168.1.0/24
```

**Flag Explanation:**
- `-sn` → "Scan No ports" — only ping to check who is alive

**Sample Output:**
```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 192.168.1.1
Host is up (0.001s latency).
MAC Address: AA:BB:CC:DD:EE:FF (Router)

Nmap scan report for 192.168.1.5
Host is up (0.003s latency).
MAC Address: 11:22:33:44:55:66 (Intel Corporate)

Nmap scan report for 192.168.1.10
Host is up (0.000s latency).

Nmap scan report for 192.168.1.15
Host is up (0.002s latency).

Nmap done: 256 IP addresses (4 hosts up) scanned in 2.45 seconds
```

**Identify Your Targets:**
- `192.168.1.1` → Router/Gateway (skip)
- `192.168.1.5` → **Windows Victim** (our target!)
- `192.168.1.10` → Kali itself
- `192.168.1.15` → Ubuntu Wazuh Server

> 📸 **Screenshot Opportunity #1:** Take screenshot of this output → `nmap-host-discovery.png`

---

## 7. Phase 3 — Basic Port Scan

Now that we have the target IP, start scanning its ports.

### Step 4: Default Port Scan (Top 1000 Ports)

```bash
nmap 192.168.1.5
```

This scans the **top 1000 most common ports** by default.

**Sample Output:**
```
Starting Nmap 7.94
Nmap scan report for 192.168.1.5
Host is up (0.0012s latency).
Not shown: 994 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5040/tcp  open  unknown
49668/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 1.42 seconds
```

**What This Tells Us:**

| Port | Service | Meaning |
|------|---------|---------|
| **135** | MSRPC | Windows Remote Procedure Call |
| **139** | NetBIOS | Old Windows file sharing |
| **445** | microsoft-ds | **SMB — our brute force target!** |
| **3389** | ms-wbt-server | **RDP — Remote Desktop Protocol** |

> 📸 **Screenshot Opportunity #2:** Take screenshot → `nmap-basic-scan.png`

### Step 5: Scan Specific Ports

```bash
nmap -p 22,80,135,139,443,445,3389 192.168.1.5
```

**Flag Explanation:**
- `-p` → Specify exact ports to scan (comma-separated)

### Step 6: Scan ALL 65535 Ports

```bash
nmap -p- 192.168.1.5
```

**Flag Explanation:**
- `-p-` → Scan ALL ports from 1 to 65535 (takes longer but thorough)

> ⚠️ This takes 5-15 minutes but finds hidden services on unusual ports.

---

## 8. Phase 4 — Service & Version Detection

Now we know what ports are open. Let's find **exactly which software and version** is running.

### Step 7: Version Detection Scan

```bash
nmap -sV 192.168.1.5
```

**Flag Explanation:**
- `-sV` → Service Version detection — probes open ports to identify software

**Sample Output:**
```
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 10 Pro microsoft-ds
3389/tcp  open  ssl/ms-wbt-server Microsoft Terminal Services
MAC Address: 11:22:33:44:55:66 (Intel)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

**What Attacker Learns:**
- Target is **Windows 10 Pro**
- **SMB is enabled** → perfect for brute force
- **RDP is open** → another attack vector
- Exact Microsoft services running

> 📸 **Screenshot Opportunity #3:** Take screenshot → `nmap-version-scan.png`

---

## 9. Phase 5 — OS Detection

### Step 8: Operating System Detection

```bash
sudo nmap -O 192.168.1.5
```

> ⚠️ **Must run as root/sudo** for OS detection

**Flag Explanation:**
- `-O` → OS fingerprinting — sends special probes to guess OS

**Sample Output:**
```
Running: Microsoft Windows 10
OS CPE: cpe:/o:microsoft:windows_10
OS details: Microsoft Windows 10 1909 - 21H2
Network Distance: 1 hop
```

**What Attacker Learns:**
- Exact Windows version → can look up specific CVEs (vulnerabilities)
- Network distance (1 hop = same subnet)

### Step 9: OS + Version Combined

```bash
sudo nmap -O -sV 192.168.1.5
```

This gives both OS and service version in one scan.

---

## 10. Phase 6 — Aggressive Full Scan (Final Attack)

This is the **complete reconnaissance scan** — everything in one command.

### Step 10: Aggressive Scan

```bash
sudo nmap -A 192.168.1.5
```

**What `-A` flag does (combines all of these):**

| Sub-flag | Function |
|----------|---------|
| `-sV` | Version detection |
| `-O` | OS detection |
| `-sC` | Default script scan |
| `--traceroute` | Network path tracing |

**Sample Output:**
```
Starting Nmap 7.94
Nmap scan report for 192.168.1.5
Host is up (0.0012s latency).
Not shown: 994 closed tcp ports

PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 10 Pro
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (DANGEROUS!)
|_smb2-time: 2026-04-25 12:30:01
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DESKTOP-VICTIM
| rdp-enum-encryption:
|   Security layer: RDP encryption
|_  RDP Encryption level: High
49668/tcp open  msrpc         Microsoft Windows RPC

OS details: Microsoft Windows 10 Pro 21H2
Network Distance: 1 hop
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows_10

Host script results:
|_clock-skew: mean: 0s, deviation: 0s
|_nbstat: NetBIOS name: DESKTOP-VICTIM, workgroup: WORKGROUP
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required

TRACEROUTE
HOP RTT     ADDRESS
1   1.21 ms 192.168.1.5

Nmap done: 1 IP address (1 host up) scanned in 15.43 seconds
```

**Goldmine of Information:**
- Machine name: `DESKTOP-VICTIM`
- SMB signing: **DISABLED** → vulnerable to relay attacks
- Windows 10 Pro 21H2 exact version
- RDP encryption level
- NetBIOS workgroup

> 📸 **Screenshot Opportunity #4:** This is your MAIN screenshot → `nmap-aggressive-scan.png`

### Step 11: Stealth SYN Scan (Advanced)

```bash
sudo nmap -sS -T2 192.168.1.5
```

**Flag Explanation:**
- `-sS` → SYN scan (half-open) — sends SYN but never completes handshake
- `-T2` → Timing template "Polite" — slower, harder to detect

**Why This Matters:**
- `-sS` doesn't complete TCP connection → harder to log
- `-T2` slows scan down → avoids rate-based detection
- Real attackers use this to stay under radar

**Timing Templates Explained:**

| Template | Flag | Speed | Detection Risk |
|----------|------|-------|---------------|
| Paranoid | `-T0` | Very Slow | Very Low |
| Sneaky | `-T1` | Slow | Low |
| Polite | `-T2` | Medium-Slow | Low-Medium |
| Normal | `-T3` | Normal | Medium (default) |
| Aggressive | `-T4` | Fast | High |
| Insane | `-T5` | Very Fast | Very High |

---

## 11. Phase 7 — Save Scan Results

Professional penetration testers always **save their results**. This also gives you files to reference in documentation.

### Step 12: Save in Normal Format

```bash
nmap -A 192.168.1.5 -oN nmap-results.txt
```

**Flag:** `-oN` → Output in Normal (human-readable) format

### Step 13: Save in All Formats

```bash
sudo nmap -A 192.168.1.5 -oA nmap-full-scan
```

**Flag:** `-oA` → Output in All formats simultaneously:
- `nmap-full-scan.nmap` → Normal text
- `nmap-full-scan.xml` → XML (for tools like Metasploit)
- `nmap-full-scan.gnmap` → Grepable format

### Step 14: View Results

```bash
cat nmap-results.txt
```

---

## 12. Phase 8 — Observe Wazuh Alerts

Switch to Wazuh Dashboard to see if the scan was detected.

### Step 15: Open Wazuh Dashboard

```
https://<Ubuntu-Server-IP>
```
Login → **Security Events**

### Step 16: Filter for Nmap Detection

Search bar mein type karo:
```
rule.description: *port scan*
```

Or:
```
rule.groups: recon
```

Or by rule ID:
```
rule.id: 40101
```

### Step 17: Check These Wazuh Rule IDs

| Rule ID | Description | Level |
|---------|-------------|-------|
| **40101** | Nmap port scan detected | 6 |
| **40111** | Multiple port scan attempts | 8 |
| **5706** | Windows port scan detected | 6 |

### Step 18: Time-Based Filter

Set time range to when you ran the Nmap scan.
```
Top Right → Time picker → Last 1 hour
```

> 📸 **Screenshot Opportunity #5:** Wazuh showing port scan alerts → `wazuh-nmap-alert.png`

> 📝 **Note:** Wazuh may not always detect Nmap depending on:
> - Which scan type was used (stealth vs aggressive)
> - Whether Windows Firewall logged the connection attempts
> - Wazuh rules configuration

### Step 19: Check Windows Event Logs in Wazuh

Filter:
```
data.win.system.eventID: 5156
```
Event ID 5156 = Windows Filtering Platform allowed a connection — shows all incoming connection attempts including port scans.

---

## 13. Phase 9 — Alert Analysis

### What Wazuh Detected:

```
ALERT — Port Scan Detected
Source IP   : 192.168.1.10  (Kali Linux - Attacker)
Target      : Windows Agent  (192.168.1.5)
Rule ID     : 40101
Level       : 6
Time        : Apr 25, 2026 @ 12:36:00
Description : Possible port scan — multiple ports probed from single source
```

### Attack Timeline:

```
12:30:00  → Nmap aggressive scan started from Kali (192.168.1.10)
12:30:01  → Windows received SYN packets on ports 1-1000
12:30:01  → Windows Firewall logged connection attempts
12:30:02  → Wazuh Agent forwarded Windows logs to Wazuh Manager
12:30:03  → Wazuh rule 40101 triggered (port scan pattern detected)
12:30:03  → Alert appeared on Wazuh Dashboard
12:30:15  → Scan completed — 15 seconds total
12:30:15  → Full reconnaissance data collected by attacker
```

### Why Nmap Was Detected:

1. **Volume of connection attempts** — Hundreds of ports hit in seconds
2. **SYN without completion** — Half-open connections are suspicious
3. **Pattern matching** — Wazuh rule recognizes port scan signatures
4. **Source IP correlation** — Same IP hitting many ports

### What Attacker Got From This Scan:

```
Target IP      : 192.168.1.5
Hostname       : DESKTOP-VICTIM
OS             : Windows 10 Pro 21H2
Open Ports     : 135, 139, 445, 3389
Key Finding    : SMB (445) is OPEN → leads to brute force attack
Key Finding    : RDP (3389) is OPEN → another attack vector
Key Finding    : SMB signing DISABLED → vulnerable
```

---

## 14. What Attacker Does With This Information

> ⚠️ *This section is for educational understanding of attacker methodology only.*

After gathering this information, here's the attacker's next steps:

### Port 445 Open → SMB Brute Force
```bash
# Using information gathered from Nmap
hydra -L usernames.txt -P passwords.txt smb://192.168.1.5
```
*(This is exactly our second attack scenario!)*

### Port 3389 Open → RDP Brute Force
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.1.5
```

### SMB Signing Disabled → NTLM Relay Attack
```bash
# Intercept and relay Windows authentication
impacket-ntlmrelayx -t 192.168.1.5 -smb2support
```

### Metasploit with Nmap XML Output
```bash
msfconsole
msf> db_import nmap-full-scan.xml
msf> hosts          # See all discovered hosts
msf> services       # See all discovered services
```

This is how Nmap feeds directly into the next phase of attack.

---

## 15. Blue Team — Detection & Defense

As a SOC Analyst, here's how to defend against Nmap reconnaissance:

### 🛡️ Detection Methods

| Method | Implementation |
|--------|---------------|
| **IDS/IPS Rules** | Snort/Suricata rule: alert if 100+ ports hit from same IP in 10s |
| **Wazuh Active Response** | Auto-block scanning IP using firewall-drop |
| **Windows Firewall Logging** | Enable connection logging for all inbound
