# Ubuntu 22.04 Desktop Setup – VirtualBox

## Overview

This document describes the installation and configuration of **Ubuntu 22.04 LTS Desktop** inside **Oracle VirtualBox** as part of the **SOC Home Lab – Security Monitoring & Threat Detection** project.

Ubuntu 22.04 is used as the primary Linux virtual machine for the SOC environment and can be configured as the **Wazuh Server**.

---

## 1. Requirements

### Hardware Requirements

The following resources are recommended for the Ubuntu virtual machine:

| Resource |             Recommended |
| -------- | ----------------------: |
| RAM      |                  4–8 GB |
| CPU      |               2–4 Cores |
| Storage  |                   50 GB |
| Network  | NAT + Host-Only Adapter |

The exact resources can be adjusted depending on the available hardware of the host computer.

### Software Requirements

* Windows Host Machine
* Oracle VirtualBox
* Ubuntu 22.04 LTS Desktop ISO

---

## 2. Download Ubuntu 22.04

Download the Ubuntu 22.04 LTS Desktop ISO from the official Ubuntu website:

[Ubuntu 22.04 LTS](https://releases.ubuntu.com/22.04/)

The required ISO should be similar to:

```text
ubuntu-22.04.x-desktop-amd64.iso
```

---

## 3. Create the Virtual Machine

Open **Oracle VirtualBox** and select:

```text
New
```

Configure the VM as follows:

```text
Name:        Ubuntu22-SOC
Type:        Linux
Version:     Ubuntu (64-bit)
```

Select the downloaded Ubuntu 22.04 Desktop ISO.

### Screenshot

Add a screenshot of the VM creation screen:

```text
![VirtualBox VM Creation](../screenshots/ubuntu-vm-creation.png)
```

---

## 4. Configure Hardware

Recommended configuration:

```text
Memory:       6 GB
Processors:   4
Storage:      50 GB
```

For systems with limited resources, the VM can be configured with:

```text
Memory:       4 GB
Processors:   2
Storage:      50 GB
```

### Screenshot

```text
![VirtualBox Hardware Configuration](../screenshots/ubuntu-hardware-config.png)
```

---

## 5. Configure Virtual Disk

Create a new virtual hard disk with:

```text
Disk Type: VDI
Storage: Dynamically Allocated
Size: 50 GB
```

The virtual disk is used only by the Ubuntu virtual machine.

---

## 6. Configure Network

The SOC lab uses two network adapters.

### Adapter 1 – NAT

NAT provides Internet access to Ubuntu.

```text
Adapter 1:
Enable Network Adapter: Yes
Attached to: NAT
```

This connection is used for:

* Ubuntu updates
* Installing packages
* Downloading software
* Wazuh installation

### Adapter 2 – Host-Only Adapter

The Host-Only Adapter provides private communication between the SOC lab virtual machines.

```text
Adapter 2:
Enable Network Adapter: Yes
Attached to: Host-Only Adapter
```

Example network:

```text
192.168.56.0/24
```

### Screenshot

```text
![VirtualBox Network Configuration](../screenshots/ubuntu-network-config.png)
```

---

# 7. Install Ubuntu 22.04 Desktop

Start the VM and boot from the Ubuntu ISO.

Select:

```text
Install Ubuntu
```

---

## 8. Select Language

Select:

```text
English
```

Click **Continue**.

---

## 9. Keyboard Layout

Select the appropriate keyboard layout.

Example:

```text
English (US)
```

Click **Continue**.

---

## 10. Network Connection

Allow Ubuntu to use the available network connection.

For the initial installation, DHCP can be used.

The static IP configuration can be performed later after the installation.

---

## 11. Installation Type

Select:

```text
Normal Installation
```

This provides the complete Ubuntu Desktop environment.

---

## 12. Third-Party Software

Enable third-party software if required.

Then continue with the installation.

---

## 13. Disk Configuration

Since this is a dedicated VirtualBox virtual disk, select:

```text
Erase disk and install Ubuntu
```

> **Important:** This option applies to the virtual disk assigned to the VM. Make sure you are installing Ubuntu inside the VirtualBox VM and not modifying a physical disk.

Continue with the installation.

---

## 14. Select Time Zone

Select the appropriate time zone.

For India:

```text
Kolkata
```

Continue.

---

## 15. Create User Account

Create a local Ubuntu user.

Example:

```text
Name:              SOC Administrator
Computer Name:     ubuntu22-soc
Username:          socadmin
Password:          ********
```

For the computer name, use:

```text
ubuntu22-soc
```

This makes the system easier to identify within the SOC lab.

---

## 16. Complete Installation

Ubuntu will begin installing.

Wait for the installation to complete and select:

```text
Restart Now
```

Remove the installation ISO if prompted.

After restarting, log in using the username and password created during installation.

---

# 17. Update Ubuntu

Open the Terminal:

```text
Ctrl + Alt + T
```

Run:

```bash
sudo apt update
```

Then:

```bash
sudo apt upgrade -y
```

Install commonly used tools:

```bash
sudo apt install curl wget git unzip net-tools openssh-server -y
```

---

# 18. Verify Ubuntu Version

Run:

```bash
lsb_release -a
```

Expected output:

```text
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.x LTS
Release:        22.04
Codename:       jammy
```

---

# 19. Verify Hostname

Run:

```bash
hostname
```

Expected output:

```text
ubuntu22-soc
```

---

# 20. Check Network Interfaces

Run:

```bash
ip addr
```

or:

```bash
hostname -I
```

The system should have network connectivity through NAT and a private IP through the Host-Only Adapter.

Example:

```text
NAT IP:
10.0.2.x

SOC Lab IP:
192.168.56.101
```

> **Note:** These are example addresses. Your actual IP addresses may be different.

---

# 21. Test Internet Connectivity

Run:

```bash
ping -c 4 google.com
```

If replies are received, Internet connectivity is working.

Example:

```text
64 bytes from google.com: ...
64 bytes from google.com: ...
```

---

# 22. Test Local SOC Network

After configuring the other SOC virtual machines, test communication with them.

For example:

```bash
ping 192.168.56.102
```

and:

```bash
ping 192.168.56.103
```

Replace these addresses with the actual IP addresses of your Windows and Ubuntu endpoints.

Successful responses confirm that the virtual machines can communicate over the SOC lab network.

---

# 23. Enable SSH

Check SSH status:

```bash
sudo systemctl status ssh
```

If SSH is not running:

```bash
sudo systemctl enable --now ssh
```

Verify:

```bash
sudo systemctl status ssh
```

SSH will be useful for remotely managing the Ubuntu machine during the SOC lab setup.

---

# 24. Configure Static IP

For the Wazuh server, a static IP is recommended so that agents always know where to send their events.

Example:

```text
IP Address:    192.168.56.101
Subnet:        255.255.255.0
Network:       192.168.56.0/24
```

The exact configuration should match the Host-Only network configured in VirtualBox.

After configuring the static IP, verify it with:

```bash
ip addr
```

and:

```bash
ip route
```

---

# 25. Create VirtualBox Snapshot

After completing the Ubuntu installation, updates, and basic configuration, create a snapshot.

In VirtualBox:

```text
VirtualBox
→ Ubuntu22-SOC
→ Snapshots
→ Take
```

Snapshot name:

```text
Ubuntu 22.04 Clean Installation
```

This provides a restore point before installing Wazuh.

---

# 26. Final Configuration

At the end of this setup, the Ubuntu VM should have:

| Configuration | Value                    |
| ------------- | ------------------------ |
| OS            | Ubuntu 22.04 LTS Desktop |
| Hostname      | `ubuntu22-soc`           |
| RAM           | 4–8 GB                   |
| CPU           | 2–4 cores                |
| Disk          | 50 GB                    |
| Adapter 1     | NAT                      |
| Adapter 2     | Host-Only                |
| SOC Network   | `192.168.56.0/24`        |
| SSH           | Enabled                  |

---

## 27. Next Step

After completing the Ubuntu 22.04 setup, the next stage of the SOC Home Lab is to install and configure **Wazuh**.

The planned deployment is:

```text
Ubuntu 22.04
      │
      ▼
Wazuh Installation
      │
      ├── Wazuh Manager
      ├── Wazuh Indexer
      └── Wazuh Dashboard
              │
              ▼
       Windows Wazuh Agent
              │
              ▼
       Ubuntu Wazuh Agent
              │
              ▼
       Security Alerts
              │
              ▼
       SOC Investigation
```

This Ubuntu machine will serve as the foundation for the **Wazuh-based SOC Home Lab**.
