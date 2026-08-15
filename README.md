# Use Case: SSH Brute-Force Detection

## Objective

Detect repeated failed SSH authentication attempts from a single source IP against a monitored Linux endpoint — a classic brute-force / credential-guessing attack pattern.

**MITRE ATT&CK mapping:** [T1110.001 – Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)

## How it works

Wazuh already decodes SSH authentication failures via its default `sshd` decoder and rule `5716` (SSH authentication failure). Rather than alert on every single failure — which is noisy and rarely meaningful on its own — this custom rule correlates **multiple** failures from the **same source IP** within a short time window, using Wazuh's `frequency` / `timeframe` / `same_source_ip` rule options.

| Setting | Value | Why |
|---|---|---|
| `if_matched_sid` | `5716` | Only fires on the built-in SSH auth-failure rule, so we inherit all field extraction (srcip, user, etc.) for free |
| `frequency` | `6` | Requires 6 matching events before the rule fires |
| `timeframe` | `120` | ...within a 120-second window |
| `same_source_ip` | — | Correlation key: all 6 failures must come from the same `srcip` |
| `level` | `10` | High enough to surface as a real alert, not noise |

This mirrors how the default Wazuh ruleset already handles brute force (rule `5710`/`5712` build on similar logic) but is written from scratch here to demonstrate the detection-engineering process end to end.

## The rule

See [`rule.xml`](rule.xml). Deployed to `/var/ossec/etc/rules/local_rules.xml` on the Wazuh manager.

## Testing / attack simulation

Simulated from an external attacker-controlled Linux host using [Hydra](https://github.com/vanhauser-thc/thc-hydra):

```bash
# On the attacker host
sudo apt-get install -y hydra
hydra -l badguy -P /usr/share/wordlists/rockyou.txt ssh://<monitored-endpoint-ip> -t 4
```

This generates repeated `Failed password` entries in `/var/log/auth.log` on the monitored endpoint, which the Wazuh agent forwards to the manager.

### Validation with `wazuh-logtest`

Before restarting the manager, the rule was validated locally by replaying a captured `auth.log` line:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

```
Nov 12 03:14:07 web01 sshd[8842]: Failed password for invalid user badguy from 203.0.113.55 port 51422 ssh2
```

Expected output confirms the log decodes as `sshd` and, after 6 repeats within 120 seconds from the same `srcip`, escalates to custom rule `100010`.

## Result

After the simulated attack, the alert appeared in **Wazuh Dashboard → Threat Hunting**, filtered on rule ID `100010`, showing:
- Source IP of the attacking host
- Target username(s) attempted
- Number of failures within the correlation window
- MITRE ATT&CK technique tag (T1110.001) surfaced automatically in the dashboard's ATT&CK view

See [`screenshots/`](screenshots/) for the alert as it appeared in the dashboard.

## Suggested follow-up (not implemented here)

- Pair this rule with [Active Response](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/ar-use-cases/blocking-ssh-brute-force.html) to automatically block the source IP via `firewall-drop` after the threshold is hit.
- Tune `frequency`/`timeframe` based on observed baseline traffic to reduce false positives on hosts with legitimate high-retry automation (e.g., CI/CD runners).
