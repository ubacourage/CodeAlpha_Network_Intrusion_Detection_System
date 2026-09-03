![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu) ![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-0265C0) ![Suricata](https://img.shields.io/badge/Suricata-IDS-red) ![VirtualBox](https://img.shields.io/badge/VirtualBox-Lab-183A61) ![Linux](https://img.shields.io/badge/Linux-Security-yellow?logo=linux) ![Status](https://img.shields.io/badge/Project-Completed-brightgreen)

# Network Intrusion Detection System (NIDS) with Wazuh and Suricata

## Overview

This project documents the design and implementation of a Network Intrusion Detection System (NIDS) using Wazuh and Suricata in a virtual cybersecurity lab built with VirtualBox.

The objective was to build a virtual Security Operations Center (SOC) environment capable of monitoring endpoint activity, detecting suspicious network traffic, correlating security events, and visualizing alerts through the Wazuh Dashboard.

Rather than deploying a standalone SIEM, this project integrates the Suricata Network Intrusion Detection System (NIDS) with Wazuh to provide centralized security monitoring and real-time intrusion detection.

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

Suricata (Kali)
        │
        ▼
Detects suspicious network traffic and writes alerts to `eve.json`
        │
        ▼
Wazuh Agent (Kali)
        │
        ▼
Forwards Suricata alerts to Wazuh Manager
        │
        ▼
Wazuh Manager (Ubuntu)
        │
        ▼
Correlates events and applies custom detection rules
        │
        ▼
Active Response (`firewall-drop`)
        │
        ▼
Wazuh Dashboard visualizes alerts and response actions



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
- Threat Detection & Alert Correlation
- Log Analysis
- IDS Rule Configuration
- Active Response Configuration

## 📂 Repository Structure
```text
├── README.md
├── config/
│   ├── manager/
│   │   ├── ossec.conf
│   │   └── local_rules.xml
│   └── agent/
│       └── ossec.conf
└── screenshots
```

## ⚙️ Implementation Steps

### 1. Ubuntu System Update & Wazuh Manager Installation
```bash
sudo apt update
sudo apt install curl
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
After installation, disabled the Wazuh repository to prevent unintended 
package updates:
```bash
sudo sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list && sudo apt update
```

### 2. Wazuh Agent Deployment (Kali)
- Installed and deployed the Wazuh Agent on Kali
- Registered the agent to the Wazuh Manager running on Ubuntu
- Confirmed agent connectivity from the Wazuh dashboard
  
### 3. Suricata Installation & Rule Configuration

**Installed Suricata via the official stable PPA:**
```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```

**Verified the installed version:**
```bash
suricata -V
```

**Installed curl (required for downloading rules):**
```bash
sudo apt install curl
```
**Downloaded and installed the Emerging Threats Open ruleset:**
```bash
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz
sudo mkdir -p /etc/suricata/rules
sudo mv rules/*.rules /etc/suricata/rules/
```

**Configured Suricata (`/etc/suricata/suricata.yaml`):**
```bash
sudo nano /etc/suricata/suricata.yaml
```

Key configuration changes made:
- Set HOME_NET to the IP address (or subnet) of the monitored network
  and left EXTERNAL_NET as any so Suricata could identify traffic originating
  outside the monitored environment.
- Set `default-rule-path` and confirmed `rule-files: - "*.rules"` so 
  Suricata loads all rule files from that path
- Confirmed the correct network interface for the Kali VM (eth0 in this lab environment)
  was configured for packet inspection.

- During configuration, also corrected a `default-rule-path` misconfiguration that
  initially pointed to the raw rules directory instead of the merged ruleset 
  generated by `suricata-update`

**Verified clean rule loading:**
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

### 4. Suricata–Wazuh Integration
- Configured the Wazuh Agent to ingest Suricata's `eve.json` alert log
- Confirmed end-to-end alert flow via the Wazuh dashboard

### 5. Custom Detection Rule
By default, all Suricata alerts are ingested under a single generic 
Wazuh rule (`86601`), regardless of severity. A custom rule was added 
([`config/manager/local_rules.xml`](./config/manager/local_rules.xml)) 
to specifically identify and escalate port-scan activity. The rule matches 
Suricata scan signatures (ET SCAN) and increases the alert severity
from the default Suricata rule to a higher-priority Wazuh alert:

```xml
<group name="local,syslog,suricata,">
  <rule id="100100" level="10">
    <if_sid>86601</if_sid>
    <field name="alert.signature">ET SCAN</field>
    <description>Suricata: Port scan detected - $(alert.signature)</description>
    <group>scan_detected,</group>
  </rule>
</group>
```

### 6. Active Response (Automated Intrusion Response)
Configured within the Wazuh Manager 
([`config/manager/ossec.conf`](./config/manager/ossec.conf)) to 
Automatically block the source IP address using Wazuh's built-in 
firewall-drop active response command for 600 seconds:

```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100100</rules_id>
  <timeout>600</timeout>
</active-response>
```

## 🧪 Testing & Results
- Simulated reconnaissance activity using `nmap -sS` from the Ubuntu 
  VM against the Kali VM
- Suricata detected multiple scan signatures, including suspicious 
  inbound traffic to MySQL, MSSQL, PostgreSQL, Oracle, and VNC ports
- The custom rule (`100100`) correctly escalated these detections to 
  level 10 alerts, distinguishing them from informational noise
- Active response (`firewall-drop`, rule `651`) triggered automatically, 
  blocking the attacking IP via `iptables`
-Verified that the attacking IP address was temporarily blocked — subsequent connection attempts 
  from the attacker IP failed — and that it auto-expired after the 
  configured 600-second timeout

## 📸 Screenshots
Step-by-step evidence of the setup and testing process is available in 
[`/screenshots`](./screenshots), including:
- Wazuh Manager and Agent installation
- Suricata installation and rule configuration
- Wazuh–Suricata integration confirmation
- ICMP ping and Nmap scan detection
- Active response (automated IP block) in action

## Key Learning Outcomes

Throughout this project, I learned how to:

- Deploy and configure Wazuh Manager and Agent in a virtual environment.
- Integrate Suricata with Wazuh using `eve.json`.
- Create and tune custom Wazuh detection rules.
- Configure automated active response using `firewall-drop`.
- Validate detections through ICMP and Nmap-based attack simulations.
- Investigate alerts through the Wazuh Dashboard.

## ✅ Conclusion
This project demonstrates the deployment of a functional Network Intrusion Detection System (NIDS) integrated with a Security Information and Event Management (SIEM) platform in a virtual SOC lab.

By combining Suricata's signature-based intrusion detection capabilities with Wazuh's log analysis, alert correlation, dashboard visualization, and active response engine, the lab provides an end-to-end workflow from network traffic monitoring to automated threat mitigation.

The project reflects practical SOC analyst skills, including IDS configuration, rule tuning, endpoint monitoring, log analysis, alert investigation, and automated incident response in a Linux-based environment.


## 🔗 About
Completed as part of the **CodeAlpha Cybersecurity Internship**.
