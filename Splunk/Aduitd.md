# Auditd
Installation and Splunk Log Forwarding on Ubuntu

## 1. Introduction

Auditd (Linux Audit Daemon) is a Linux security auditing service that monitors system activities and records security-related events in audit log files.

It helps Security Operations Center (SOC) analysts monitor activities such as authentication attempts, user actions, file access, and system changes.

In this lab, Auditd is installed on Ubuntu Linux, configured to run as a system service, and its audit logs are forwarded to Splunk Universal Forwarder for centralized security monitoring and analysis.

## 2. Environment

- **Operating System:** Ubuntu Linux
- **Security Tool:** Auditd
- **SIEM:** Splunk Enterprise
- **Log Forwarding:** Splunk Universal Forwarder

## 3. Installation and Configuration

### Step 1: Install Auditd

```bash
sudo apt install auditd audispd-plugins -y
```

**Command Explanation:**

- `sudo`: Executes the command with administrative privileges.
- `apt install`: Installs the specified packages.
- `auditd`: Linux Audit Daemon responsible for collecting and recording audit events.
- `audispd-plugins`: Provides plugins for processing and distributing audit events.
- `-y`: Automatically confirms the installation prompt.

### Step 2: Enable Auditd Service

```bash
sudo systemctl enable auditd
```

**Command Explanation:**

- `systemctl`: Manages system services in Linux.
- `enable`: Configures Auditd to start automatically during system boot.
- `auditd`: Specifies the Auditd service.

### Step 3: Start Auditd Service

```bash
sudo systemctl start auditd
```

**Command Explanation:**

- `start`: Starts the Auditd service immediately.
- `auditd`: Specifies the service to start.

### Step 4: Verify Auditd Service Status

```bash
sudo systemctl status auditd
```

**Command Explanation:**

Displays the current status of the Auditd service.

Check whether the service is active and running successfully.

Expected status:

```text
Active: active (running)
```

### Step 5: Verify Audit Log Directory

```bash
sudo ls /var/log/audit
```

**Command Explanation:**

- `ls`: Lists files and directories.
- `/var/log/audit`: Default directory where Auditd stores audit log files.

The main audit log file is:

```text
/var/log/audit/audit.log
```

### Step 6: Forward Audit Logs to Splunk

