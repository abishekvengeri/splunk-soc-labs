# Splunk SOC Lab: Day 5 - Creating Alerts for SQL Injection Detection

## Overview
This repository documents Day 5 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab focuses on configuring Splunk alerts to automatically detect SQL injection attacks on an Apache web server and integrating them into the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Configure a Splunk alert for automated SQL injection detection.
- Verify alert triggering with simulated attacks.
- Update the "SOC Web Monitoring" dashboard with alert details.

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
     index=main sourcetype=access_combined uri=*
     ```
   - Ensure logs from `/var/log/apache2/access.log` appear.

## Lab Steps
1. **Simulate SQL Injection**:
   - On Ubuntu VM, create and run `simulate_sql_injection_v2.sh`:
     ```bash
     #!/bin/bash
     logfile="/tmp/sql_injection_log.txt"
     echo "Starting SQL injection simulation at $(date)" > "$logfile"
     payloads=(
       "/?id=1%20OR%201=1"
       "/?query=UNION%20SELECT%201,2,3%20FROM%20users"
       "/?id=1';%20DROP%20TABLE%20users--"
       "/?input=1%20AND%201=1"
       "/?search=1%20OR%20'a'='a'"
     )
     for i in {1..100}; do
       url="http://127.0.0.1:80${payloads[$((RANDOM % ${#payloads[@]}))]}"
       response=$(curl -s -o /dev/null -w "%{http_code}" "$url")
       echo "[$(date)] URL: $url | Status: $response" >> "$logfile"
       sleep 0.05
     done
     echo "Simulation complete. Check $logfile for details." >> "$logfile"
     ```
   - Run: `chmod +x simulate_sql_injection_v2.sh; ./simulate_sql_injection_v2.sh`.
   - Check: `cat /tmp/sql_injection_log.txt`.

2. **Create Splunk Alert**:
   - In Splunk > **Search & Reporting**, run:
     ```spl
     index=main sourcetype=access_combined uri="*id=*" OR uri="*query=*" OR uri="*search=*" | table _time, clientip, uri, status | eval attack_type="SQL Injection"
     ```
   - Save As > Alert:
     - Title: "SQL Injection Alert"
     - Description: "Detects SQL injection attempts in Apache access logs based on URI patterns (e.g., *id=*, *query=*, *search=*). Triggers when results exceed 0."
     - Alert Type: Scheduled
     - Schedule: Run every 5 minutes, Time Range: Last 5 minutes
     - Trigger Condition: Number of Results > 0
     - Trigger Actions: Add to Triggered Alerts (Severity: High); (optional) Send Email to `<your-email>` if SMTP configured
     - Permissions: Shared in App
     - Save.

3. **Verify Alert**:
   - Wait 5–10 minutes (e.g., until 08:17 PM IST) or rerun `simulate_sql_injection_v2.sh`.
   - Check **Alert** > **Triggered Alerts** for "SQL Injection Alert".

4. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > New > Search:
       ```spl
       | search sourcetype=alerts "SQL Injection Alert" | table _time, clientip, uri, status
       ```
     - Set Title: "SQL Injection Alerts", Time Range: Last 24 hours.
     - Save.

## Adding Screenshots to Repository
1. **Save Screenshots**:
   - `day5_alert_config.png`: Alert configuration.
   - `day5_alert_triggered.png`: Triggered alert.
   - `day5_dashboard.png`: Updated dashboard with alert panel.
2. **Add to GitHub**:
   - Create a folder (e.g., `screenshots`):
     ```bash
     mkdir screenshots
     mv day5_alert_config.png day5_alert_triggered.png day5_dashboard.png screenshots/
     ```
   - Add and commit:
     ```bash
     git add screenshots/*.png
     git commit -m "Add Day 5 screenshots"
     git push origin main
     ```
3. **Link in README**:
   - Embed screenshots using Markdown (see Screenshots section).

## Results
- Configured a Splunk alert to automatically detect SQL injection attempts.
- Verified alert triggering with simulated attacks.
- Updated the "SOC Web Monitoring" dashboard with alert details.

## Screenshots
- **Alert Configuration**:
  ![Alert Config](screenshots/day5_alert_config.png)
- **Triggered Alert**:
  ![Triggered Alert](screenshots/day5_alert_triggered.png)
- **Updated Dashboard**:
  ![Dashboard](screenshots/day5_dashboard.png)

## Notes
- Ensure Apache logs to `/var/log/apache2/access.log` with `combined` format.
- Use a broad time range (e.g., Last 60 minutes) if logs are delayed.
- This lab aligns with CSA exam topics: incident detection (26%) and response (29%).

## Future Improvements
- Simulate additional attack types (e.g., XSS).
- Integrate threat intelligence feeds.