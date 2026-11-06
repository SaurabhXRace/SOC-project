# Index & PowerShell Logging

### 1. Start Ubuntu Linux
Opened the Ubuntu Linux virtual machine to configure Splunk Universal Forwarder and prepare the log collection process.

---

### 2. Start Splunk Universal Forwarder

**Command:**
sudo /opt/splunkforwarder/bin/splunk start

**Description:**
Started the Splunk Universal Forwarder service on Ubuntu Linux to enable log collection and forwarding to the Splunk Enterprise server.

---

### 3. Configure the inputs.conf File

**Command:**
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

**Description:**
Opened the inputs.conf configuration file using the Nano text editor with administrative privileges. This file is used to define log sources that Splunk Universal Forwarder monitors and collects.

---

### 4. Configure Linux Authentication Log Monitoring

**Configuration:**
[monitor:///var/log/auth.log]
disabled = false
index = linux
sourcetype = linux_auth

**Description:**
- monitor: Configured Splunk to monitor the Ubuntu authentication log file.
- disabled = false: Enabled monitoring of the specified log source.
- index = linux: Assigned collected authentication events to the linux index in Splunk.
- sourcetype = linux_auth: Defined a custom sourcetype to identify and categorize Linux authentication logs.

---

### 5. Save the Configuration

**Commands / Shortcuts:**
- Ctrl + O — Saved the configuration file.
- Enter — Confirmed the file name.
- Ctrl + X — Exited the Nano text editor.

**Description:**
Saved the updated log monitoring configuration in inputs.conf to apply the required changes.

---

### 6. Restart Splunk Universal Forwarder

**Command:**
sudo /opt/splunkforwarder/bin/splunk restart

**Description:**
Restarted Splunk Universal Forwarder to reload the updated configuration and apply the new log monitoring settings.