Add the Auditd log file as a monitored input in Splunk Universal Forwarder.

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/audit/audit.log
```

**Command Explanation:**

- `sudo`: Executes the command with administrative privileges.
- `/opt/splunkforwarder/bin/splunk`: Executes the Splunk Universal Forwarder command-line interface.
- `add monitor`: Configures Splunk to monitor a specified file.
- `/var/log/audit/audit.log`: Specifies the Auditd log file that Splunk should monitor.

After executing the command, provide your Splunk username and password when prompted.

Splunk Universal Forwarder will monitor the Auditd log file and forward newly collected log events to the configured Splunk receiving server.

## 4. Restart Splunk Universal Forwarder

After adding the Auditd log file as a monitored input, restart Splunk Universal Forwarder to apply the configuration.

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

**Command Explanation:**

- `sudo`: Executes the command with administrative privileges.
- `/opt/splunkforwarder/bin/splunk`: Executes the Splunk Universal Forwarder command-line interface.
- `restart`: Stops and starts the Splunk Universal Forwarder service to reload its configuration.

## 5. Basic Linux Commands

Use the following commands in the Ubuntu terminal to navigate directories, check files, and read log contents.

### 1. pwd – Print Working Directory

```bash
pwd
```

**Command Explanation:**

Displays the current working directory (the location where you are currently working).

### 2. ls – List Files and Directories

```bash
ls
```

**Command Explanation:**

Displays the files and directories available in the current working directory.

### 3. cd – Change Directory

```bash
cd
```

**Command Explanation:**

Changes the current working directory.

Examples:

```bash
cd /var/log/audit
```

Navigates to the Auditd log directory.

```bash
cd ..
```

Moves one directory level up.

### 4. cat – Display File Contents

```bash
cat filename
```

**Command Explanation:**

Displays the contents of a specified file in the terminal.

Example:

```bash
sudo cat /var/log/audit/audit.log
```

Displays the contents of the Auditd log file.

### 5. Open Multiple Files

```bash
cat file1 file2
```

**Command Explanation:**

Displays the contents of multiple files sequentially in the terminal.

Example:

```bash
cat file1.txt file2.txt
```

## 6. Search Auditd Logs in Splunk

After configuring the Splunk Universal Forwarder, open Splunk Enterprise and navigate to:

**Search & Reporting → Search**

Enter the following SPL query:

```spl
index=main source="/var/log/audit/audit.log"
```

**Search Query Explanation:**

- `index=main`: Searches for events stored in the `main` index.
- `source="/var/log/audit/audit.log"`: Filters events collected from the specified Auditd log file.

### Step 1: Select Time Range

Select **Last 60 minutes** from the time range picker to display events received during the previous 60 minutes.

### Step 2: Filter Specific Activities

To investigate specific activities, add relevant keywords to the search query.

Example: Search for events containing `sudo`.

```spl
index=main source="/var/log/audit/audit.log" sudo
```

**Query Explanation:**

- `sudo`: Filters events containing the keyword `sudo`.
- This helps identify audit log entries that contain references to sudo activity.

You can replace `sudo` with other keywords depending on the activity you want to investigate.

## 7. Create Sudo Activity / Privilege Escalation Alert in Splunk

### 1. Introduction

Sudo (Superuser Do) allows Linux users to execute commands with elevated privileges.

Monitoring sudo activity is important for SOC analysts because unauthorized or suspicious privilege escalation attempts may indicate potential security threats.

In this lab, we use Splunk Search Processing Language (SPL) to identify sudo command activity, extract usernames and executed commands, summarize the results, and create an alert for monitoring.

### 2. SPL Search Query

Open **Splunk Enterprise → Search & Reporting → Search**.

Enter the following SPL query:

```spl
index=main sourcetype="linux_audit" "sudo:"
| rex field=_raw "(?i)^\s*sudo:\s+(?<user>\S+)\s*:\s+.*?COMMAND=(?<command>[^;\r\n]+)"
| stats count values(command) as command by host, user
| sort -count
```

### 3. Command Explanation

**Command 1: Search Sudo Events**

```spl
index=main sourcetype="linux_audit" "sudo:"
```

- `index=main`: Searches events stored in the main index.
- `sourcetype="linux_audit"`: Filters events identified with the linux_audit sourcetype.
- `"sudo:"`: Searches for events containing the sudo keyword.

**Command 2: Extract Username and Executed Command**

```spl
| rex field=_raw "(?i)^\s*sudo:\s+(?<user>\S+)\s*:\s+.*?COMMAND=(?<command>[^;\r\n]+)"
```

- `rex`: Extracts fields from raw event data using a regular expression.
- `field=_raw`: Applies the regular expression to the original raw log event.
- `(?i)`: Makes the regular expression case-insensitive.
- `(?<user>\S+)`: Extracts the username and stores it in a field named `user`.
- `.*?`: Matches characters between the username and the COMMAND field.
- `(?<command>[^;\r\n]+)`: Extracts the executed command and stores it in a field named `command`.

**Command 3: Count and Group Sudo Activities**

```spl
| stats count values(command) as command by host, user
```

- `stats`: Performs statistical calculations on search results.
- `count`: Counts the number of matching events.
- `values(command)`: Displays unique executed commands.
- `as command`: Assigns the output field name as `command`.
- `by host, user`: Groups the results by hostname and username.

**Command 4: Sort Results**

```spl
| sort -count
```

- `sort`: Sorts the search results.
- `-count`: Sorts the results in descending order based on event count.

This displays users with the highest number of matching sudo events first.

### 4. Create a Sudo Activity Alert

After executing the search query, save it as an alert.

**Steps to Create the Alert:**

1. Execute the SPL search query in Splunk.
2. Click **Save As**.
3. Select **Alert**.
4. Enter an alert title:

   `Sudo Activity - Privilege Escalation Monitoring`

5. Add a description explaining that the alert monitors sudo command activity.
6. Configure the alert type and trigger conditions.
7. Configure the alert action, if required.
8. Save the alert.

**Recommended Alert Configuration:**

- **Alert Name:** Sudo Activity - Privilege Escalation Monitoring
- **Search Type:** Scheduled
- **Trigger Condition:** Number of results greater than 0
- **Time Range:** Last 60 minutes
- **Alert Action:** Add to Triggered Alerts

### 5. View Triggered Alerts

After saving the alert:

1. Navigate to **Activity → Triggered Alerts** in Splunk.
2. Locate the Sudo Activity alert.
3. Open the alert to review its triggering events.
4. Investigate the hostname, username, and executed commands.
