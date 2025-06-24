# Splunk SOC Lab: Day 7 - Integrating Threat Intelligence Feed

## Overview
This repository documents Day 7 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab focuses on integrating a mock threat intelligence feed into Splunk to detect malicious IPs in Apache logs, with visualization in the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Import a mock threat intelligence feed into Splunk.
- Correlate threat data with Apache logs to detect malicious IPs.
- Update the "SOC Web Monitoring" dashboard with threat intel visualization.

## Environment
- **Host**: Windows laptop with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder and Apache (e.g., VM IP: `192.168.56.101`).
- **Tools**: Splunk, Apache, `curl`.

## Prerequisites
- Splunk Enterprise installed on Windows.
- Splunk Universal Forwarder and Apache installed on Ubuntu Desktop VM.
- VirtualBox with host-only network adapter configured.
- Apache logging to `/var/log/apache2/access.log` with `sourcetype=access_combined`.

## Setup Instructions
1. **Start Splunk Enterprise (Windows)**:
   ```bash
   "C:\Program Files\Splunk\bin\splunk" start
   ```
2. **Start Forwarder and Apache (Ubuntu VM)**:
   ```bash
   /opt/splunkforwarder/bin/splunk start
   sudo systemctl start apache2
   ```
3. **Verify Log Ingestion**:
   - In Splunk, run:
     ```spl
     index=main sourcetype=access_combined clientip=*
     ```
   - Ensure logs from `/var/log/apache2/access.log` appear.

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
   - Verify status: `sudo systemctl status apache2`.

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
       url="http://192.168.56.101:80${payloads[$((RANDOM % ${#payloads[@]}))]}"
       response=$(curl -s -H "X-Forwarded-For: $ip" -o /dev/null -w "%{http_code}" "$url")
       echo "[$(date)] IP: $ip | URL: $url | Status: $response" >> "$logfile"
       sleep 0.05
     done
     echo "Simulation complete. Check $logfile for details." >> "$logfile"
     ```
   - Run: `chmod +x simulate_xss_attack.sh; ./simulate_xss_attack.sh`.
   - Replace `192.168.56.101` with your VM’s actual IP if different.
   - Check: `tail -f /var/log/apache2/access.log` should show `192.168.56.102` or `10.0.0.5` as the first field.

4. **Create Splunk Search**:
   - In Splunk > **Search & Reporting**, run:
     ```spl
     index=main sourcetype=access_combined
     | lookup threat_intel ip_address AS clientip OUTPUT threat_level, description
     | where threat_level="High" OR threat_level="Medium"
     | table _time, clientip, uri, status, threat_level, description
     ```
   - Set time range to Last 15 minutes.
   - If no results, try:
     ```spl
     index=main sourcetype=access_combined
     | rename X-Forwarded-For AS clientip
     | lookup threat_intel ip_address AS clientip OUTPUT threat_level, description
     | where threat_level="High" OR threat_level="Medium"
     | table _time, clientip, uri, status, threat_level, description
     ```
   - Save As > Report: "Threat Intel Matches" (Table).

5. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > From Report > "Threat Intel Matches".
     - Set Title: "Threat Intel Alerts", Time Range: Last 15 minutes.
     - Save.

## Adding Screenshots to Repository
1. **Save Screenshots**:
   - `day7_lookup.png`: Lookup file configuration.
   - `day7_search.png`: Search results with threat intel.
   - `day7_dashboard.png`: Updated dashboard with threat intel panel.
2. **Add to GitHub**:
   - Create a folder (e.g., `screenshots`):
     ```bash
     mkdir screenshots
     mv day7_lookup.png day7_search.png day7_dashboard.png screenshots/
     ```
   - Add and commit:
     ```bash
     git add screenshots/*.png
     git commit -m "Add Day 7 screenshots"
     git push origin main
     ```
3. **Link in README**:
   - Embed screenshots using Markdown (see Screenshots section).

## Results
- Correlated Apache logs with threat intelligence to identify malicious IPs.
- Updated the "SOC Web Monitoring" dashboard to display high/medium threat alerts.
- Improved detection capabilities for CSA exam prep.

## Screenshots
- **Lookup Configuration**:
  ![Lookup Config](screenshots/day7_lookup.png)
- **Search Results**:
  ![Search Results](screenshots/day7_search.png)
- **Updated Dashboard**:
  ![Dashboard](screenshots/day7_dashboard.png)

## Notes
- Updated Apache `LogFormat` to log `X-Forwarded-For` as the client IP.
- Ensure the VM IP (`192.168.56.101`) in the script matches your setup.
- This lab aligns with CSA exam topics: incident detection (26%) and response (29%).

## Future Improvements
- Automate threat intel updates with a scheduled script.
- Create alerts for threat intel matches.