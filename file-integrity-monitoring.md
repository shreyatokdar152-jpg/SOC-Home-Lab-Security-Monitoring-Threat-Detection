# File Integrity Monitoring (FIM)

## Overview

File Integrity Monitoring (FIM) is a security monitoring technique used to detect unauthorized changes to important files and directories.

In this SOC Home Lab, **Wazuh File Integrity Monitoring** is configured to monitor files on the Ubuntu endpoint. The purpose of this use case is to identify file creation, modification, and deletion events and generate security alerts when monitored files are changed.

This use case demonstrates how a SOC analyst can use Wazuh to detect potentially unauthorized changes to files on a monitored endpoint.

---

## Objective

The main objectives of this use case are:

- Monitor important files and directories.
- Detect file creation.
- Detect file modification.
- Detect file deletion.
- Generate Wazuh security alerts.
- Investigate the detected changes.
- Identify the affected file and endpoint.
- Demonstrate practical SOC monitoring and investigation.

---

## Environment

| Component | Details |
|---|---|
| SIEM | Wazuh |
| Endpoint | Ubuntu 22.04 |
| Wazuh Agent | Installed on Ubuntu endpoint |
| Wazuh Manager | Ubuntu 22.04 |
| Detection | File Integrity Monitoring |
| Testing Environment | VirtualBox SOC Home Lab |

Example endpoint:

```text
Ubuntu Agent:
192.168.56.103
