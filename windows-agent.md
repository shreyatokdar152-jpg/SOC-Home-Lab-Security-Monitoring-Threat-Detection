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
