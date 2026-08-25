# Network Intrusion Detection System using Wazuh & Suricata

## Overview

This project documents the design and implementation of a Network Intrusion Detection System (NIDS) using Wazuh and Suricata in a virtual cybersecurity lab built with VirtualBox.

The objective was to simulate a Security Operations Center (SOC) environment capable of monitoring endpoint activity, detecting suspicious network traffic, and visualizing security alerts through the Wazuh Dashboard.

Unlike a traditional SIEM-only deployment, this project integrates Suricata IDS with Wazuh to provide real-time intrusion detection and centralized security monitoring.

## Lab Architecture

### Environment
| Virtual Machine	    | Role |
|------------------|----------------------------|
| Ubuntu 22.04	| Wazuh Manager |
| Kali Linux	  | Wazuh Agent + Suricata IDS |
| VirtualBox    |	Virtual Lab Environment | 

## Project Objectives
- Deploy a Wazuh SIEM server.
- Configure Kali Linux as a monitored endpoint.
- Install and configure Suricata IDS.
- Integrate Suricata alerts into Wazuh.
- Detect and investigate network events through Threat Hunting.

## Tools Used
| Tool	| Purpose |
| Wazuh	| SIEM Platform |
Suricata	Network Intrusion Detection System
Ubuntu 22.04	Wazuh Manager
Kali Linux	Endpoint & IDS Sensor
VirtualBox	Lab Virtualization
Linux CLI	Installation & Configuratio
