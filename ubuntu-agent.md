# Wazuh Agent Installation Guide (Ubuntu)

This guide covers installing the Wazuh agent on an Ubuntu endpoint (22.04/24.04) and enrolling it with an existing Wazuh manager/server. Repeat this on every host you want to monitor.

## 1. Prerequisites

- Root or sudo access on the endpoint
- Network connectivity from the endpoint to the Wazuh manager on ports:
  - `1514/tcp` — event/log forwarding
  - `1515/tcp` — initial enrollment (auto-registration)
- The IP address or hostname of your Wazuh manager
- ~35 MB RAM available (the agent is lightweight)

Use the official Wazuh repository rather than any distro-provided package — distro repos often lag several major versions behind.

## 2. Install prerequisite packages

```bash
sudo apt-get update
sudo apt-get install -y gnupg apt-transport-https
```

## 3. Add the Wazuh GPG key and repository

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
  sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt-get update
```

## 4. Install the agent with the manager address set

Set `WAZUH_MANAGER` to your Wazuh server's IP or hostname before installing so the agent auto-enrolls on first start:

```bash
sudo WAZUH_MANAGER='<wazuh-manager-ip-or-hostname>' apt-get install -y wazuh-agent
```

Optional deployment variables you can set alongside `WAZUH_MANAGER`:

| Variable | Purpose |
|---|---|
| `WAZUH_AGENT_NAME` | Custom name for this agent (defaults to hostname) |
| `WAZUH_AGENT_GROUP` | Assign the agent to a group at enrollment |
| `WAZUH_REGISTRATION_PASSWORD` | Password if the manager requires it for enrollment |
| `WAZUH_REGISTRATION_SERVER` | Separate registration server, if different from `WAZUH_MANAGER` |

Example with a custom agent name and group:

```bash
sudo WAZUH_MANAGER='10.10.100.10' \
     WAZUH_AGENT_NAME='web01' \
     WAZUH_AGENT_GROUP='webservers' \
     apt-get install -y wazuh-agent
```

## 5. Enable and start the agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

Check that it's running:

```bash
sudo systemctl status wazuh-agent
```

You should see `active (running)`.

## 6. Verify enrollment

Tail the agent log to confirm it connected:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

Look for a line indicating the agent started and is using an encrypted connection to the manager.

On the **Wazuh manager**, list registered agents to confirm this endpoint appeared:

```bash
sudo /var/ossec/bin/agent_control -l
```

The new host should show status `Active` within a minute or two. If it shows `Never connected`, see the troubleshooting section below.

## 7. (Optional) Monitor additional log files

By default the agent collects `/var/log/syslog` and auth logs. To add application-specific log files, edit the agent's local config:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add a `<localfile>` block inside `<ossec_config>` for each extra log source, for example:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/nginx/access.log</location>
</localfile>
```

Restart the agent to apply changes:

```bash
sudo systemctl restart wazuh-agent
```

## 8. Manual registration (if auto-enrollment is disabled)

If the manager doesn't allow auto-enrollment, install without the `WAZUH_MANAGER` variable, then register manually from the manager side and import the key on the agent:

```bash
# On the manager: generate a key for the new agent, then on the agent:
sudo /var/ossec/bin/agent-auth -m <wazuh-manager-ip>
sudo systemctl restart wazuh-agent
```

## 9. Troubleshooting

- **Agent shows "Never connected"**: confirm ports 1514/1515 are open between the endpoint and manager (check firewalls/security groups on both ends).
- **Wrong manager IP**: check `/var/ossec/etc/ossec.conf` on the agent for the `<address>` field under `<client>` and correct it if needed, then restart the service.
- **Clock skew**: large time differences between agent and manager can break the encrypted channel — verify NTP is working on both hosts.

## 10. Uninstalling the agent

```bash
sudo apt-get remove --purge wazuh-agent
```

Package removal does not delete all files by default. To fully clean up:

```bash
sudo rm -rf /var/ossec
```

## References

- [Wazuh Agent Installation Guide](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)
- [Deployment Variables for Linux](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/deployment-variables/deployment-variables-linux.html)
- [Wazuh Agent Enrollment](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html)
- [Configuring Log Data Collection](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html)

---
*Note: The repository URL uses the `4.x` series, which always points to the latest stable release. Confirm current package/version details against the [official Wazuh agent docs](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html) before deploying at scale.*
