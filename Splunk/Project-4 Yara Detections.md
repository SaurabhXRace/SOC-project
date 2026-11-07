# YARA Malware Detection & Splunk Log Monitoring on Ubuntu Linux

## 1. Introduction

In this lab, we configure YARA on Ubuntu Linux to detect suspicious or potentially malicious files using custom YARA rules.

We will also integrate YARA with Splunk Universal Forwarder to collect and monitor malware detection logs in Splunk Enterprise.

### Objectives

- Install and configure YARA on Ubuntu Linux.
- Create custom YARA detection rules.
- Detect suspicious PowerShell commands and Mimikatz-related strings.
- Generate YARA detection logs.
- Forward YARA logs to Splunk Enterprise.
- Configure automated malware scanning using Cron Jobs.
- Detect suspicious files without knowing their filenames in advance.
- Create Splunk alerts for detection events.

### Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu Linux |
| Detection Tool | YARA |
| SIEM | Splunk Enterprise |
| Log Forwarder | Splunk Universal Forwarder |
| Log Source | YARA Detection Logs |
| Automation | Cron Jobs |

---

## 2. Install YARA on Ubuntu Linux

### Step 1: Open Ubuntu Terminal

Open the Ubuntu terminal to install YARA.

### Step 2: Install YARA

**Command:**

```bash
sudo apt install yara -y
```
<img width="1622" height="603" alt="Screenshot 2026-11-07 132404" src="https://github.com/user-attachments/assets/e0935a4f-e4e2-411b-8ff5-a17f67638c0e" />


**Command Explanation:**

- `sudo`: Executes the command with administrator privileges.
- `apt`: Ubuntu's package management tool.
- `install`: Installs a software package.
- `yara`: Specifies the YARA package.
- `-y`: Automatically confirms the installation prompt.

**Purpose:**

This command installs YARA on Ubuntu Linux. YARA is a pattern-matching tool used to identify suspicious files based on predefined rules and string patterns.

### Step 3: Verify YARA Installation

**Command:**

```bash
yara --version
```
<img width="1096" height="220" alt="Screenshot 2026-11-07 133227" src="https://github.com/user-attachments/assets/a447ba43-8bd7-43dc-b31b-2f692c8e87ec" />


**Command Explanation:**

- `yara`: Runs the YARA command-line utility.
- `--version`: Displays the installed YARA version.

**Purpose:**

This command verifies that YARA has been successfully installed and is accessible from the terminal.

---

## 3. Create a File Containing Suspicious Commands

### Step 1: Create malware.txt

**Command:**

```bash
nano malware.txt
```
<img width="1645" height="435" alt="Screenshot 2026-11-07 133730" src="https://github.com/user-attachments/assets/e2775f11-2710-4a5e-ad84-71c33eb49eb2" />

**Command Explanation:**

- `nano`: Opens the Nano text editor.
- `malware.txt`: Specifies the name of the file to create or edit.
  
**Purpose:**

This file stores suspicious command strings that will be used as test data for YARA detection.

### Step 2: Add Suspicious PowerShell Commands

Search for examples of suspicious PowerShell command-line patterns and add them to `malware.txt`.

Examples of detection strings:

```text
EncodedCommand
Invoke-WebRequest
Mimikatz
```

**Explanation:**

- `EncodedCommand`: A PowerShell parameter commonly associated with executing Base64-encoded commands. It can also be used for legitimate administrative purposes.
- `Invoke-WebRequest`: A PowerShell command used to make HTTP and HTTPS requests. Attackers may also misuse it to download files or communicate with remote servers.
- `Mimikatz`: The name of a well-known credential-access tool that can be used to extract credentials from Windows systems.

**Purpose:**

These strings act as test indicators for our custom YARA rule.

**Note:** These are detection indicators, not proof that a file is malicious. They can also appear in legitimate scripts and documentation.

### Step 3: Save the File

Use the following Nano keyboard shortcuts:

- `Ctrl + X`: Exit the Nano editor.
- `Y`: Confirm saving changes.
- `Enter`: Confirm the filename.

---

## 4. Create a Custom YARA Detection Rule

### Step 1: Create the YARA Rule File

**Command:**

