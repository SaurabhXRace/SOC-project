# Authentication Logs 

### Step 1: Check Universal Forwarder Status
Verify that the Splunk Universal Forwarder installed on your Linux system is actively connected to the Splunk Server.
```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```
* **Splunk Username:** `Userhero`
* **Password:** `Userhero@1826`
> **Tip:** Enter the Splunk admin username and password that were configured during the initial Splunk installation on Ubuntu.
---

### Step 2: Add Authentication Log File to Splunk Monitoring
Tell the forwarder to monitor `/var/log/auth.log`, which is the Linux system file where all user login attempts, `sudo` usage, and authentication activities are recorded.
```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

---

### Step 3: Restart Splunk Universal Forwarder
Restart the forwarder service so that it reloads its configuration and begins forwarding the newly added log path.
```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

---

### Step 4: Verify Log Collection in Splunk Dashboard
Open the Splunk Web UI on your browser and run a basic SPL (Search Processing Language) search to ensure raw logs are arriving in real time.
```spl
index=main source="/var/log/auth.log"
```

---

### Step 5: Generate Test Authentication Events
Create test log entries directly on the Ubuntu machine to test if Splunk captures both successful and failed events properly.

1. Open a new terminal instance in Ubuntu.
2. Run the superuser command:
   ```bash
   sudo su
   ```
3. Type an **incorrect password 3 to 4 times** to generate failed login logs.
4. Optionally, type the correct password to generate a successful login log.

---

### Step 6: Search & Filter Security Logs in Splunk
Return to the Splunk Search bar, refresh the page, and execute targeted SPL queries to investigate specific authentication events:

* **Filter for Failed Login Attempts:**
  ```spl
  index=main source="/var/log/auth.log" "Authentication failure"
  ```
  *Description: Displays all events where a user entered an incorrect password or failed authentication.*

* **Filter for Successful Login Attempts:**
  ```spl
  index=main source="/var/log/auth.log" "Accepted password"
  ```
  *Description: Displays all events where access was granted successfully.*

> **Tip:** Click the arrow (`>`) next to any log entry in Splunk to expand and inspect event details like source IP, username, and exact timestamps.

---
# SSH Monitoring
### SSH Simulation & Auditd Integration with Splunk

A step-by-step guide on setting up SSH log forwarding, simulating brute-force attacks, configuring Splunk alerts, and integrating Linux `auditd` for privilege escalation detection.

---
## Phase 1: SSH Service Setup & Brute-Force Simulation in Ubuntu
 ### 1.  SSH Service Verification & Setup
To capture SSH authentication activity, the OpenSSH server daemon must be active and running on the target system. First, verify whether the SSH daemon is running on your Ubuntu system.

### Ubuntu Target Setup

1. Check whether the SSH service is currently active:
   ```bash
   sudo systemctl status ssh
   ```

2. If the service is inactive or not installed, install the OpenSSH server:
   ```bash
   sudo apt update && sudo apt install openssh-server -y
   ```

3. Start the service and confirm it is running without errors:
   ```bash
   sudo systemctl start ssh
   sudo systemctl status ssh
   ```

4. Check and record the IP address of the Ubuntu machine:
   ```bash
   ip a
   ```
   > **Note:** For demonstration purposes, assume the Ubuntu host IP address is `192.168.22.58`.


---

### 2. Simulate Attack & Valid SSH Connections for Log Generation & Analysis (Kali Linux Attacker)
Simulate both unauthorized brute-force attempts and legitimate logins using an external machine (e.g., Kali Linux) to verify that Splunk is ingesting logs correctly.

### Kali Linux Terminal Actions

* **Scenario A: Simulating Brute-Force Attempts (Invalid User)**  
  Attempt to log into the Ubuntu machine using a non-existent account name (`Fakeuser`) and deliberately enter incorrect passwords 3 to 4 times:
* **Simulate Brute-Force Attack (Invalid Account):**
  ```bash
  ssh Fakeuser@192.168.22.58
  ```
 Simulates a brute-force attack using a non-existent account by entering an incorrect password 3 times, generating "Authentication failure" log entries in /var/log/auth.log.

* **Scenario B: Simulating Successful Authentication (Valid User)**  
  Log into the Ubuntu machine using valid credentials:
  ```bash
  ssh ubuntu@192.168.22.58
  ```
 * **Purpose:** Generates an `"Accepted password"` entry in `/var/log/auth.log`.

### Splunk Web Verification

Open Splunk Web in your browser and execute the following Search Processing Language (SPL) queries:

* **Query for Failed Login Attempts:**
  ```splunk
  index=main source="/var/log/auth.log" "Failed password"
  ```
  *(Filters all failed authentication attempts to inspect potential brute-force activity)*

* **Query for Successful Logins:**
  ```splunk
  index=main source="/var/log/auth.log" "Accepted password"
  ```
  *(Filters all successful SSH logins to monitor legitimate user access)*
---

## Phase 2: Splunk log Analysis, SSH Brute-force detection, & Alert Creation

