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