```bash
nano detect_malware.yara
```
<img width="1308" height="432" alt="Screenshot 2026-11-07 134745" src="https://github.com/user-attachments/assets/9aa4c1a9-35ac-498c-8126-0cbd04c375f7" />


**Command Explanation:**

- `nano`: Opens the Nano text editor.
- `detect_malware.yara`: Creates a file to store the custom YARA detection rule.

**Purpose:**

This file contains the conditions and string patterns YARA will use to identify suspicious files.

### Step 2: Write the YARA Rule

Add the following rule:

```yara
rule Suspicious_Powershell
{
    meta:
        author = "Saurabh"
        description = "Detect Suspicious Powershell Behavior"

    strings:
        $a = "EncodedCommand"
        $b = "Invoke-WebRequest"
        $c = "Mimikatz"

    condition:
        any of them
}
```

### Line-by-Line Explanation

**1. Rule Declaration**

```yara
rule Suspicious_Powershell
```

- `rule`: Defines a new YARA detection rule.
- `Suspicious_Powershell`: Name assigned to the rule.

This name identifies the rule when YARA detects a matching file.

**2. Metadata Section**

```yara
meta:
    author = "Saurabh"
    description = "Detect Suspicious Powershell Behavior"
```

- `meta:`: Defines the metadata section of the rule.
- `author`: Specifies the rule creator.
- `description`: Explains the purpose of the rule.

Metadata helps document and manage custom YARA rules.

**3. Strings Section**

```yara
strings:
    $a = "EncodedCommand"
    $b = "Invoke-WebRequest"
    $c = "Mimikatz"
```

- `strings:`: Defines the text patterns YARA should search for.
- `$a`: First string identifier.
- `$b`: Second string identifier.
- `$c`: Third string identifier.

YARA searches the scanned file for these defined patterns.

**4. Condition Section**

```yara
condition:
    any of them
```

- `condition:`: Defines the matching condition.
- `any of them`: Returns a match if at least one of the defined strings is found.

For example:
- If `EncodedCommand` is found, the rule matches.
- If `Invoke-WebRequest` is found, the rule matches.
- If `Mimikatz` is found, the rule matches.

**Important Note:**

You can add more strings to the `strings:` section and extend your detection capabilities. You can also create multiple rules in the same YARA file.

### Step 3: Save the YARA Rule

- `Ctrl + X`: Exit Nano.
- `Y`: Confirm saving.
- `Enter`: Confirm the filename.

---

## 5. Test the YARA Detection Rule

### Step 1: Run the YARA Scan

**Command:**

```bash
yara detect_malware.yara malware.txt
```
<img width="1755" height="107" alt="Screenshot 2026-11-07 135032" src="https://github.com/user-attachments/assets/ecd861d9-c757-4571-942f-c3bf52e25f5a" />

**Command Explanation:**

- `yara`: Executes the YARA scanning utility.
- `detect_malware.yara`: Specifies the rule file.
- `malware.txt`: Specifies the file to scan.

**Purpose:**

YARA compares the contents of `malware.txt` against the patterns defined in `detect_malware.yara`.

If at least one string matches, YARA displays the matching rule name and scanned filename.

**Expected Output Example:**

```text
Suspicious_Powershell malware.txt
```

This indicates that the file matched the `Suspicious_Powershell` rule.

---

## 6. Create a Directory and Log File for YARA

YARA does not automatically create a dedicated directory for storing detection logs. We need to create one manually.

### Step 1: Create the YARA Log Directory

**Command:**

```bash
sudo mkdir /var/log/yara
```
<img width="1410" height="83" alt="Screenshot 2026-11-07 140150" src="https://github.com/user-attachments/assets/8f8648fd-c1af-4960-a449-7fe51c15f006" />

**Command Explanation:**

- `sudo`: Executes the command with administrator privileges.
- `mkdir`: Creates a new directory.
- `/var/log/yara`: Specifies the directory location.

**Purpose:**

Creates a dedicated directory under `/var/log` to store YARA detection logs.

### Step 2: Create the YARA Log File

**Command:**

```bash
sudo touch /var/log/yara/yara_logs.log
```

**Command Explanation:**

- `sudo`: Executes the command with administrator privileges.
- `touch`: Creates an empty file if it does not already exist.
- `/var/log/yara/yara_logs.log`: Specifies the log file location.

