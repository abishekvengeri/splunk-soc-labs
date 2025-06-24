# Splunk SOC Lab: Day 7 - Integrating Threat Intelligence Feed

## Overview
This repository documents Day 7 of a two-week Splunk lab series. The lab focuses on integrating a mock threat intelligence feed into Splunk to detect malicious IPs in Apache logs, with visualization in the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Import a mock threat intelligence feed into Splunk.
- Correlate threat data with Apache logs to detect malicious IPs.
- Update the "SOC Web Monitoring" dashboard with threat intel visualization.

## Prerequisites
- Splunk Enterprise installed on Windows.
- Splunk Universal Forwarder and Apache installed on Ubuntu Desktop VM.
- VirtualBox with host-only network adapter configured.

## Lab Steps
1. **Update Apache Log Format to Include `X-Forwarded-For`**:
   - Edit `/etc/apache2/apache2.conf` on the Ubuntu VM:
     - Replace the `combined` `LogFormat`:
       ```
       LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
       ```
       with:
       ```
       LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
       ```
   - Save and restart Apache:
     ```bash
     sudo systemctl restart apache2
     ```

2. **Create Mock Threat Intelligence Lookup**:
   - On Windows, create `threat_intel.csv` in `C:\Program Files\Splunk\etc\apps\search\lookups`:
     ```
     ip_address,threat_level,description
     192.168.56.102,High,Malicious Botnet
     10.0.0.5,Medium,Known Phishing Source
     172.16.1.10,Low,Suspicious Activity
     ```
   - In Splunk > **Settings** > **Lookups** > **Add New** (Lookup table files):
     - Destination app: `search`
     - Lookup file: Upload `threat_intel.csv`
     - Destination filename: `threat_intel.csv`
     - Save.
   - In **Lookups** > **Add New** (Lookup definitions):
     - Name: `threat_intel`
     - Type: File-based
     - Lookup file: `threat_intel.csv` (from dropdown)
     - Save.
   - Set permissions: **Permissions** > Read for `All apps`, Write for `admin`, Save.

3. **Simulate Traffic with Malicious IPs**:
   - On Ubuntu VM, update `simulate_xss_attack.sh`:
     ```bash
     #!/bin/bash
     logfile="/tmp/xss_attack_log.txt"
     echo "Starting XSS simulation with malicious IPs at $(date)" > "$logfile"
     payloads=(
       "/?q=<script>alert('xss')</script>"
       "/?search=<img src=x onerror=alert('xss')>"
     )
     malicious_ips=("192.168.56.102" "10.0.0.5")
     for i in {1..50}; do
       ip=${malicious_ips[$((RANDOM % ${#malicious_ips[@]}))]}
       url="http://VM IP:80${payloads[$((RANDOM % ${#payloads[@]}))]}"
       response=$(curl -s -H "X-Forwarded-For: $ip" -o /dev/null -w "%{http_code}" "$url")
       echo "[$(date)] IP: $ip | URL: $url | Status: $response" >> "$logfile"
       sleep 0.05
     done
     echo "Simulation complete. Check $logfile for details." >> "$logfile"
     ```
   - Run: `chmod +x simulate_xss_attack.sh; ./simulate_xss_attack.sh`.
   - Replace `VM IP` with your VM’s actual IP if different.
   - Check: `tail -f /var/log/apache2/access.log` should show `192.168.56.102` or `10.0.0.5` as the first field.

4. **Create Splunk Search**:
   - In Splunk > **Search & Reporting**, run:
     ```spl
     index=main sourcetype=access_combined
     | lookup threat_intel ip_address AS clientip OUTPUT threat_level, description
     | where threat_level="High" OR threat_level="Medium"
     | table _time, clientip, uri, status, threat_level, description
     ```
   - Save As > Report: "Threat Intel Matches" .

5. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > From Report > "Threat Intel Matches".
     - Set Title: "Threat Intel Alerts", Time Range: Last 15 minutes.
     - Save.

## Results
- Correlated Apache logs with threat intelligence to identify malicious IPs.
- Updated the "SOC Web Monitoring" dashboard to display high/medium threat alerts.
- Improved detection capabilities for CSA exam prep.

## Screenshots
- **Lookup Configuration**:
  ![Lookup Config](lok.png)
- **Search Results**:
  ![Search Results](search.png)
- **Updated Dashboard**:
  ![Dashboard](dashboard.png)
