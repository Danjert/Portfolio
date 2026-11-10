# Module 2: Splunk Enterprise SIEM Deployment & Core Configuration

## 1. Executive Summary & Core Objectives

In this module, I focused on provisioning, configuring, and hardening the Ubuntu server hosting the Security Information and Event Management (SIEM) Splunk Enterprise within my SOC lab environment. 

Building upon the network segmentation established in Module 1, this phase involved:
* Provisioning an Ubuntu Server virtual machine with a dual-NIC network topology in VirtualBox.
* Applying system-level performance tuning and security hardening (creating a dedicated non-root user, allocating swap space, tuning kernel sysctl parameters, and raising file descriptor limits).
* Installing Splunk Enterprise and establishing systemd service persistence under an unprivileged user.
* Configuring network ingestion listeners (UDP `5140` for OPNsense remote syslog and TCP `9997` receiving port for future forwarders).
* Installing and configuring the Splunk Add-on for OPNsense (`TA-opnsense`) to extract, normalize, and parse raw firewall telemetry into Common Information Model (CIM) compliant fields.
* Validating end-to-end log data pipelines from the OPNsense border firewall into Splunk.

---

## 2. Ubuntu Server Provisioning & System Hardening

### 2.1 VirtualBox Hardware & Network Architecture
I deployed the Splunk Indexer and Search Head onto a single dedicated Ubuntu Server 22.04/24.04 LTS virtual machine (`Splunk-Indexer`).

* **Compute Allocation**: 4 vCPUs, 8 GB RAM, 80 GB Virtual Disk (Dynamic Allocation).
* **Dual-NIC Network Topology**:
  * **Adapter 1 (`enp0s3`)**: NAT (Temporary internet access for system updates and package installation).
  * **Adapter 2 (`enp0s8`)**: Internal Network / Host-Only Adapter (`10.0.100.0/24` Management Subnet).

### 2.2 Netplan Static IP & Interface Binding
To prevent dynamic IP shifts from disrupting connections and to ensure reliable management access, I assigned a static IP address (`10.0.100.10/24`) to `enp0s8`.

**File Configuration (`/etc/netplan/00-installer-config.yaml`)**:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      match:
        macaddress: 08:00:27:1a:c8:99
      set-name: enp0s8
      addresses:
        - 10.0.100.10/24
      routes:
        - to: default
          via: 10.0.100.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

I applied the Netplan settings using:
```bash
sudo netplan apply
ip a
```

### 2.3 Timezone & Clock Synchronization
To ensure log correlation and prevent timestamp misalignments when analyzing security events alongside perimeter firewall logs, I synchronized the system clock to Eastern Time:
```bash
sudo timedatectl set-timezone America/New_York
timedatectl status
```

### 2.4 Swap Space Allocation (4GB)
To guard against Out-Of-Memory (OOM) kernel panics during memory-intensive log parsing or heavy SPL search execution, I allocated a 4GB swap file:
```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make swap persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 2.5 Kernel Tuning & File Limits Configuration
Because Splunk Enterprise can be very resource intensive under specific circumstances, I increased system limits and tuned kernel parameters to prevent resource exhaustion.

1. **Increased File Descriptors & Process Limits (`/etc/security/limits.conf`)**:
   ```ini
   splunk           hard    nofile          64000
   splunk           soft    nofile          64000
   splunk           hard    nproc           20480
   splunk           soft    nproc           20480
   splunk           hard    fsize           unlimited
   splunk           soft    fsize           unlimited
   ```

2. **Kernel Parameters (`/etc/sysctl.conf`)**:
   ```ini
   vm.max_map_count = 262144
   net.core.somaxconn = 1024
   ```
   I reloaded the sysctl configuration using:
   ```bash
   sudo sysctl -p
   ```

### 2.6 Uncomplicated Firewall (UFW) Hardening
I configured host-based firewall rules to add another layer of security, strictly allowing only the traffic necessary for the server to perform its work:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp          # SSH Access
sudo ufw allow 8000/tcp        # Splunk Web UI
sudo ufw allow 5140/udp        # OPNsense Remote Syslog Ingestion
sudo ufw allow 9997/tcp        # Universal Forwarder Receiving Port
sudo ufw enable
sudo ufw status verbose
```

---

## 3. Dedicated Service User & Splunk Enterprise Installation

### 3.1 Downloading the Splunk Enterprise Package
I downloaded the Splunk Enterprise Linux installer archive (`.tgz`) from the official Splunk Portal directly into the `/tmp` directory:
```bash
#This URl would change depending on the version details
cd /tmp
wget -O splunk-9.x.x-linux-x86_64.tgz "https://download.splunk.com/products/splunk/releases/9.x.x/linux/splunk-9.x.x-linux-x86_64.tgz"
```

### 3.2 Non-Root Service Account & Extraction
To keep the infrastructure aligned with  least-privilege security principles, I created a dedicated system user (`splunk`) to run Splunk Enterprise without superuser privileges:
```bash
# Create dedicated splunk system group and user
sudo addgroup --system splunk
sudo adduser --system --ingroup splunk --disabled-password splunk

# Extract installer archive to /opt
sudo tar -xvzf /tmp/splunk-9.x.x-linux-x86_64.tgz -C /opt

# Transfer directory ownership to splunk user
sudo chown -R splunk:splunk /opt/splunk
```

### 3.3 Systemd Integration & Boot Persistence
I enabled systemd unit file generation under the unprivileged service user to ensure Splunk initializes automatically upon server boot:
```bash
sudo /opt/splunk/bin/splunk enable boot-start -user splunk --accept-license
sudo systemctl status splunk
```

---

## 4. Index Design & Data Input Configuration