**Purpose:**

Creates a log file where YARA detection results will be stored.

### Step 3: Switch to Root User

**Command:**

```bash
sudo su
```

**Command Explanation:**

- `sudo`: Requests administrator privileges.
- `su`: Switches to another user, root by default.

**Purpose:**

Switches the current terminal session to the root user so administrative operations can be performed.

**Security Note:** Use root privileges only when necessary.

---

## 7. Store YARA Detection Results in the Log File

### Step 1: Execute YARA and Redirect Its Output

**Command:**

```bash
sudo yara detect_malware.yara malware.txt >> /var/log/yara/yara_logs.log
```
<img width="1502" height="145" alt="Screenshot 2026-11-07 140858" src="https://github.com/user-attachments/assets/01e18eac-007e-4002-bf59-f694771745e0" />

**Command Explanation:**

- `sudo`: Executes the command with elevated privileges.
- `yara`: Runs the YARA scanning utility.
- `detect_malware.yara`: Specifies the detection rule.
- `malware.txt`: Specifies the file to scan.
- `>>`: Appends command output to an existing file without overwriting previous contents.
- `/var/log/yara/yara_logs.log`: Destination log file.

**Purpose:**

Executes the YARA scan and stores matching detection results in the YARA log file.

### Step 2: View the YARA Log File

**Command:**

```bash
cat /var/log/yara/yara_logs.log
```

**Command Explanation:**

- `cat`: Displays the contents of a file.
- `/var/log/yara/yara_logs.log`: Specifies the log file.

**Purpose:**

Displays the YARA detection results stored in the log file.

---

## 8. Configure Splunk Universal Forwarder to Monitor YARA Logs

Now we configure Splunk Universal Forwarder to monitor the YARA log file and forward the detection results to Splunk Enterprise.

### Step 1: Open inputs.conf

**Command:**

```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```
<img width="1006" height="633" alt="Screenshot 2026-11-07 163203" src="https://github.com/user-attachments/assets/5fd970a2-d539-40c0-adf3-f30dbe577ff0" />

**Command Explanation:**

- `sudo`: Runs the command with administrator privileges.
- `nano`: Opens the Nano editor.
- `/opt/splunkforwarder/etc/system/local/inputs.conf`: Specifies the Splunk Universal Forwarder input configuration file.

**Purpose:**

The `inputs.conf` file defines which log files Splunk Universal Forwarder should monitor and collect.

### Step 2: Add the YARA Log Monitoring Configuration

Add the following configuration:

```ini
[monitor:///var/log/yara/yara_logs.log]
disabled = false
index = linux
sourcetype = yara_logs
```

**Configuration Explanation:**

**1. Monitor Stanza**

```ini
[monitor:///var/log/yara/yara_logs.log]
```

- `monitor://`: Tells Splunk Universal Forwarder to monitor a specified file.
- `/var/log/yara/yara_logs.log`: Specifies the YARA log file.

**2. Enable Monitoring**

```ini
disabled = false
```

Enables monitoring of the specified log file.

**3. Specify the Index**

```ini
index = linux
```

Specifies the Splunk index where YARA logs will be stored.

The `linux` index must already exist in Splunk Enterprise.

**4. Specify the Sourcetype**

```ini
sourcetype = yara_logs
```

Assigns the sourcetype `yara_logs` to the collected events.

This helps identify and search YARA detection events in Splunk.

### Step 3: Save the Configuration

- `Ctrl + X`: Exit Nano.
- `Y`: Confirm saving.
- `Enter`: Confirm the filename.

### Step 4: Restart Splunk Universal Forwarder

