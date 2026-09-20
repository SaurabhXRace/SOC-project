# 🚀 Splunk Universal Forwarder Installation & Configuration Guide (Ubuntu/Debian)

This repository provides a complete, step-by-step guide for installing, configuring, and managing the **Splunk Universal Forwarder** on Linux systems (Ubuntu/Debian).
> ⚠️ **Note:** Please execute all commands in exact numerical sequence (Steps 1 to 8). Ensure Ubuntu is installed, open the terminal, and paste the commands accordingly.
---

## 📋 Prerequisites & Overview

- **Operating System:** Ubuntu / Debian Linux
- **Privileges:** `sudo` or `root` administrative access
- **Default Port:** `9997` (used by Splunk Indexers to receive forwarded logs)

---

## 🛠️ Step-by-Step Commands & Detailed Breakdown

### 1️⃣ Download the Splunk Universal Forwarder Package

This command uses `wget` to download the official Splunk Universal Forwarder `.deb` installer directly from Splunk's releases page and saves it locally under the filename `splunkforwarder.deb`.

```bash
wget -O splunkforwarder.deb "[https://download.splunk.com/products/universalforwarder/releases/9.4.2/linux/splunkforwarder-9.4.2-e9664af3d956-linux-amd64.deb](https://download.splunk.com/products/universalforwarder/releases/9.4.2/linux/splunkforwarder-9.4.2-e9664af3d956-linux-amd64.deb)"
```

---

### 2️⃣ Install the `.deb` Package

This command uses the Debian package manager (`dpkg`) with superuser privileges (`sudo`) to unpack and install the software. It sets up all required binary files, scripts, and default configuration folders under the `/opt/splunkforwarder` directory.

```bash
sudo dpkg -i splunkforwarder.deb
```

---

### 3️⃣ Initialize Splunk & Accept the License

This command runs the initial startup routine for the Splunk Universal Forwarder. 
- The `--accept-license` flag automatically accepts the Splunk End User License Agreement (EULA) without requiring manual interactive scrolling.
- **Important:** During this first execution, you will be prompted by the CLI to create an administrative username and password for local forwarder management.

```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

---

### 4️⃣ Enable Boot-Start Service

This command configures the system's service manager (such as `systemd` or `init`) to launch the Splunk Universal Forwarder automatically whenever the operating system reboots or starts up.

```bash
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

---

### 5️⃣ Manual Service Management (Start Command)

Use this command whenever you need to manually launch or start the Splunk Universal Forwarder service after it has been stopped.

```bash
sudo /opt/splunkforwarder/bin/splunk start
```

---

### 6️⃣ Configure Target Splunk Indexer / Receiving Server

This command configures the Universal Forwarder to direct all gathered log streams to your central Splunk Indexer or Heavy Forwarder.

> ⚠️ **Note:** Replace `192.168.33.36` with the actual IP address or hostname of your central Splunk Indexer. Port `9997` is the default receiving port configured on Splunk deployment servers.

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.33.36:9997
```

---

### 7️⃣ Verify Forwarder Connection Status

This command queries the local Splunk service and outputs a list of all configured receiving servers along with their current active connection status (e.g., active, connected, or unreachable).

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

---

### 8️⃣ Restart the Service

This command restarts the Splunk Universal Forwarder engine. It is essential to execute this after editing configuration files (like `inputs.conf` or `outputs.conf`) to ensure all updated settings are properly reloaded into memory.

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```
