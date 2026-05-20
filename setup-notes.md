Phase 1 — Environment Setup
1.1 Repository & Workspace

Created a public GitHub repository (wazuh-soc-homelab) to document the build and maintain version control.
Established a dedicated cybersecurity workspace on the Windows 11 host machine.

1.2 Installing VMware Workstation Pro

Installed VMware Workstation Pro on the Windows 11 host.
Created a new virtual machine named Wazuh-Server with the following specs:

RAM: 10 GB
CPU: 6 cores
Storage: 80 GB
Networking: NAT


NAT networking was chosen to allow internet access from the VM while keeping it isolated from the host network.

1.3 Installing Ubuntu Server

⚠️ ISO Compatibility Issue
Ubuntu Server 26.04 LTS was initially used but is not supported by Wazuh 4.11. The wazuh-indexer service repeatedly crashed on startup due to incompatibility with unsupported libraries, kernel behaviour, and Java/OpenSearch dependencies. This caused "connection refused" errors between the dashboard and backend services, making the SIEM stack non-functional despite the installer reporting success.
Diagnosed with:
bashsystemctl status wazuh-indexer
Fix: Rebuild the VM using Ubuntu Server 22.04.5 LTS, which is officially supported by Wazuh 4.11.


Installed Ubuntu Server 22.04.5 LTS as the VM operating system.
Enabled SSH during installation for remote administration.
Configured passwordless SSH authentication using ED25519 keys.
Updated the system:

bash
"sudo apt update && sudo apt upgrade -y"

## 1.4 Deploying Wazuh SIEM Platform

Successfully deployed the Wazuh 4.11 all-in-one stack on Ubuntu Server 22.04.5 LTS using the official automated installer.

### Components Installed
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

### Deployment Method
Installed using the official Wazuh installation script:

```bash
curl -sO https://packages.wazuh.com/4.11/wazuh-install.sh
chmod +x wazuh-install.sh
sudo ./wazuh-install.sh -a

## Validation Checks

Verified:

Wazuh manager service operational
Dashboard service accessible
Indexer service functioning correctly
Remote SSH administration working through VS Code Remote-SSH
## Dashboard Access

Dashboard URL:

https://192.168.197.130

Successfully authenticated to the Wazuh web interface using the generated admin credentials.
Browser displayed a self-signed certificate warning, which is expected in a homelab environment.
VS Code Remote-SSH was configured successfully for remote administration and Git operations directly from the Ubuntu server.
Initial dashboard confirmed successful deployment with active alert ingestion and system monitoring enabled.
## Outcome

The SOC homelab now has a fully operational SIEM platform capable of:

~centralized log collection
~threat detection
~endpoint monitoring
~alert investigation
~security event visualization

## Next phase:
Deploy Windows agents and integrate Sysmon telemetry for endpoint detection and attack simulation.

## 1.5 Windows Agent Enrollment

### Objective

Connect a Windows 11 endpoint to the Wazuh SIEM platform for centralized endpoint monitoring and telemetry collection.

---

### Actions Performed

#### Verified Wazuh Manager Services

Validated that the Wazuh manager and agent authentication services were operational on the Ubuntu server.

Commands used:

```bash
sudo systemctl status wazuh-manager
sudo ss -tulpn | grep 1514
sudo ss -tulpn | grep 1515
```

Verified:
- Port 1514 open for agent communication
- Port 1515 open for agent enrollment/authentication

---

#### Installed Windows Wazuh Agent

Installed the Wazuh Windows agent on the Windows 11 endpoint system.

Agent version:
- Wazuh Agent v4.11.0

---

#### Agent Enrollment

Registered the Windows endpoint with the Wazuh manager using agent-auth.

Command executed from Administrator PowerShell:

```powershell
& "C:\Program Files (x86)\ossec-agent\agent-auth.exe" -m 192.168.197.130 -A WIN11-ENDPOINT-2
```

Successfully received a valid authentication key from the Wazuh manager.

---

### Troubleshooting Performed

#### Issue 1 — Duplicate Agent Name

Initial enrollment failed due to an existing agent entry already registered on the manager.

Resolution:
- Renamed the endpoint to:
  - WIN11-ENDPOINT-2

---

#### Issue 2 — Wazuh Service Would Not Start

The Windows Wazuh agent service repeatedly failed after enrollment.

Log analysis revealed:

```text
ERROR: Invalid server address found: '0.0.0.0'
ERROR: No client configured. Exiting.
```

Cause:
- Incorrect manager IP configured inside:
  - ossec.conf

Resolution:
- Updated manager address from:
  - 0.0.0.0
- To:
  - 192.168.197.130

Configuration file updated:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

---

### Validation

Successfully verified:

- Wazuh agent service running
- Endpoint visible in Wazuh dashboard
- Agent status marked as active
- Windows OS details populated correctly
- Endpoint IP address detected
- Telemetry successfully reaching the SIEM server

PowerShell validation command:

```powershell
Get-Service WazuhSvc
```

Result:

```text
Running  WazuhSvc
```

---

### Outcome

The SOC homelab now includes an actively monitored Windows 11 endpoint connected to the Wazuh SIEM platform.

Current capabilities include:

- Centralized endpoint monitoring
- Windows telemetry collection
- Agent-based log forwarding
- Endpoint inventory visibility
- Real-time SIEM ingestion
- Foundation for Sysmon integration and threat detection workflows