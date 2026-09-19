# SOC-project
A SOC (Security Operations Center) Analyst is a front-line defender responsible for monitoring an organization's digital infrastructure, investigating potential cyber threats, and responding to security incidents. They analyze data across endpoints, cloud environments, identity managers, and network traffic to keep malicious actors out.
### <ins>Key Technical & Soft Skills</ins>
* SIEM & Log Analytics: Expertise in querying platforms like Splunk (SPL), Microsoft Sentinel (KQL), or Elastic to correlate logs across disparate sources. 
* Endpoint Detection & Response (EDR/XDR): Navigating tools like Wazuh, CrowdStrike, or Defender to track malicious processes, persistence mechanisms, and lateral movement. 
* Threat Frameworks: Mapping attacker behaviors directly to the MITRE ATT&CK matrix to identify tactics, techniques, and procedures (TTPs).
* Network & Endpoint Analysis: Understanding packet captures (Wireshark), Sysmon event logs, and OS internals (Windows/Linux). 
* Scripting & Automation: Writing Python, PowerShell, or Bash scripts to automate routine investigations and build SOAR playbooks.
* Report Writing & Communication: Articulating complex technical incidents into actionable business-impact reports for management and IT teams.
---

### SEIM TOOL USE IN THIS PROJECT
End-to-end Security Operations Center (SOC) lab environment featuring threat detection, log analysis, and automated incident response using Wazuh (SIEM/XDR), Splunk (log analytics), and Microsoft Sentinel (cloud-native SIEM/SOAR).
* Wazuh: Serves as the host-level XDR/SIEM engine, managing File Integrity Monitoring (FIM), rootkit detection, and endpoint telemetry.
* Splunk: Acts as the centralized log repository, utilizing custom Search Processing Language (SPL) queries and dashboards to analyze cross-platform event correlation.
* Microsoft Sentinel: Functions as the primary cloud-native SIEM/SOAR platform, handling incident management, threat intelligence integration, and automated playbook execution.
---
## 📋 Prerequisites & Lab Architecture Requirements for this projects

To build and operate this multi-SIEM SOC lab, ensure you meet the following hardware, virtual machine, and platform requirements:

---

### 1. Hardware & Virtualization Setup
Running multiple SIEM instances, endpoint agents, and attack environments requires adequate system resources:
* **RAM:** Minimum 16 GB (32 GB recommended to comfortably run target endpoints, Wazuh Manager, Splunk Enterprise, and Kali Linux simultaneously).
* **CPU:** 4+ Cores (Virtualization VT-x/AMD-V enabled in BIOS).
* **Storage:** 100 GB–150 GB SSD space for disk images, log indexing, and database storage.
* **Hypervisor:** [Oracle VirtualBox](https://www.virtualbox.org/) or [VMware Workstation / Fusion](https://www.vmware.com/).

---

### 2. Attack & Victim Machines (Lab Infrastructure)
* **Adversary Node:** 
  * **Kali Linux (Latest Release):** Used to execute red-team attack techniques, brute-force simulations, credential dumping, port scanning (Nmap), and payload execution (Metasploit/Cobalt Strike framework simulation).
* **Target Endpoints:**
  * **Windows 10/11 or Windows Server:** Target host equipped with **Sysmon** (using SwiftOnSecurity config) and custom PowerShell logging enabled.
  * **Ubuntu Linux 22.04 LTS:** Target server for SSH brute-force and web application attack simulations.

---

### 3. SIEM & Security Platforms Setup

#### 🐺 Wazuh (Host-Level XDR & SIEM)
* **Components:** Wazuh Manager, Wazuh Indexer, and Wazuh Dashboard (deployed via Docker, OVA, or Linux package).
* **Agents:** **Wazuh Agent** installed on Windows and Linux targets for:
  * File Integrity Monitoring (FIM)
  * Active Response scripts (blocking malicious IPs)
  * Vulnerability detection and Rootkit audit logging

#### ⚡ Splunk (Centralized Log Analytics & Splunk Enterprise)
* **Components:** **Splunk Enterprise** (Local/Free Edition) + **Splunk Universal Forwarder**.
* **Integrations:**
  * Ingesting Windows Event Logs, Sysmon XML logs, and Linux `/var/log/auth.log`.
  * Custom **Search Processing Language (SPL)** queries for event correlation and security dashboards.

#### ☁️ Microsoft Sentinel (Cloud-Native SIEM & SOAR)
* **Account:** Active **Microsoft Azure Subscription** (Free Trial or Pay-As-You-Go).
* **Components:**
  * **Azure Log Analytics Workspace (LAW)** configured for data collection.
  * **Azure Monitor Agent (AMA) / Data Collection Rules (DCR):** To bridge local endpoints and cloud telemetry into Sentinel.
  * **Kusto Query Language (KQL):** For writing detection rules, threat hunting, and configuring Logic App **SOAR Playbooks**.

---

### 4. Knowledge & Skill Prerequisites
* **Red Team Concepts:** Basic understanding of attack vectors (Port scanning, brute-forcing, lateral movement, credential dumping).
* **Query Languages:** Basic syntax familiarity with **SPL (Splunk)** and **KQL (Microsoft Sentinel)**.
* **Networking & Protocols:** Understanding Syslog forwarding (UDP/TCP 514), TCP/IP ports, Windows Event IDs, and the **MITRE ATT&CK Framework**.
