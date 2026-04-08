# 🔐 Wazuh Installation Guide (Complete Step-by-Step)

## 📌 Project Overview
This guide explains how to build a complete SOC home lab using:
- Ubuntu Server (Wazuh Manager)
- Windows Machine (Wazuh Agent)
- Kali Linux (Attacker)

---

## 🧱 Step 1: Install VirtualBox

Download:
https://www.virtualbox.org/wiki/Downloads

Install:
- Run installer
- Click Next → Install → Finish

---

## 💿 Step 2: Download ISO Files

Ubuntu Server:
https://ubuntu.com/download/server

Kali Linux:
https://www.kali.org/get-kali/#kali-installer

---

## 🖥️ Step 3: Setup Ubuntu Server

Create VM:
- Name: Wazuh-Server
- Type: Linux
- Version: Ubuntu (64-bit)

Resources:
- RAM: 4GB+
- Storage: 40GB

Install Ubuntu:
- Attach ISO
- Start VM
- Follow installation steps

Check IP:
```bash
ip a
```

Update system:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🧠 Step 4: Install Wazuh Server

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

```bash
sudo bash wazuh-install.sh -a
```

Access dashboard:
https://YOUR_SERVER_IP

---

## 💻 Step 5: Install Wazuh Agent (Windows)

Download:
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.x.x.msi

Install:
- Run installer
- Enter server IP

Start agent:
```bash
net start wazuh
```

---

## ⚔️ Step 6: Setup Kali Linux

Install Kali using ISO

Update:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🌐 Step 7: Network Setup

Set all VMs to:
- Bridged Adapter

Test:
```bash
ping YOUR_SERVER_IP
```

---

## ⚔️ Step 8: Attack Simulation

Manual brute force:
- Enter wrong password multiple times on Windows

Nmap scan:
```bash
nmap YOUR_SERVER_IP
```

---

## 🚨 Step 9: Detection

Search in Wazuh:
- failed login
- authentication failure

---

## 📸 Step 10: Screenshots

Capture:
- Dashboard
- Alerts
- Attack terminal
- Logs

---

## 🧠 Conclusion

This lab demonstrates:
- SIEM setup
- Attack simulation
- Threat detection
