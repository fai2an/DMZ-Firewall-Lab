# DMZ-Firewall-Lab 

A hands-on home lab simulating a real-world enterprise network with firewall-enforced zone segmentation, a DMZ-hosted web server, and an SSH brute force attack simulation. Built to demonstrate core Network Engineering and Network Security skills.

---

## Project Overview

Most networking projects stop at "configure a firewall." This project goes further — it builds a complete attack scenario:

1. Design and deploy a segmented network with pfSense
2. Host a web server in an isolated DMZ zone
3. Launch a real brute force attack from the LAN
4. Verify firewall rules, capture traffic, and collect attack evidence

---

## Network Architecture

```
[Kali Linux - Attacker]          [Ubuntu - Web Server]
  192.168.1.101                    192.168.2.10
       |                                |
  LAN (VMnet2)                   DMZ (VMnet3)
  192.168.1.0/24                 192.168.2.0/24
       |                                |
       +----------[pfSense]------------+
                  Firewall
              192.168.1.1 (LAN)
              192.168.2.1 (DMZ)
                      |
                 WAN (VMnet1)
              192.168.157.x
                      |
                  Internet
```

| Zone | Subnet | Gateway | VM |
|------|--------|---------|-----|
| WAN | DHCP (VMnet1 NAT) | VMware NAT | pfSense em0 |
| LAN | 192.168.1.0/24 | 192.168.1.1 | Kali Linux |
| DMZ | 192.168.2.0/24 | 192.168.2.1 | Ubuntu Server |

---

## Tools & Technologies

| Tool | Purpose |
|------|---------|
| pfSense 2.7.2 | Firewall, Router, Network Segmentation |
| VMware Workstation | Virtualization Platform |
| Ubuntu Server 25.04 | DMZ Web Server |
| Kali Linux 2025.3 | Attacker Machine |
| Apache2 | Web Server hosted in DMZ |
| Nmap | Attacker Reconnaissance |
| Ncrack | SSH Brute Force Attack |
| pfSense Packet Capture | Traffic Analysis |

---

## Firewall Rules

### DMZ Rules

| # | Action | Protocol | Source | Destination | Port | Purpose |
|---|--------|----------|--------|-------------|------|---------|
| 1 | ✅ Pass | TCP | Any | DMZ subnets | 80, 443 | Allow HTTP/HTTPS into DMZ |
| 2 | ❌ Block | Any | DMZ subnets | LAN subnets | Any | Block DMZ to LAN (prevent lateral movement) |
| 3 | ✅ Pass | TCP | DMZ subnets | Any | 80, 443 | Allow DMZ web traffic out |

### LAN Rules

| # | Action | Protocol | Source | Destination | Port | Purpose |
|---|--------|----------|--------|-------------|------|---------|
| 1 | ✅ Pass | TCP | LAN subnets | DMZ subnets | 80 | Allow LAN to access DMZ web server |

---

## What I Did — Step by Step

### 1. Built the Virtual Network
- Created 3 VMware virtual networks: VMnet1 (NAT/WAN), VMnet2 (LAN), VMnet3 (DMZ)
- Deployed pfSense with 3 network adapters mapped to each zone
- Assigned static IPs to LAN (192.168.1.1) and DMZ (192.168.2.1) interfaces

### 2. Deployed Web Server in DMZ
- Installed Apache2 on Ubuntu Server
- Assigned static IP 192.168.2.10 to Ubuntu on VMnet3 (DMZ)
- Verified web server accessible from LAN through pfSense firewall

### 3. Wrote & Tested Firewall Rules
- Configured DMZ rules to allow inbound HTTP/HTTPS only
- Blocked all DMZ-to-LAN traffic to prevent lateral movement
- Verified rules using pfSense Packet Capture — confirmed one-way traffic only

### 4. Performed Attacker Reconnaissance
- Ran Nmap service version scan from Kali (LAN) against DMZ web server
- Discovered open ports: 22 (SSH), 80 (HTTP), 443 (HTTPS)
- Identified OS and service versions

### 5. Simulated SSH Brute Force Attack
- Launched Ncrack against SSH port 22 using Rockyou wordlist
- Captured attack evidence in Ubuntu auth.log
- Hundreds of failed authentication attempts logged from attacker IP 192.168.1.101

---

## Attack Evidence

### Nmap Reconnaissance Results
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.9p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.63 (Ubuntu)
443/tcp open ssl/https
Service Info: OS: Linux
```

### Auth Log — Brute Force Detected
```
sshd[1400]: drop connection #0 from [192.168.1.101]:57338
penalty: failed authentication
sshd[1400]: drop connection #0 from [192.168.1.101]:57340
penalty: failed authentication
sshd[1400]: drop connection #0 from [192.168.1.101]:57352
penalty: failed authentication
```

### pfSense Packet Capture — Traffic Analysis
```
192.168.1.101.57566 > 192.168.2.10.80: tcp 0
192.168.1.101.57566 > 192.168.2.10.80: tcp 0
```
Confirmed: Packets reaching DMZ interface. Firewall routing working correctly.

---

## 📸 Screenshots

| Screenshot | Description |
|------------|-------------|
| `screenshots/pfsense-dashboard.jpeg` | pfSense dashboard showing WAN/LAN/DMZ interfaces |
| `screenshots/firewall-rules-dmz.jpeg` | DMZ firewall rules configured |
| `screenshots/apache-page-kali.jpeg` | Apache default page accessed from Kali browser |
| `screenshots/nmap-scan.jpeg` | Nmap reconnaissance results from Kali |
| `screenshots/ncrack-running.jpeg` | SSH brute force attack in progress |
| `screenshots/auth-log-evidence.jpeg` | Ubuntu auth.log showing failed SSH attempts |
| `screenshots/packet-capture.jpeg` | pfSense packet capture showing traffic flow |

---

##  What I Learned

- How pfSense routes and filters traffic between network zones
- How to design a DMZ to isolate public-facing servers from internal LAN
- How firewall rules enforce security policies and prevent lateral movement
- How to read packet captures to diagnose network issues
- How attackers perform reconnaissance (Nmap) before launching attacks (Ncrack)
- Real-world troubleshooting — debugging no-carrier, routing failures, DHCP conflicts

---

## Planned Extension

This project will be extended by deploying a **Wazuh SIEM agent** on the DMZ Ubuntu server to enable real-time detection of the SSH brute force attack, mapping it to **MITRE ATT&CK T1110 (Brute Force)**. This will integrate with my existing [HomeSOCLab](https://github.com/fai2an/HomeSOCLab) project — creating a complete pipeline from network segmentation to SIEM alerting.

---

## Related Projects

- [HomeSOCLab](https://github.com/fai2an/HomeSOCLab) — Home SOC with Wazuh SIEM, brute force detection and MITRE ATT&CK mapping
- [NexaCorp-VAPT](https://github.com/fai2an/nexacorp-vapt) — Vulnerability Assessment and Penetration Testing report

---

## Lab Setup

| VM | Role | Network | IP | Specs |
|----|------|---------|-----|-------|
| pfSense 2.7.2 | Firewall/Router | VMnet1/2/3 | 192.168.1.1 / 192.168.2.1 | 1GB RAM, 20GB |
| Ubuntu Server 25.04 | DMZ Web Server | VMnet3 | 192.168.2.10 | 2GB RAM, 20GB |
| Kali Linux 2025.3 | Attacker | VMnet2 | 192.168.1.101 | 2GB RAM, 80GB |

All VMs run on VMware Workstation on a single Windows laptop.