### 1. Analyze SSH Logs with Field Extraction (SPL)
Navigate to **Splunk Search & Reporting** to analyze failed SSH authentication attempts. By applying Regular Expressions (`rex`), Splunk parses unstructured log data to extract structured fields, such as targeted user accounts and attacker IP addresses.

#### SPL Query with Field Extraction (`rex`): splunk
```spl
index=main source="/var/log/auth.log" ("Failed password" OR "Authentication failure")
| rex "Failed password for (invalid user)? (?<user>\S+) from (?<src_ip>\S+)"
| stats count as failures values(user) as users by src_ip
| sort -count
```

#### Explanation of the Query:
* `index=main source="/var/log/auth.log"`: Restricts search scope to Linux authentication logs.
* `("Failed password" OR "Authentication failure")`: Filters for explicit failure events.
* `| rex ...`: Uses field extraction to isolate the username (`user`) and attacker IP address (`src_ip`).
* `| stats count as failures values(user) as users by src_ip`: Groups the total failure count and targeted user list by source IP.
* `| sort -failures`: Orders output so the highest failure counts appear at the top.
---

### Configuring the Splunk Alert

1. Click **Save As** in the top right corner of Splunk Search and select **Alert**.
2. Configure the modal form with the following parameters:

| Configuration Field | Setting / Value |
| :--- | :--- |
| **Title** | `SSH Brute Force Detection` |
| **Description** | Triggers when multiple failed SSH login attempts are detected from a single source. |
| **Alert Type** | Scheduled (Select *Run on Cron Schedule*) |
| **Time Range** | Last 24 hours |
| **Cron Schedule** | `*/2 * * * *` *(Executes every 2 minutes)* |
| **Expire** | 24 hours |
| **Trigger Actions** | Click **+ Add Action** $\rightarrow$ **Add to Triggered Alerts** |
| **Severity** | Medium |

3. Leave remaining parameters as Default and click **Save**.

---

### 3. Verify Triggered Alerts
To test if the alert system is functional:
1. Return to the Kali Linux terminal and execute multiple failed SSH commands to trigger the rule.
2. In Splunk, go to the top navigation bar: **Activity** in the top navigation bar and select **Triggered Alerts** (or navigate to **Activity** $\rightarrow$ **Jobs**).
3. Locate the `SSH Brute Force Detection` alert, click **View Results** to inspect the attacker's IP and target usernames, or click **View Events** to drill down into raw log lines.

---

## Phase 3:  Auditd Installation & Log Forwarding
The Linux Audit Daemon (`auditd`) provides low-level kernel tracking, capturing process executions, file modifications, and privilege escalations (`sudo`).
### Ubuntu Setup
### 1. Install & Configure Auditd (Ubuntu Target)
`auditd` (Linux Audit Daemon) provides kernel-level event tracking to log system calls, command execution, and file accesses.

* **Install Audit Daemon and Plugins:**
  ```bash
  sudo apt install auditd audispd-plugins -y
  ```

* **Enable `auditd` to ensure it launches automatically upon boot, then start the service:**
  ```bash
  sudo systemctl enable auditd
  sudo systemctl start auditd
  ```

* **Verify that the system audit log file is being written:**
  ```bash
  sudo ls /var/log/audit
  ```

---

### 2. Forward Audit Logs to Splunk
Add the newly generated audit log path to the Splunk Universal Forwarder.

* **Add the audit log path to the Splunk Universal Forwarder configuration:**
  ```bash
  sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/audit/audit.log
  ```
  > **Note:** Enter your Splunk administrator credentials when prompted.

* **Restart the Splunk Universal Forwarder to begin forwarding data:**
  ```bash
  sudo /opt/splunkforwarder/bin/splunk restart
  ```

---

### 3. Generating System Activity for Audit Log Verification
Execute standard terminal commands on Ubuntu to populate log entries into `/var/log/audit/audit.log`:

```bash
pwd
ls -lacd /var/log
cat auth.log
```

---

* **Auditd Investigation & Root Escalation Alert**
  To view system audit records, set the time range picker to **Last 60 minutes** and run:
  ```spl
  index=main source="/var/log/audit/audit.log" "sudo"
  ```
 ### Root Session / Privilege Escalation Detection
Monitor when standard users execute administrative commands using `sudo` privileges.
 
#### SPL Query for Sudo Command Extraction
```splunk
index=main sourcetype="linux_audit" "sudo:"
| rex "(?i)^(?:\s*)sudo:\s+(?<user>\S+)\s*:\s*\s*COMMAND=(?<command>.*)$"
| stats count values(command) as command by host, user
| sort -count
```

#### Query Explanation
* `sourcetype="linux_audit" "sudo:"`: Targets system audit events involving administrative commands.
* `| rex ...`: Parses out the executing account (`user`) and the actual command executed (`command`).
* `| stats count values(command) as command by host, user`: Groups all unique commands run by each user on every monitored host.

### Alert Setup Procedure

1. Run the `sudo` extraction query in Splunk Search.
2. Click **Save As** $\rightarrow$ **Alert**.
3. Set the title to `Sudo Privilege Escalation Detected`.
4. Set the scheduled frequency, assign a **Medium** or **High** severity level, and click **Save**.
