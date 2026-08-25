# Network Intrusion Detection System using Wazuh & Suricata

## Overview

This project documents the design and implementation of a Network Intrusion Detection System (NIDS) using Wazuh and Suricata in a virtual cybersecurity lab built with VirtualBox.

The objective was to simulate a Security Operations Center (SOC) environment capable of monitoring endpoint activity, detecting suspicious network traffic, and visualizing security alerts through the Wazuh Dashboard.

Unlike a traditional SIEM-only deployment, this project integrates Suricata IDS with Wazuh to provide real-time intrusion detection and centralized security monitoring.

## Lab Architecture

### Environment
| Virtual Machine	    | Role |
| Ubuntu 22.04	Wazuh Manager
| Kali Linux	Wazuh Agent + Suricata IDS
| VirtualBox	Virtual Lab Environme