**Command:**

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```
<img width="1482" height="538" alt="Screenshot 2026-11-07 142939" src="https://github.com/user-attachments/assets/79315418-68b7-425b-ac7d-b07d51dd93fd" />

**Command Explanation:**

- `sudo`: Executes the command with administrator privileges.
- `/opt/splunkforwarder/bin/splunk`: Specifies the Splunk Universal Forwarder executable.
- `restart`: Restarts the Splunk Universal Forwarder service.

**Purpose:**

Applies the updated configuration and restarts the forwarder so it can monitor the YARA log file.

---

## 9. Verify YARA Logs in Splunk Enterprise

### Step 1: Execute Another YARA Scan

**Command:**

```bash
yara detect_malware.yara malware.txt
```

**Purpose:**

Runs another scan to generate a fresh detection event.

For the event to appear in Splunk, its output must also be written to the monitored log file. For example:

```bash
sudo yara detect_malware.yara malware.txt >> /var/log/yara/yara_logs.log
```

### Step 2: Open Splunk Enterprise

Navigate to:

**Splunk Enterprise → Search & Reporting**

### Step 3: Search for YARA Detection Logs

**Splunk Search Query:**

```spl
index=linux sourcetype="yara_logs"
```
<img width="1781" height="797" alt="Screenshot 2026-11-07 163034" src="https://github.com/user-attachments/assets/26070bfe-9239-483e-82ad-6dd37efe572b" />

**Query Explanation:**

- `index=linux`: Searches events stored in the Linux index.
- `sourcetype="yara_logs"`: Filters events associated with the YARA log sourcetype.

**Expected Result:**

Splunk should display the YARA detection events received through Splunk Universal Forwarder.

---

## 10. Automate YARA Scanning Using Cron Jobs

Instead of manually executing YARA scans every time, we can use Cron Jobs to schedule automatic scanning at regular intervals.

### Step 1: Open the Crontab Editor

**Command:**

```bash
crontab -e
```

**Command Explanation:**

- `crontab`: Manages scheduled tasks for the current user.
- `-e`: Opens the crontab file for editing.

**Purpose:**

Allows us to schedule YARA scanning at predefined time intervals.

Select option `1` if prompted to choose an editor.

### Step 2: Add the Cron Job

Add the following line at the end of the crontab file:

```cron
*/2 * * * * echo "$(date '+\%Y-\%m-\%d \%H:\%M:\%S') - $(/usr/bin/yara /home/perna/Desktop/detct_malware.yara /home/pererna/Desktop/malware.txt)" >> /var/log/yara/yara_logs.log
```

**Important:** The above is your original command. It contains inconsistent directory names and a potentially incorrect rule filename. Replace them with your actual file paths before using it.

For example, assuming both files are in `/home/perna/Desktop/`:

```cron
*/2 * * * * echo "$(date '+\%Y-\%m-\%d \%H:\%M:\%S') - $(/usr/bin/yara /home/perna/Desktop/detect_malware.yara /home/perna/Desktop/malware.txt)" >> /var/log/yara/yara_logs.log
```

### Line-by-Line Explanation

**1. Cron Timing**

```cron
*/2 * * * *
```

This specifies the scanning schedule.

- `*/2`: Executes the task every two minutes.
- First `*`: Every hour.
- Second `*`: Every day of the month.
- Third `*`: Every month.
- Fourth `*`: Every day of the week.

**2. Date Command**

```bash
date '+\%Y-\%m-\%d \%H:\%M:\%S'
```

Displays the current date and time in the following format:

```text
2026-11-07 17:30:00
```

The backslashes before `%` are necessary in crontab command fields because an unescaped `%` has special meaning to cron.

**3. Execute YARA**

```bash
/usr/bin/yara /home/perna/Desktop/detect_malware.yara /home/perna/Desktop/malware.txt
```

- `/usr/bin/yara`: Absolute path of the YARA executable.
- `detect_malware.yara`: Specifies the detection rule.
- `malware.txt`: Specifies the file to scan.

**4. Append Results**

```bash
>> /var/log/yara/yara_logs.log
```

Appends the date, time, and YARA scan output to the log file.

**Purpose:**

This Cron Job automatically executes the YARA scan every two minutes and stores its output in the YARA log file.

**Important:** This example uses the current user's crontab. Ensure that the user has permission to read the files and write to the log file. Otherwise, configure the job through root's crontab using `sudo crontab -e`.

### Step 3: Save the Cron Job

Save and exit the editor.

### Step 4: Verify Logs in Splunk

Open Splunk Enterprise and execute:

```spl
index=linux sourcetype="yara_logs"
```

Check whether the scheduled scan results are being received.

### Step 5: Configure a Splunk Alert

Navigate to:

**Splunk Enterprise → Search & Reporting**

1. Execute the YARA detection search.
2. Select **Save As → Alert**.
3. Provide an alert name.
4. Configure the alert trigger conditions.
5. Select the desired alert schedule.
6. Configure an alert action, such as email notification, if available.

**Purpose:**

The alert helps notify the security analyst when YARA detection events are received.

---

## 11. Detect Suspicious Files Without Knowing Their Filenames

In the previous steps, we scanned a specific file named `malware.txt`.

However, in a real-world environment, we may not know the filename of a suspicious file in advance.

To address this limitation, we can use Linux commands and Cron Jobs to identify and scan files within a specified directory.

### Step 1: Remove the Previous Test File

**Command:**

```bash
rm -r malware.txt
```

**Command Explanation:**

- `rm`: Removes files or directories.
- `-r`: Enables recursive removal.
- `malware.txt`: Specifies the file to remove.

**Purpose:**

Removes the previous test file so we can demonstrate detection using a newly created file.

**Note:** For a regular file, `rm malware.txt` is sufficient; the `-r` option is unnecessary.

Removing `malware.txt` means that the previous scan can no longer detect that particular file because it no longer exists.

### Step 2: Understand Filename-Independent Detection

YARA does not depend on a specific filename to identify suspicious content.

It analyzes the contents of a file against the configured rules.

For example, a file named:

```text
malware_demo.sh
```

can trigger the same YARA rule if its contents contain one or more matching strings.

### Step 3: Create a New Test File

**Command:**

```bash
nano malware_demo.sh
```

**Command Explanation:**

- `nano`: Opens the Nano text editor.
- `malware_demo.sh`: Creates a new shell script file.

Add harmless test strings:

```bash
#!/bin/bash

