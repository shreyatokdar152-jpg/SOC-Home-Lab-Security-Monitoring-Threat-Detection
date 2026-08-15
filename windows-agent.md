# Windows Wazuh Agent Setup

## Overview

This document describes the installation and configuration of the **Wazuh Agent on a Windows endpoint** as part of the **SOC Home Lab – Security Monitoring & Threat Detection** project.

The Windows virtual machine acts as a monitored endpoint. The Wazuh Agent collects security-related events from the Windows system and forwards them to the centralized Wazuh Manager for analysis and alert generation.

---

## Architecture

The Windows endpoint communicates with the Wazuh Server through the internal SOC Home Lab network.

```text
                SOC HOME LAB

        ┌──────────────────────┐
        │   Ubuntu 22.04 VM    │
        │                      │
        │    Wazuh Server      │
        │    Wazuh Manager     │
        │    Wazuh Indexer     │
        │    Wazuh Dashboard   │
        └──────────┬───────────┘
                   │
                   │
             SOC Network
          192.168.56.0/24
                   │
                   ▼
        ┌──────────────────────┐
        │    Windows VM        │
        │                      │
        │    Wazuh Agent       │
        │                      │
        │    Windows Logs      │
        │    Security Events   │
        │    PowerShell Logs   │
        └──────────────────────┘
1. Lab Environment
Component	Configuration
Endpoint	Windows VM
Agent	Wazuh Agent
Server	Ubuntu 22.04
SIEM	Wazuh
Virtualization	VirtualBox
Network	Host-Only
Windows Role	Monitored Endpoint

Example network:

Wazuh Server:
192.168.56.101


Windows Endpoint:
192.168.56.102

Replace these example IP addresses with the actual IP addresses used in the lab.

2. Prerequisites

Before installing the Wazuh Agent, verify the following:

Windows VM is running.
Windows VM has network connectivity.
Wazuh Server is running.
Wazuh Manager IP address is known.
Windows user has Administrator privileges.
Windows can communicate with the Wazuh Server.

Wazuh requires administrator privileges to install the Windows Agent. The current Wazuh documentation supports Windows endpoint deployment through either the GUI or command line.

3. Check Windows IP Address

Open Command Prompt or PowerShell as Administrator.

Run:

ipconfig

Locate the network adapter connected to the SOC Host-Only network.

Example:

IPv4 Address: 192.168.56.102
4. Test Connectivity to Wazuh Server

From the Windows VM, test connectivity to the Wazuh Server.

ping 192.168.56.101

Expected result:

Reply from 192.168.56.101

If the Wazuh server responds, network communication between the endpoint and server is working.

If ping is blocked by firewall rules, successful ping is not mandatory. The important requirement is that the required Wazuh communication ports are reachable.

5. Download Wazuh Agent

The Wazuh Agent package can be downloaded from the official Wazuh documentation.

Official documentation:

https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html

Download the Windows .msi installer appropriate for the Wazuh version being used by the Wazuh Server.

Example:

wazuh-agent-4.x.x-1.msi

Keep the Wazuh Agent version compatible with the Wazuh Manager. Wazuh states that compatibility is guaranteed when the manager version is greater than or equal to the agent version.

6. Install Wazuh Agent Using GUI

Right-click the downloaded MSI file and select:

Run as administrator

Follow the installation wizard.

During installation, configure the Wazuh Manager address if prompted.

Example:

Wazuh Manager:
192.168.56.101

Complete the installation.

7. Install Wazuh Agent Using PowerShell

Alternatively, the Wazuh Agent can be installed from an elevated PowerShell window.

Navigate to the directory containing the MSI file.

Example:

cd C:\Users\<USERNAME>\Downloads

Then run:

msiexec.exe /i .\wazuh-agent-4.x.x-1.msi /q WAZUH_MANAGER="192.168.56.101"

Replace:

4.x.x

with the actual Wazuh Agent version.

Replace:

192.168.56.101

with the actual Wazuh Manager IP address.

The Wazuh documentation provides this deployment-variable approach for automatically configuring the manager address during installation.

8. Configure the Wazuh Manager Address

If the manager address was not configured during installation, edit the Wazuh Agent configuration file.

For a 64-bit Windows installation, the default configuration file is:

C:\Program Files (x86)\ossec-agent\ossec.conf

Open PowerShell as Administrator:

notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"

Locate the <client> section.

Configure the Wazuh Manager address:

<client>
  <server>
    <address>192.168.56.101</address>
  </server>
</client>

Replace the IP address with the actual Wazuh Manager IP.

Wazuh documents this configuration method for Windows agent enrollment.

9. Configure Windows Log Collection

The Wazuh Agent can be configured to collect Windows Event Logs.

Inside:

C:\Program Files (x86)\ossec-agent\ossec.conf

you can configure event channels such as:

<localfile>
  <location>Application</location>
  <log_format>eventchannel</log_format>
</localfile>


<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>


<localfile>
  <location>System</location>
  <log_format>eventchannel</log_format>
</localfile>

These channels provide useful security information for the SOC lab.

Examples include:

Authentication events
Account activity
System events
Application events
Security events
10. Start the Wazuh Agent

After configuration, start the Wazuh Agent service.

Open PowerShell as Administrator:

Start-Service wazuhsvc

Alternatively, using Command Prompt:

NET START WazuhSvc

The Wazuh documentation provides both methods for starting the Windows agent service.

11. Check Wazuh Agent Service

Run:

Get-Service wazuhsvc

Expected output should show the service as:

Status    Name
------    ----
Running   WazuhSvc

You can also use:

Get-Service | Where-Object {$_.Name -like "*wazuh*"}
12. Restart the Agent

Whenever ossec.conf is modified, restart the Wazuh Agent.

Restart-Service wazuhsvc

Alternatively:

net stop wazuh
net start wazuh

Restarting the service applies the updated configuration.

13. Check Wazuh Agent Log

The default Wazuh Agent installation directory on a 64-bit Windows system is:

C:\Program Files (x86)\ossec-agent

The agent log can be found at:

C:\Program Files (x86)\ossec-agent\ossec.log

View the latest entries using PowerShell:

Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50

To continuously monitor the log:

Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Wait

Look for messages indicating that the agent is connected to the Wazuh Manager.

14. Verify Agent in Wazuh Dashboard

Open the Wazuh Dashboard.

Navigate to:

Agents Management
        ↓
Summary

The Windows endpoint should appear in the list of agents.

The Wazuh documentation recommends checking Agents management > Summary to verify the newly enrolled Windows agent and its connection status.

Example:

Agent Name: Windows-SOC
Operating System: Windows
Status: Active
15. Screenshot – Windows Agent

Add a screenshot of the Windows Agent configuration:

![Windows Wazuh Agent](../screenshots/windows-agent-installation.png)
16. Screenshot – Wazuh Dashboard Agent

Add a screenshot showing the Windows agent connected to Wazuh:

![Windows Agent Connected](../screenshots/windows-agent-connected.png)
17. Verify Agent Communication

From the Wazuh Dashboard, verify:

Agent name
Agent ID
IP address
Operating system
Agent status
Last keepalive
Wazuh version

Example:

Agent:
Windows-SOC


Status:
Active


IP:
192.168.56.102
18. Generate a Test Security Event

To verify that the Windows endpoint is actually being monitored, generate a controlled security event.

For example, perform several incorrect login attempts on the Windows VM.

The Windows Security Event Log should record the authentication failures.

Open:

Event Viewer
        ↓
Windows Logs
        ↓
Security

Look for the relevant authentication event.

19. Verify the Event in Wazuh

Open:

Wazuh Dashboard
        ↓
Security Events

Search/filter for events generated by:

Windows-SOC

Verify that the event is visible in the Wazuh Dashboard.

20. Test PowerShell Monitoring

PowerShell activity can also be used as a SOC detection test.

For example, execute a harmless command:

Get-Process

or:

Get-Service

The event can be monitored through the configured Windows logging and Wazuh rules.

Only perform security testing inside your authorized home-lab environment.

21. Test File Integrity Monitoring

A test directory can be created for File Integrity Monitoring.

Example:

mkdir C:\SOC-Test

Create a test file:

New-Item C:\SOC-Test\test.txt

Modify it:

"Test modification" | Out-File C:\SOC-Test\test.txt

Delete it:

Remove-Item C:\SOC-Test\test.txt

If the directory is configured for Wazuh FIM, these changes can generate security events.

22. Troubleshooting
Agent is not appearing in Wazuh

Check:

Get-Service wazuhsvc

Make sure the service is running.

Check the agent log:

Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50

Verify the manager IP in:

C:\Program Files (x86)\ossec-agent\ossec.conf
Duplicate Agent Name

If the Wazuh Manager reports:

Duplicate agent name

make sure the Windows VM has a unique agent name.

For example:

Windows-SOC

or:

Windows-Endpoint-01

Do not register multiple endpoints with the same intended agent identity.

Agent Shows as Disconnected

Check:

Get-Service wazuhsvc

Then verify network connectivity:

ping 192.168.56.101

Check the Wazuh Manager IP:

C:\Program Files (x86)\ossec-agent\ossec.conf

Restart the agent:

Restart-Service wazuhsvc

Then check:

Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50
23. Evidence / Screenshots

The following screenshots document the Windows Agent deployment.

Windows IP Configuration
![Windows IP Configuration](../screenshots/windows-ipconfig.png)
Wazuh Agent Installation
![Wazuh Agent Installation](../screenshots/windows-agent-installation.png)
Agent Configuration
![Agent Configuration](../screenshots/windows-agent-config.png)
Wazuh Dashboard – Connected Agent
![Connected Windows Agent](../screenshots/windows-agent-connected.png)
Windows Security Event
![Windows Security Event](../screenshots/windows-security-event.png)
Wazuh Security Alert
![Wazuh Security Alert](../screenshots/windows-wazuh-alert.png)
24. Final Verification Checklist

The Windows Agent deployment is considered successful when:

 Windows VM is connected to the SOC network
 Wazuh Agent is installed
 Wazuh Manager IP is configured
 Wazuh Agent service is running
 Agent is enrolled
 Agent appears in Wazuh Dashboard
 Agent status is Active
 Windows Security Events are collected
 Events appear in Wazuh
 Test security events generate alerts
25. SOC Workflow

The completed Windows monitoring workflow is:

Windows VM
    │
    ▼
Windows Security Events
    │
    ▼
Wazuh Agent
    │
    │ Encrypted/Auth. Communication
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
Wazuh Indexer
    │
    ▼
Wazuh Dashboard
    │
    ▼
SOC Analyst
    │
    ├── Alert Validation
    ├── Investigation
    ├── Evidence Collection
    └── Response
26. Related Documentation
Ubuntu 22.04 Setup
Wazuh Installation
Ubuntu Wazuh Agent
SOC Use Cases
