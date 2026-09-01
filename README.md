# Network Intrusion Detection System using Wazuh & Suricata

## Overview

This project documents the design and implementation of a Network Intrusion Detection System (NIDS) using Wazuh and Suricata in a virtual cybersecurity lab built with VirtualBox.

The objective was to simulate a Security Operations Center (SOC) environment capable of monitoring endpoint activity, detecting suspicious network traffic, and visualizing security alerts through the Wazuh Dashboard.

Unlike a traditional SIEM-only deployment, this project integrates Suricata IDS with Wazuh to provide real-time intrusion detection and centralized security monitoring.

This project was completed as part of the CodeAlpha Cybersecurity Internship.

## Project Objectives
- Detect suspicious or malicious network activity in real time.
- Integrate Suricata alerts into Wazuh.
- Configure and tune detection rules for accurate alerting.
- Automatically respond to detected intrusions (active response).
- Visualize alerts and attack activity on a centralized dashboard.
  
## Lab Architecture

### Environment
| Virtual Machine	    | Role |
|------------------|----------------------------|
| **Kali VM** | Runs Suricata (NIDS engine) + Wazuh Agent — monitored/defending host |
| **Ubuntu VM** | Runs Wazuh Manager + Dashboard — central alerting & response engine; also used as the "attacker" machine for testing |
| VirtualBox    |	Virtual Lab Environment | 

**Data flow:**
Suricata (Kali) → detects traffic → writes alerts to eve.json
↓
Wazuh Agent (Kali) → forwards alerts to Wazuh Manager
↓
Wazuh Manager (Ubuntu) → applies custom detection rules → correlates alerts
↓
Active Response → automatically blocks malicious source IP (firewall-drop)
↓
Wazuh Dashboard → visualizes alerts, severity, and response actions

## Tools Used
| Tool	              | Purpose |
|------------------|----------------------------|
| Wazuh	 (Manager + Agent)  | SIEM, log correlation, active response, dashboard
| Suricata (v8.0.6)	|Network Intrusion Detection System (NIDS) engine 
| Ubuntu	| Wazuh Manager host, also used to simulate attacker traffic
| Kali Linux	| Wazuh endpoint & monitored host running Suricata
| Nmap | Used to simulate port scanning / suspicious activity
| VirtualBox	| Lab Virtualization
| Linux CLI	| Installation & Configuration

## Skills Demonstrated
- SIEM Deployment
- Endpoint Monitoring
- Network Intrusion Detection
- Linux Administration
- Threat Hunting
- Log Analysis
- IDS Rule Configuration

## 📂 Repository Structure
