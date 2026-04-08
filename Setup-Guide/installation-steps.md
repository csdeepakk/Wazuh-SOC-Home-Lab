# 🛡️ Wazuh SOC Home Lab — Complete Installation Guide

> **Audience:** Beginners & cybersecurity students building their first SOC lab from scratch.  
> **Lab Goal:** Deploy a fully functional SIEM using Wazuh, simulate real attacks from Kali Linux, and detect them on the dashboard.

---

## 📋 Table of Contents

1. [Prerequisites & Lab Architecture](#-prerequisites--lab-architecture)
2. [Step 1 — Install VirtualBox](#-step-1--install-virtualbox)
3. [Step 2 — Download ISO Files](#-step-2--download-iso-files)
4. [Step 3 — Setup Ubuntu Server VM](#-step-3--setup-ubuntu-server-vm)
5. [Step 4 — Install Wazuh on Ubuntu Server](#-step-4--install-wazuh-on-ubuntu-server)
6. [Step 5 — Install Wazuh Agent on Windows](#-step-5--install-wazuh-agent-on-windows)
7. [Step 6 — Setup Kali Linux VM](#-step-6--setup-kali-linux-vm)
8. [Step 7 — Network Configuration](#-step-7--network-configuration)
9. [Step 8 — Attack Simulation](#-step-8--attack-simulation)
10. [Step 9 — Threat Detection in Wazuh](#-step-9--threat-detection-in-wazuh)
11. [Troubleshooting](#-troubleshooting)
12. [Conclusion](#-conclusion)

---

## 🗺️ Prerequisites & Lab Architecture

### What You'll Need

| Requirement | Minimum Spec |
|---|---|
| Host Machine RAM | 16 GB (recommended) |
| Host Machine Storage | 100 GB free |
| OS | Windows 10/11 or Linux (host) |
| Internet Connection | Required for downloads |

### Lab Topology

```
┌─────────────────────────────────────────────────┐
│                  VirtualBox Host                │
│                                                 │
│  ┌──────────────┐    ┌──────────────────────┐   │
│  │  Kali Linux  │    │   Ubuntu Server      │   │
│  │  (Attacker)  │───▶│   (Wazuh Manager)    │   │
│  │  192.168.x.x │    │   192.168.x.x        │   │
│  └──────────────┘    └──────────┬───────────┘   │
│                                 │               │
│                    ┌────────────▼────────────┐  │
│                    │  Windows Machine        │  │
│                    │  (Wazuh Agent)          │  │
│                    │  192.168.x.x            │  │
│                    └─────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### VM Summary

| VM Name | OS | Role | RAM | Storage |
|---|---|---|---|---|
| `Wazuh-Server` | Ubuntu Server 22.04 LTS | Wazuh Manager + Dashboard | 4–8 GB | 50 GB |
| `Windows-Agent` | Windows 10/11 | Wazuh Agent (victim) | 2–4 GB | 40 GB |
| `Kali-Attacker` | Kali Linux 2024.x | Attack machine | 2 GB | 30 GB |

---

## 📦 Step 1 — Install VirtualBox

### 1.1 Download VirtualBox

Go to: **https://www.virtualbox.org/wiki/Downloads**

Choose your host OS:
- **Windows:** `VirtualBox-x.x.x-Win.exe`
- **Linux (Ubuntu/Debian):** `.deb` package

### 1.2 Install on Windows

1. Run the downloaded `.exe` file
2. Click **Next** on the welcome screen
3. Leave default installation path → Click **Next**
4. On "Warning: Network Interfaces" popup → Click **Yes** (your internet will disconnect briefly)
5. Click **Install**
6. If prompted by Windows UAC → Click **Yes**
7. Click **Finish** — VirtualBox will launch automatically

### 1.3 Install VirtualBox Extension Pack (Recommended)

Download the Extension Pack from the same downloads page.

```
VirtualBox → Tools → Extensions → Click the [+] icon → Select downloaded .vbox-extpack → Install
```

> **Why?** Enables USB 2.0/3.0, RDP, and better VM performance.

---

## 💿 Step 2 — Download ISO Files

Download all three ISO files before creating VMs to save time.

| OS | Download Link | File Size |
|---|---|---|
| Ubuntu Server 22.04 LTS | https://ubuntu.com/download/server | ~2.0 GB |
| Kali Linux (Installer) | https://www.kali.org/get-kali/#kali-installer | ~4.0 GB |
| Windows 10 ISO | https://www.microsoft.com/software-download/windows10 | ~5.0 GB |

> **Tip:** Save all ISOs in one folder like `C:\ISO-Files\` for easy access.

---

## 🖥️ Step 3 — Setup Ubuntu Server VM

### 3.1 Create the VM in VirtualBox

1. Open VirtualBox → Click **New**
2. Fill in the details:
   - **Name:** `Wazuh-Server`
   - **Folder:** Leave default or choose your preferred location
   - **ISO Image:** Browse and select your Ubuntu Server ISO
   - **Type:** Linux
   - **Version:** Ubuntu (64-bit)
3. Click **Next**

### 3.2 Allocate Resources

| Setting | Recommended Value |
|---|---|
| RAM (Base Memory) | 4096 MB (4 GB) minimum, 8192 MB preferred |
| CPU Processors | 2 cores minimum |
| Storage | 50 GB |

4. On **Hardware** screen → Set RAM to `4096` MB, CPUs to `2`
5. On **Hard Disk** screen → Select **Create a Virtual Hard Disk Now** → Set size to `50 GB`
6. Click **Finish**

### 3.3 Install Ubuntu Server

1. Select `Wazuh-Server` VM → Click **Start**
2. On the GRUB menu → Select **Try or Install Ubuntu Server** → Press Enter
3. **Language:** English → Press Enter
4. **Installer Update:** Choose **Continue without updating**
5. **Keyboard Layout:** Select your layout → Done
6. **Network:** Wait for it to auto-configure DHCP (you'll see an IP assigned) → Done
7. **Storage Configuration:**
   - Choose **Use an entire disk**
   - Leave defaults → Done → **Continue** (confirm destructive action)
8. **Profile Setup:**
   - Your name: `wazuh-admin` (or anything you like)
   - Server's name: `wazuh-server`
   - Username: `wazuh` (remember this!)
   - Password: Set a strong password (remember this!)
9. **SSH Setup:** Select **Install OpenSSH server** ✅ → Done
10. **Featured Snaps:** Skip → Done
11. Installation will begin — wait ~5–10 minutes
12. When done → **Reboot Now**

### 3.4 First Boot Configuration

Login with your username and password, then:

**Check your IP address:**
```bash
ip a
```
Look for an IP under `enp0s3` or `eth0` — write this down (e.g., `192.168.1.105`).

**Update the system:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Install essential tools:**
```bash
sudo apt install curl wget net-tools -y
```

---

## 🧠 Step 4 — Install Wazuh on Ubuntu Server

> This installs the **Wazuh Manager**, **Indexer**, and **Dashboard** — all-in-one.

### 4.1 Download the Wazuh Installer

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

### 4.2 Run the All-in-One Installer

```bash
sudo bash wazuh-install.sh -a
```

**What this does:**
- Installs Wazuh Manager (receives agent data)
- Installs OpenSearch/Wazuh Indexer (stores logs)
- Installs Wazuh Dashboard (web UI)

> ⏳ This takes **10–20 minutes**. Do not interrupt it.

### 4.3 Save Your Credentials

When installation completes, you'll see output like:

```
INFO: --- Summary ---
INFO: You can access the web interface https://192.168.x.x
    User: admin
    Password: <RANDOM_PASSWORD>
```

**⚠️ Copy and save this password immediately!**

### 4.4 Access the Wazuh Dashboard

Open a browser on your host machine and go to:

```
https://YOUR_UBUNTU_SERVER_IP
```

- Accept the SSL certificate warning (click **Advanced → Proceed**)
- Login with `admin` and the password from above

You should now see the Wazuh Dashboard! 🎉

### 4.5 Verify All Services Are Running

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All three should show `active (running)`.

---

## 💻 Step 5 — Install Wazuh Agent on Windows

### 5.1 Create the Agent in Wazuh Dashboard

1. In the Wazuh Dashboard → Go to **Agents** → **Deploy New Agent**
2. Select **Windows**
3. Under **Server address**, enter your Ubuntu Server IP (e.g., `192.168.1.105`)
4. Copy the installation command shown (it includes your server IP and group)

### 5.2 Download the Wazuh Agent MSI

Go to: `https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.x.msi`

Or use the direct link from the Wazuh Dashboard.

### 5.3 Install the Agent

**Option A — GUI Install:**
1. Run the `.msi` file
2. Click **Next**
3. In **Manager IP** field → Enter your Ubuntu Server IP
4. Click **Install** → **Finish**

**Option B — PowerShell Install (paste the command from dashboard):**
```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.x.msi -OutFile wazuh-agent.msi
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER="192.168.x.x" WAZUH_REGISTRATION_SERVER="192.168.x.x"
```

### 5.4 Start the Wazuh Agent Service

Open **Command Prompt as Administrator**:

```cmd
net start wazuh
```

Or via PowerShell:
```powershell
Start-Service -Name wazuh
```

### 5.5 Verify Agent is Connected

In the Wazuh Dashboard → **Agents** → You should see your Windows machine listed as **Active** ✅

---

## ⚔️ Step 6 — Setup Kali Linux VM

### 6.1 Create the VM

1. VirtualBox → **New**
2. **Name:** `Kali-Attacker`
3. **ISO Image:** Select your Kali Linux ISO
4. **Type:** Linux | **Version:** Debian (64-bit)
5. **RAM:** 2048 MB | **CPU:** 2 | **Storage:** 30 GB
6. Click **Finish**

### 6.2 Install Kali Linux

1. Start the VM → Select **Graphical Install**
2. Language → **English**
3. Country → Select yours
4. Keyboard → Select yours
5. **Hostname:** `kali-attacker`
6. Leave domain blank
7. **Root Password:** Set a strong password
8. **New User:** Create a regular user account
9. **Partition Disk:** Choose **Guided – use entire disk** → Continue
10. **Software Selection:** Leave defaults (includes GUI + tools) → Continue
11. **GRUB:** Install on `/dev/sda` → Continue
12. Installation complete → **Continue** to reboot

### 6.3 Update Kali

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🌐 Step 7 — Network Configuration

All VMs must be on the same network to communicate.

### 7.1 Set Bridged Adapter on All VMs

For **each VM** (Wazuh-Server, Windows-Agent, Kali-Attacker):

1. VirtualBox → Select VM → **Settings** → **Network**
2. **Adapter 1:**
   - Enable Network Adapter: ✅
   - Attached to: **Bridged Adapter**
   - Name: Select your physical network card (WiFi or Ethernet)
3. Click **OK**

> **Why Bridged?** Each VM gets its own IP from your router, making them accessible to each other like real machines on the same network.

### 7.2 Find Each VM's IP

On Ubuntu/Kali:
```bash
ip a
```

On Windows:
```cmd
ipconfig
```

Write down all three IPs.

### 7.3 Test Connectivity

From Kali, ping the Wazuh Server:
```bash
ping 192.168.x.x    # Wazuh Server IP
```

From Kali, ping Windows:
```bash
ping 192.168.x.x    # Windows IP
```

All pings should succeed ✅

### 7.4 Open Required Ports on Ubuntu (if needed)

```bash
sudo ufw allow 1514/tcp    # Agent communication
sudo ufw allow 1515/tcp    # Agent registration
sudo ufw allow 443/tcp     # Dashboard (HTTPS)
sudo ufw allow 9200/tcp    # Wazuh Indexer
sudo ufw enable
```

---

## 🔴 Step 8 — Attack Simulation

Now you'll simulate real attacks that Wazuh should detect.

### 8.1 SSH Brute Force Attack (Kali → Wazuh Server)

**Using Hydra:**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.x.x
```

**Manual brute force (simpler):**
```bash
for i in {1..10}; do ssh wronguser@192.168.x.x; done
```

### 8.2 Failed Windows Login (Windows Machine)

On the Windows VM:
- Press `Win + L` to lock screen
- Try logging in with **wrong passwords** 5–10 times
- This triggers **Event ID 4625** (failed login) which Wazuh detects

### 8.3 Port Scan (Kali → Wazuh Server)

**Basic Nmap scan:**
```bash
nmap 192.168.x.x
```

**Aggressive scan (more detectable):**
```bash
nmap -A -T4 192.168.x.x
```

**SYN Stealth scan:**
```bash
sudo nmap -sS 192.168.x.x
```

### 8.4 Simulate Malware Activity (Windows)

Open **Command Prompt** on Windows and run:
```cmd
whoami
net user
net localgroup administrators
ipconfig /all
```

> These commands mimic post-exploitation enumeration and trigger Wazuh rules.

---

## 🚨 Step 9 — Threat Detection in Wazuh

### 9.1 View Active Alerts

1. Go to `https://YOUR_SERVER_IP`
2. Login → **Security Events** (or **Discover**)
3. Set time range to **Last 15 minutes**

### 9.2 Search for Specific Events

In the search bar, use these queries:

| Attack Type | Search Query |
|---|---|
| Failed SSH login | `rule.groups: authentication_failed` |
| Windows failed login | `rule.id: 60122` |
| Nmap port scan | `rule.groups: recon` |
| System enumeration | `data.win.eventdata.commandLine: whoami` |

### 9.3 Key Dashboard Sections

| Section | What to Look For |
|---|---|
| **Security Events** | All triggered alerts |
| **Threat Intelligence** | Malicious IPs/domains |
| **Integrity Monitoring** | File changes on endpoints |
| **Vulnerabilities** | CVEs on connected agents |

### 9.4 View Specific Alert Details

1. Click any alert row to expand it
2. Look at:
   - `rule.description` — what was detected
   - `rule.level` — severity (1–15, higher = critical)
   - `agent.name` — which machine triggered it
   - `data.srcip` — source IP of the attack

---

## 🔧 Troubleshooting

### Agent shows "Disconnected" in Dashboard

```bash
# On Ubuntu Server — restart manager
sudo systemctl restart wazuh-manager

# On Windows — restart agent
net stop wazuh
net start wazuh
```

### Wazuh Dashboard not loading

```bash
sudo systemctl status wazuh-dashboard
sudo systemctl restart wazuh-dashboard
```

### Can't ping between VMs

- Make sure all VMs are set to **Bridged Adapter**
- Disable Windows Firewall temporarily for testing:
  ```cmd
  netsh advfirewall set allprofiles state off
  ```

### Wazuh installation failed midway

```bash
# Remove partial install and retry
sudo bash wazuh-install.sh -a --overwrite
```

### Check Wazuh Manager logs

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

---

## ✅ Conclusion

You've successfully built a **complete SOC home lab**! Here's what you accomplished:

| ✅ Task | Details |
|---|---|
| SIEM Deployment | Wazuh Manager + Dashboard on Ubuntu Server |
| Endpoint Monitoring | Windows agent sending logs to Wazuh |
| Attack Simulation | Brute force, port scans, enumeration from Kali |
| Threat Detection | Real-time alerts visible in Wazuh dashboard |

### Skills Demonstrated
- Virtualization (VirtualBox)
- Linux server administration
- SIEM/SOC tool deployment (Wazuh)
- Network configuration
- Offensive security basics (Nmap, Hydra)
- Log analysis and alert investigation

---

## 📚 Resources

- [Wazuh Official Docs](https://documentation.wazuh.com)
- [Wazuh Community Forum](https://community.wazuh.com)
- [Kali Linux Tools](https://www.kali.org/tools/)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)

---

*Guide created for educational purposes — SOC Home Lab Project*