echo "EncodedCommand"
echo "Invoke-WebRequest"
echo "Mimikatz"
```

**Purpose:**

Creates a new test file containing the same indicators defined in the YARA rule.

### Step 4: Save the File

- `Ctrl + X`: Exit Nano.
- `Y`: Confirm saving.
- `Enter`: Confirm the filename.

### Step 5: Configure Scanning of Files Without Knowing Their Names

The following example scans all regular files inside a specified directory using `find`.

Create a scanning script:

```bash
sudo nano /usr/local/bin/yara_directory_scan.sh
```

Add:

```bash
#!/bin/bash

RULE="/home/perna/Desktop/detect_malware.yara"
TARGET="/home/perna/Desktop"
LOG="/var/log/yara/yara_logs.log"

find "$TARGET" -type f -print0 |
while IFS= read -r -d '' file
do
    result=$(/usr/bin/yara "$RULE" "$file" 2>/dev/null)

    if [ -n "$result" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $result" >> "$LOG"
    fi
done
```

Replace the example paths with your actual locations.

### Line-by-Line Explanation

**1. Shebang**

```bash
#!/bin/bash
```

Specifies that the script should execute using the Bash shell.

**2. Define the YARA Rule Path**

```bash
RULE="/home/perna/Desktop/detect_malware.yara"
```

Stores the location of the YARA rule file in the `RULE` variable.

**3. Define the Target Directory**

```bash
TARGET="/home/perna/Desktop"
```

Specifies the directory containing files that need to be scanned.

**4. Define the Log File**

```bash
LOG="/var/log/yara/yara_logs.log"
```

Specifies where detection results should be stored.

**5. Find Files**

```bash
find "$TARGET" -type f -print0
```

- `find`: Searches for files and directories.
- `"$TARGET"`: Specifies the directory to search.
- `-type f`: Selects regular files only.
- `-print0`: Separates filenames using null characters, which safely handles filenames containing spaces or special characters.

**6. Read Each Filename**

```bash
while IFS= read -r -d '' file
```

Reads each filename returned by `find` and stores it in the `file` variable.

**7. Execute YARA**

```bash
result=$(/usr/bin/yara "$RULE" "$file" 2>/dev/null)
```

- `/usr/bin/yara`: Executes YARA.
- `"$RULE"`: Specifies the detection rule.
- `"$file"`: Specifies the current file being scanned.
- `2>/dev/null`: Suppresses error messages.
- `result=$(...)`: Stores YARA's output in the `result` variable.

**8. Check Detection Results**

```bash
if [ -n "$result" ]; then
```

Checks whether the result variable contains any output.

- `if`: Starts a conditional statement.
- `-n`: Checks whether a string is non-empty.

If YARA finds a matching string, the condition becomes true.

**9. Store Detection Results**

```bash
echo "$(date '+%Y-%m-%d %H:%M:%S') - $result" >> "$LOG"
```

- `echo`: Prints the detection result.
- `date`: Adds the current timestamp.
- `$result`: Contains the YARA detection output.
- `>>`: Appends the result to the log file.
- `"$LOG"`: Specifies the destination log file.

**10. End the Conditional**

```bash
fi
```

Closes the `if` statement.

**11. End the Loop**

```bash
done
```

Ends the `while` loop after all files have been processed.

### Step 6: Give Execution Permission

**Command:**

```bash
sudo chmod +x /usr/local/bin/yara_directory_scan.sh
```

**Command Explanation:**

- `sudo`: Executes with administrator privileges.
- `chmod`: Changes file permissions.
- `+x`: Adds executable permission.
- `/usr/local/bin/yara_directory_scan.sh`: Specifies the scanning script.

### Step 7: Execute the Scanning Script

**Command:**

```bash
sudo /usr/local/bin/yara_directory_scan.sh
```

**Purpose:**

Executes the scanning script and checks regular files inside the configured directory.

If a file contains matching indicators, its detection result is appended to the YARA log file.

### Step 8: Schedule Directory Scanning Using Cron

Open the root crontab:

```bash
sudo crontab -e
```

Add:

```cron
*/2 * * * * /usr/local/bin/yara_directory_scan.sh
```

**Explanation:**

- `*/2`: Executes every two minutes.
- `/usr/local/bin/yara_directory_scan.sh`: Runs the automated directory scanning script.

Because this is root's crontab, the script can write to `/var/log/yara/` and read files accessible to root.

**Purpose:**

Automates scanning of files inside the configured directory without requiring us to know each filename in advance.

**Note:** This scans files in the specified directory and does not automatically scan the entire Ubuntu filesystem. Files created outside the target directory will not be checked.

---

## 12. Verify the Final Detection in Splunk

### Step 1: Open Splunk Enterprise

Navigate to:

**Splunk Enterprise → Search & Reporting**

### Step 2: Execute the Search Query

```spl
index=linux sourcetype="yara_logs"
```

### Step 3: Verify Detection Events

Check the results for:

- YARA rule name.
- Detected filename.
- Detection timestamp.
- Matching detection events.

If the newly created `malware_demo.sh` contains the configured indicators and is located inside the target directory, its detection should appear in Splunk after the scanning script executes and the forwarder collects the updated log.

---

## 13. Final Outcome

After completing this lab, we have:

1. Installed and configured YARA on Ubuntu Linux.
2. Created custom YARA rules for suspicious PowerShell-related indicators.
3. Scanned test files using YARA.
4. Created a dedicated YARA logging directory.
5. Stored detection results in a log file.
6. Configured Splunk Universal Forwarder to monitor YARA logs.
7. Forwarded detection events to Splunk Enterprise.
8. Automated scanning using Cron Jobs.
9. Configured directory-based scanning without depending on predefined filenames.
10. Created a Splunk alert to notify analysts about detection events.

**Final Result:**

YARA performs file-based pattern detection on Ubuntu Linux, while Splunk Enterprise provides centralized log collection, monitoring, investigation, and alerting.

This lab demonstrates a basic integration of file-based malware detection with SIEM monitoring.

---

## 14. Important Technical Limitations

- YARA detects patterns defined in rules; a string match alone does not prove that a file is malware.
- The current rule performs basic text matching and does not decode or analyze Base64-encoded PowerShell commands.
- The directory scanning script checks regular files only and scans the configured directory, not the entire system.
- Cron Jobs perform scheduled scans rather than real-time file monitoring.
- Splunk receives detection output only when the scan results are written to the monitored log file.
- If no YARA rule matches, the directory scanning script does not generate a detection event for that file.
- The same file may generate duplicate detection events during repeated scheduled scans.

**Lab Status:** YARA-based file detection and Splunk SIEM log integration.
