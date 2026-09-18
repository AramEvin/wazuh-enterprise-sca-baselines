# Custom Wazuh SCA Policies for Enterprise Hardening

This repository contains a curated collection of production-ready, custom **Wazuh Security Configuration Assessment (SCA)** policies. These rules extend default baselines to provide comprehensive compliance and security auditing across heterogeneous environments.

## 🛡️ Covered OS Families

### 🐧 Linux (Debian & Red Hat Families)
* **Debian Family (Ubuntu / Debian):** Audits for package management (APT), UFW/iptables state, user management, and strict file system permissions (`/etc/shadow`, `/etc/passwd`).
* **Red Hat Family (RHEL / Rocky Linux / CentOS):** Verifies SELinux configurations, DNF/YUM security policies, and systemd core-dump restrictions.
* **Global Linux Checks:** SSH daemon hardening (ciphers, root login restrictions), cron-job integrity, and legacy unencrypted services detection (Telnet/FTP).

### 🪟 Windows Family
* **Windows Client & Server:** Validates local account lockout thresholds, Advanced Audit Policy Configurations, and Windows Defender settings.
* **Registry-Level Audits:** Detects insecure protocols (SMBv1, TLS 1.0/1.1) and enforces Remote Desktop Services (RDS) encryption requirements.

---

## 🚀 Deployment Instructions

### Method 1: Centralized Deployment via Wazuh Manager (Recommended)
To push policies to specific agent groups automatically:

1. Copy the desired `.yml` policy file into the shared group directory on your **Wazuh Manager**:
   ```bash
   cp linux/debian-family/custom_debian_baseline.yml /var/ossec/etc/shared/<YOUR_GROUP_NAME>/
   ```
2. Reference the policy within the group's `agent.conf` file:
   ```xml
   <sca>
     <policies>
       <policy enabled="yes">etc/shared/custom_debian_baseline.yml</policy>
     </policies>
   </sca>
   ```

### Method 2: Manual Local Agent Installation
For testing or standalone environments:
* **Linux:** Drop the `.yml` file into `/var/ossec/ruleset/sca/` and restart the agent:
  ```bash
  systemctl restart wazuh-agent
  ```
* **Windows:** Place the `.yml` file into `C:\Program Files (x86)\ossec-agent\ruleset\sca\` and restart the `Wazuh` service using `services.msc` or PowerShell.

---

## 🧪 Validation & Best Practices
* **Syntax Checking:** Before deploying to the manager, it is highly recommended to validate the YAML structure using a linter tool like `yamllint`.
* **Unique Rule IDs:** All custom rules in this repository use specific ID ranges to eliminate any risk of collisions with Wazuh's default upstream SCA rulesets.

## 📄 License
This project is open-source software licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.
