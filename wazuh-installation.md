# Wazuh SIEM Installation

## Overview

This document describes the installation and initial configuration of **Wazuh SIEM** on an **Ubuntu 22.04 LTS** virtual machine as part of the **SOC Home Lab – Security Monitoring & Threat Detection** project.

Wazuh is used as the central security monitoring platform to collect, analyze, and visualize security events from Windows and Linux endpoints.

The Wazuh deployment consists of:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

---

## 1. Lab Environment

| Component | Configuration |
|---|---|
| Operating System | Ubuntu 22.04 LTS |
| SIEM | Wazuh |
| Virtualization | VirtualBox |
| RAM | 4–8 GB |
| CPU | 2–4 Cores |
| Storage | 50 GB |
| Network | NAT + Host-Only |
| Server Role | Wazuh Server |

Example Wazuh Server IP:

```text
192.168.56.101
2. Wazuh Architecture

The Wazuh environment consists of three main server components.

                    ┌──────────────────────┐
                    │   Wazuh Dashboard    │
                    │   Web Interface      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Wazuh Indexer     │
                    │ Event Storage/Search │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Wazuh Manager      │
                    │ Event Analysis        │
                    │ Detection Rules       │
                    └──────────┬───────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                    ▼                     ▼
             Windows Agent         Ubuntu Agent
3. Prerequisites

Before installing Wazuh, verify that Ubuntu is updated.

Run:

sudo apt update

Then:

sudo apt upgrade -y

Check the Ubuntu version:

lsb_release -a

Expected:

Ubuntu 22.04 LTS

Check the system hostname:

hostname

Example:

ubuntu22-soc

Check the IP address:

hostname -I
4. Configure Hostname

Set the hostname:

sudo hostnamectl set-hostname ubuntu22-soc

Verify:

hostnamectl

Expected:

Static hostname: ubuntu22-soc
5. Download Wazuh Installation Assistant

The Wazuh installation assistant is used to deploy the Wazuh server components.

Official documentation:

Wazuh Installation Guide

Download the installation assistant using the command provided in the official Wazuh documentation.

Example:

curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh

Make the script executable:

chmod +x wazuh-install.sh

Always verify the current Wazuh documentation before installation because the installation commands may change between Wazuh releases.

6. Generate Wazuh Configuration

Generate the Wazuh configuration files using the installation assistant.

sudo bash wazuh-install.sh --generate-config-files

This creates the configuration required for the Wazuh components.

7. Install Wazuh Indexer

The Wazuh Indexer is responsible for storing and indexing security events.

Install the indexer using the installation assistant according to the Wazuh installation documentation.

Example:

sudo bash wazuh-install.sh --wazuh-indexer wazuh-server

After installation, verify the service:

sudo systemctl status wazuh-indexer

The service should show:

active (running)
8. Install Wazuh Manager

The Wazuh Manager receives events from Wazuh agents and analyzes them using decoders and detection rules.

Install the manager:

sudo bash wazuh-install.sh --wazuh-server wazuh-server

Check the service:

sudo systemctl status wazuh-manager

Expected:

active (running)
9. Install Wazuh Dashboard

The Wazuh Dashboard provides the web-based interface used by SOC analysts to monitor and investigate alerts.

Install the dashboard:

sudo bash wazuh-install.sh --wazuh-dashboard wazuh-server

Check the service:

sudo systemctl status wazuh-dashboard

Expected:

active (running)
10. Verify Wazuh Services

Check the status of all major services:

sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard

All required services should be running.

Example:

wazuh-manager     active (running)
wazuh-indexer     active (running)
wazuh-dashboard   active (running)
11. Access Wazuh Dashboard

Find the Wazuh server IP:

hostname -I

Then open a browser from the host machine.

Example:

https://192.168.56.101

Replace 192.168.56.101 with the actual Wazuh server IP.

The browser may display a certificate warning because the lab installation can use a self-signed certificate.

Proceed to the Wazuh Dashboard.

12. Wazuh Dashboard Login

Use the administrator credentials generated during the Wazuh installation.

After successful login, the Wazuh Dashboard should be displayed.

Screenshot

Add your actual screenshot here:

![Wazuh Dashboard](../screenshots/wazuh-dashboard.png)
13. Verify Wazuh Manager

Check the Wazuh Manager service:

sudo systemctl status wazuh-manager

Check Wazuh processes:

ps aux | grep wazuh

Check the Wazuh Manager log:

sudo tail -f /var/ossec/logs/ossec.log

This log can help identify configuration or service-related problems.

14. Verify Wazuh Indexer

Check:

sudo systemctl status wazuh-indexer

Check whether the service is listening:

sudo ss -tulpn

The Wazuh Indexer should be listening on its configured port.

15. Verify Wazuh Dashboard

Check:

sudo systemctl status wazuh-dashboard

If the dashboard is not accessible, check its logs and service status.

16. Firewall Configuration

If UFW is enabled, verify its status:

sudo ufw status

Required Wazuh ports should be allowed according to the Wazuh deployment architecture and the official documentation.

Do not open unnecessary ports.

17. Add Wazuh Agents

After successfully installing the Wazuh server, endpoints can be connected using Wazuh Agents.

The planned lab endpoints are:

Endpoint	Operating System	Role
Wazuh Server	Ubuntu 22.04	SIEM
Endpoint 1	Windows	Wazuh Agent
Endpoint 2	Ubuntu	Wazuh Agent

The agents send security events to the Wazuh Manager.

Windows Endpoint
       │
       ▼
 Wazuh Agent
       │
       ▼
Wazuh Manager
       │
       ▼
Detection Rules
       │
       ▼
Security Alert
       │
       ▼
Wazuh Dashboard
18. Verify Connected Agents

After deploying the agents, verify their status from:

Wazuh Dashboard
→ Agents

The agents should appear as connected/active.

Screenshot
![Wazuh Agents](../screenshots/wazuh-agents.png)
19. Test Security Monitoring

After connecting an endpoint, generate controlled security events inside the lab.

Examples:

Failed login attempts
Successful login
PowerShell activity
New user creation
File modification
SSH authentication failures

The Wazuh Agent should collect the events and forward them to the Wazuh Manager.

20. Verify Alerts

Navigate to:

Wazuh Dashboard
→ Security Events

Review the generated alerts.

Useful information includes:

Alert timestamp
Rule ID
Rule description
Severity
Source IP
Username
Hostname
Event type
Screenshot
![Wazuh Security Alert](../screenshots/wazuh-alert.png)
21. Installation Verification

The Wazuh installation is considered successful when:

 Ubuntu 22.04 is configured
 Wazuh Indexer is running
 Wazuh Manager is running
 Wazuh Dashboard is running
 Dashboard is accessible
 Wazuh Agent is installed
 Agent is connected
 Security events are being received
 Alerts are generated successfully
22. Troubleshooting

If a Wazuh service fails, check its status:

sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard

Check Wazuh Manager logs:

sudo tail -n 50 /var/ossec/logs/ossec.log

Check system logs:

sudo journalctl -xe

Check listening ports:

sudo ss -tulpn

Check network connectivity:

ping <WAZUH_SERVER_IP>
