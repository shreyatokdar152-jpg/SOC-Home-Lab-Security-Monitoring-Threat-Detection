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
