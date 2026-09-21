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
