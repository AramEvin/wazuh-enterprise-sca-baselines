# Custom Wazuh SCA Policies for Enterprise Hardening

This repository contains a curated collection of production-ready, custom **Wazuh Security Configuration Assessment (SCA)** policies. These rules extend default baselines to provide comprehensive compliance and security auditing across heterogeneous environments.

## 🛡️ Covered OS Families

### 🐧 Linux (Debian & Red Hat Families)
* **Debian Family (Ubuntu / Debian):** Audits for package management (APT), UFW/iptables state, user management, and strict file system permissions (`/etc/shadow`, `/etc/passwd`).
* **Red Hat Family (RHEL / Rocky Linux / CentOS):** Verifies SELinux configurations, DNF/YUM security policies, and systemd core-dump restrictions.

### 🪟 Windows Family
* **Windows Client & Server:** Validates local account lockout thresholds, Advanced Audit Policy Configurations, and Windows Defender settings.

---

## 🚀 Step-by-Step Deployment Guide

### Step 1: Centralized Deployment (Wazuh Manager Side)

To deploy policies to specific agent groups automatically, manage them from the **Wazuh Manager**:

1. Copy the desired `.yml` policy file into the shared group directory on your manager:
   * **For Linux Agents:**
     ```bash
     cp linux/debian-family/custom_debian_baseline.yml /var/ossec/etc/shared/<YOUR_GROUP_NAME>/
     ```
   * **For Windows Agents:**
     ```bash
     cp windows/custom_windows_baseline.yml /var/ossec/etc/shared/<YOUR_GROUP_NAME>/
     ```

2. Set the **correct file permissions** on the Wazuh Manager so the engine can read and distribute the files:
   ```bash
   chown -R wazuh:wazuh /var/ossec/etc/shared/<YOUR_GROUP_NAME>/
   chmod 660 /var/ossec/etc/shared/<YOUR_GROUP_NAME>/*.yml
   ```

3. Update the group's **`agent.conf`** file on the Manager (via Web UI or directly in `/var/ossec/etc/shared/<YOUR_GROUP_NAME>/agent.conf`) to register the policies:
   ```xml
   <agent_config>
     <sca>
       <enabled>yes</enabled>
       <scan_on_start>yes</scan_on_start>
       <interval>12h</interval>
       <policies>
         <!-- Path relative to the agent's shared directory -->
         <policy enabled="yes">etc/shared/custom_debian_baseline.yml</policy>
         <policy enabled="yes">etc/shared/custom_windows_baseline.yml</policy>
       </policies>
     </sca>
   </agent_config>
   ```

---

### Step 2: Agent Configuration & Verification (Agent Side)

#### 1. Enable Configuration Receipt
Ensure your agents are allowed to accept remote configurations from the manager. 

* **Linux Agent (`/var/ossec/etc/ossec.conf`):**
  Ensure the `<sca>` block exists or check if remote execution is allowed if your rules use local commands:
  ```xml
  <sca>
    <enabled>yes</enabled>
  </sca>
  ```
* **Windows Agent (`C:\Program Files (x86)\ossec-agent\ossec.conf`):**
  Open the file with Administrative privileges and verify:
  ```xml
  <sca>
    <enabled>yes</enabled>
  </sca>
  ```

#### 2. Check if the Rules File is Received
Once the manager restarts or pushes the configuration, verify that the agent successfully downloaded the custom policy file:

* **Linux Agent Verification:**
  Check if the file arrived in the local shared folder and has secure permissions (`root:ossec` or `root:wazuh`):
  ```bash
  ls -la /var/ossec/etc/shared/
  # The custom_debian_baseline.yml should be visible here
  ```
* **Windows Agent Verification:**
  Open PowerShell as Administrator and check the local shared directory:
  ```powershell
  Get-ChildItem "C:\Program Files (x86)\ossec-agent\etc\shared\"
  # Verify custom_windows_baseline.yml is present
  ```

---

### Step 3: Log Monitoring & Troubleshooting

To ensure the SCA module executed your custom rules successfully without errors, monitor the agent logs.

#### 🐧 On Linux Agents:
Filter the `ossec.log` file for SCA engine specific logs:
```bash
grep -i "sca" /var/ossec/logs/ossec.log
```
* **What to look for:**
  * `wazuh-modulesd:sca: INFO: Loaded policy: 'etc/shared/custom_debian_baseline.yml'` (Success)
  * `wazuh-modulesd:sca: ERROR: ...` (Indicates a YAML syntax error or invalid operator in your rule)

#### 🪟 On Windows Agents:
Run the following command in PowerShell to audit the SCA scan process:
```powershell
Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.log" -Pattern "sca"
```
* **What to look for:**
  * `wazuh-agent: INFO: Loaded policy: 'etc/shared/custom_windows_baseline.yml'`
  * If the file is missing or blocked by permissions, look for `WARNING: Could not open policy` errors.

---

## 📄 License
This project is open-source software licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.
