# Wazuh All-in-One Installation Guide (Ubuntu/Debian)

This guide walks through installing Wazuh 4.14.x as a single-node, all-in-one deployment on Ubuntu (22.04/24.04) or Debian 12, using the official assisted installer. All three central components — the **Wazuh indexer**, **Wazuh server**, and **Wazuh dashboard** — are installed on one host.

## 1. Prerequisites

| Requirement | Minimum |
|---|---|
| OS | Ubuntu 22.04 / 24.04, Debian 12 |
| CPU | 4 vCPU |
| RAM | 8 GB (indexer JVM heap defaults to ~4 GB) |
| Disk | 50 GB+ |
| Access | Root or sudo |
| Network | Outbound access to `packages.wazuh.com` |

Update the system first:

```bash
sudo apt update && sudo apt upgrade -y
```

## 2. Download the installation assistant

```bash
cd /root
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
ls -la wazuh-install.sh
```

If the file is empty or you get a 404, check the current release series at the [Wazuh installation guide](https://documentation.wazuh.com/current/installation-guide/index.html) — the version number in the URL (`4.14`) may have advanced.

## 3. Run the all-in-one installation

```bash
sudo bash wazuh-install.sh -a
```

The `-a` flag installs the indexer, server, and dashboard together on this host. This typically takes 10–20 minutes depending on your connection and hardware.

What happens during install:
- The Wazuh indexer (an OpenSearch fork) is installed and configured, listening on port `9200`.
- The Wazuh server (manager) is installed, listening on `1514/tcp` (agent traffic) and `1515/tcp` (agent enrollment).
- Filebeat is installed to ship alerts from the server to the indexer.
- The Wazuh dashboard is installed behind HTTPS on port `443`.
- Self-signed certificates are generated automatically for internal communication.

## 4. Retrieve the generated admin credentials

The installer prints admin credentials at the end of the run, and also saves them to a log file. Retrieve them with:

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Save the `admin` username and password securely (e.g., in a password manager) — you'll need them to log into the dashboard.

## 5. Access the Wazuh dashboard

From a browser on the same network, navigate to:

```
https://<wazuh-server-ip>
```

Accept the self-signed certificate warning (or replace it with a proper cert later — see Step 7) and log in with the `admin` credentials retrieved above.

## 6. Verify services are running

```bash
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
sudo systemctl status filebeat
```

All four should show `active (running)`. Enable them on boot if not already:

```bash
sudo systemctl enable wazuh-indexer wazuh-manager wazuh-dashboard filebeat
```

## 7. (Optional) Tune the indexer JVM heap

The installer sets the indexer heap to ~4 GB, suited to hosts with 8 GB+ RAM. On smaller hosts, adjust it to roughly half your available RAM (capped at 31 GB):

```bash
sudo nano /etc/wazuh-indexer/jvm.options
```

Edit the `-Xms` and `-Xmx` lines, then restart:

```bash
sudo systemctl restart wazuh-indexer
```

## 8. (Optional) Use a real TLS certificate

The dashboard ships with a self-signed cert. To replace it with a Let's Encrypt certificate, follow the [Wazuh dashboard SSL guide](https://documentation.wazuh.com/current/user-manual/wazuh-dashboard/configuring-third-party-certs/ssl.html).

## 9. Enroll a Wazuh agent

On each endpoint you want to monitor, install the Wazuh agent and point it at this server's IP or hostname. Example for a Debian/Ubuntu agent host:

```bash
curl -o wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.0-1_amd64.deb \
  && sudo WAZUH_MANAGER='<wazuh-server-ip>' dpkg -i ./wazuh-agent.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Check the exact package filename and latest version on the [Wazuh agent installation page](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html) for other OSes (Windows, macOS, Solaris, AIX, HP-UX).

Once the agent connects, it appears under **Agents** in the dashboard and alerts should begin flowing within a minute or two.

## 10. Uninstalling (if needed)

```bash
sudo bash wazuh-install.sh --uninstall
```

This removes the central components. To remove an agent, see the [uninstalling agents guide](https://documentation.wazuh.com/current/installation-guide/uninstalling-wazuh/agent.html).

## References

- [Wazuh Installation Guide](https://documentation.wazuh.com/current/installation-guide/index.html)
- [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
- [Wazuh Agent Enrollment](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html)
- [Wazuh Indexer Tuning](https://documentation.wazuh.com/current/user-manual/wazuh-indexer/wazuh-indexer-tuning.html)

---
*Note: Always confirm the current release series (`4.x`) against the [official installation guide](https://documentation.wazuh.com/current/installation-guide/index.html) before running these commands, as package URLs are versioned and change with each release.*

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
