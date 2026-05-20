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
##Outcome

The SOC homelab now has a fully operational SIEM platform capable of:

~centralized log collection
~threat detection
~endpoint monitoring
~alert investigation
~security event visualization

## Next phase:
Deploy Windows agents and integrate Sysmon telemetry for endpoint detection and attack simulation.