### 4.1 Dedicated Index Creation (`opnsense`)
I created a dedicated `opnsense` index to optimize search performance, manage policies, and isolate network perimeter telemetry for a cleaner work environment.
1. In Splunk Web (`https://10.0.100.10:8000`), I navigated to **Settings** > **Indexes**.
2. I created a new event index named `opnsense` and saved the configuration.

### 4.2 UDP Syslog Listener (`5140`) for OPNsense
To capture syslog streams emitted by the OPNsense firewall, I configured a UDP data input via the Splunk Web interface:
1. Navigated to **Settings** > **Data Inputs** > **UDP**.
2. Created a new UDP input with the following specifications:
   * **Port:** `5140`
   * **Source type:** `opnsense:filterlog`
   * **Host:** `IP`
   * **Index:** `opnsense`

### 4.3 TCP Receiver Port (`9997`) Groundwork
To prepare the SIEM for endpoint telemetry onboarding via the Splunk Universal Forwarder, I enabled receiving port `9997`:
1. Navigated to **Settings** > **Forwarding and receiving**.
2. Under **Receive data**, clicked **Add new**.
3. Set the receiving port to `9997` and verified that its status showed as **Enabled**.

---

## 5. Splunk Add-on for OPNsense (`TA-opnsense`) Installation & Parsing Rationale

### 5.1 Architecture & Design Rationale
Raw OPNsense logs arrive over UDP as comma-separated BSD syslog strings. Without specialized parsing, Splunk treats these logs as unformatted text blocks, which could make the analysis hard to work with due to the need of writing complex manual regex extractions during search time.

To solve this, I installed the **Splunk Add-on for OPNsense** (`TA-opnsense`). Installing this Add-on provides the following benefits:
* **Automatic Field Extraction:** Automatically parses raw comma-separated `filterlog` entries into key-value pairs (e.g., `action`, `src_ip`, `dest_ip`, `src_port`, `dest_port`, `protocol`, `interface`).
* **CIM Compliance:** Normalizes extractions into the Splunk Common Information Model (CIM) Network Traffic datamodel, enabling integration with pre-built dashboards, correlation searches, and Enterprise Security.
* **Sourcetype Management:** Maps firewall events cleanly to the `opnsense:filterlog` sourcetype.

### 5.2 Add-on Installation & Configuration Process
1. **Downloading / Installing the TA:**
   * In Splunk Web, I navigated to **Apps** > **Find More Apps**.
   * I manually uploaded the app `.tgz` package via **Apps** > **Manage Apps** > **Install app from file**.
2. **Restarting `splunkd`:**
   * To ensure the new field extractions, props.conf, and transforms.conf took effect across the parsing pipeline, I restarted the Splunk daemon:
     ```bash
     sudo /opt/splunk/bin/splunk restart
     ```

---

## 6. OPNsense Remote Syslog Forwarding Integration

### 6.1 Remote Logging Setup in OPNsense Web GUI
With Splunk listening on UDP 5140 and parsing incoming events via the OPNsense TA, I configured the firewall to stream its logs:
1. Logged into the OPNsense Web GUI (`https://10.0.100.1`).
2. Navigated to **System** > **Settings** > **Logging / Targets**.
3. Created a new target entry with the following parameters:
   * **Transport**: `UDP(4)`
   * **Applications**: `firewall (filterlog)`
   * **Hostname**: `10.0.100.10`
   * **Port**: `5140`
4. Saved and applied the configuration.

---

## 7. Verification & Pipeline Testing

To ensure end-to-end data flow and verify that incoming telemetry was properly parsed, I performed standard diagnostic and verification steps:

### 7.1 UDP Port Connectivity & Packet Capture
1. **Socket Binding Verification:**
   I confirmed `splunkd` was actively listening on UDP port 5140:
   ```bash
   sudo ss -ulnp | grep 5140
   ```
2. **Zero-I/O Connectivity Check:**
   I checked remote port reachability using netcat:
   ```bash
   nc -zv -u 10.0.100.10 5140
   ```
3. **Live Packet Capture:**
   I ran `tcpdump` on the Ubuntu server to confirm raw UDP packets were arriving from OPNsense (`10.0.100.1`):
   ```bash
   sudo tcpdump -i any udp port 5140 -nn -c 5
   ```

### 7.2 OPNsense Synthetic Test Trigger
I triggered a manual log entry from the OPNsense shell to test the pipeline:
```bash
logger -p local0.info -t filterlog "OPNsense test firewall packet forwarding verification"
```

### 7.3 Splunk Search & Parsing Verification
I opened Splunk Web (`https://10.0.100.10:8000`) and ran the following query in Search & Reporting:
```spl
index=opnsense source="udp:5140"
```
**Results:** I confirmed that live logs were successfully populating the `opnsense` index with fully extracted fields (`action`, `src_ip`, `dest_ip`, `dest_port`) supplied by `TA-opnsense`.

---

## 8. Deployment State Summary

| Component | Configuration Parameter | Operational Status |
| :--- | :--- | :--- |
| **Splunk IP** | `10.0.100.10` (Static) | Active & Bound |
| **VirtualBox NICs** | `enp0s3` (NAT), `enp0s8` (`10.0.100.0/24`) | Active |
| **System Tuning** | 4GB Swap, `limits.conf` (64k nofile), `sysctl` | Applied |
| **Service Account** | `splunk` user (Non-root) | Systemd Boot-Start Enabled |
| **Index & Technology Add-on** | `opnsense` index, `TA-opnsense` installed | Active & Extracting CIM Fields |
| **Syslog Listener** | `UDP:5140` (`sourcetype=opnsense:filterlog`) | Receiving & Parsing Firewall Logs |
| **Forwarder Listener** | `TCP:9997` | Active (Prepped for Future Sources) |
| **UFW Rules** | Ports `22`, `8000`, `5140`, `9997` allowed | Active |