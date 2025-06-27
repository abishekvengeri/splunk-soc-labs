# Splunk SOC Lab: Day 8 - Creating Alerts for Detected Threats

## Overview
This repository documents Day 8 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab focuses on creating alerts for high/medium threat IPs detected via a threat intelligence feed, displaying them in the Triggered Alerts section (no email), and updating the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Create alerts for high/medium threat IPs in Splunk, shown only in Triggered Alerts.
- Test alerts with simulated malicious traffic.
- Update the dashboard with alert status visualization.

## Environment
- **Host**: Windows laptop with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder and Apache (e.g., VM IP: `192.168.56.101`).
- **Tools**: Splunk, Apache, `curl`.

## Prerequisites
- Splunk Enterprise installed on Windows.
- Splunk Universal Forwarder and Apache installed on Ubuntu Desktop VM.
- VirtualBox with host-only network adapter configured.
- Apache logging to `/var/log/apache2/access.log` with `sourcetype=access_combined` and updated `LogFormat` from Day 7.

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
1. **Create Alert (UI Only)**:
   - In Splunk > **Search & Reporting**, run:
     ```spl
     index=main sourcetype=access_combined
     | lookup threat_intel ip_address AS clientip OUTPUT threat_level, description
     | where threat_level="High" OR threat_level="Medium"
     | table _time, clientip, uri, status, threat_level, description
     ```
   - Click **Save As** > **Alert**.
   - Name: "High/Medium Threat IP Alert".
   - Description: "Alert for detecting high or medium threat IPs from threat_intel.csv in Apache logs (UI only)".
   - Permissions: Shared in App (search).
   - Trigger Conditions: Number of results > 0, Trigger: Once.
   - Trigger Actions: Add to Triggered Alerts (enabled), no email action.
   - Schedule: Run every 5 minutes (Cron: `*/5 * * * *`), Time Range: Last 15 minutes.
   - Save.

2. **Simulate Traffic to Trigger Alert**:
   - Update `simulate_xss_attack.sh` on Ubuntu VM:
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
   - Replace `192.168.56.101` with your VM’s actual IP.
   - Wait 5 minutes.

3. **Verify Alert**:
   - In Splunk > **Alerts**, check "High/Medium Threat IP Alert" under Triggered Alerts.
   - Note: No email should be sent.

4. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > Create New.
     - Search: `| rest /servicesNS/-/-/saved/searches | search title="High/Medium Threat IP Alert" | table title, next_scheduled_time, last_scheduled_time, triggered_count`.
     - Panel Title: "Alert Status".
     - Format: Table.
     - Save.

## Adding Screenshots to Repository
1. **Save Screenshots**:
   - `day8_alert_config.png`: Alert configuration.
   - `day8_alert_triggered.png`: Triggered alert.
   - `day8_dashboard.png`: Updated dashboard with alert status.
2. **Add to GitHub**:
   - Create a folder (e.g., `screenshots`):
     ```bash
     mkdir screenshots
     mv day8_alert_config.png day8_alert_triggered.png day8_dashboard.png screenshots/
     ```
   - Add and commit:
     ```bash
     git add screenshots/*.png
     git commit -m "Add Day 8 screenshots"
     git push origin main
     ```
3. **Link in README**:
   - Embed screenshots using Markdown (see Screenshots section).

## Results
- Successfully created and triggered an alert for high/medium threat IPs in the Triggered Alerts section.
- Confirmed no email notifications are sent as per configuration.
- Enhanced the dashboard with alert status monitoring.

## Screenshots
- **Alert Configuration**:
  ![Alert Config](screenshots/day8_alert_config.png)
- **Triggered Alert**:
  ![Triggered Alert](screenshots/day8_alert_triggered.png)
- **Updated Dashboard**:
  ![Dashboard](screenshots/day8_dashboard.png)

## Notes
- Email action is disabled to focus on UI-based alerting.
- This lab aligns with CSA exam topics: incident detection (26%) and response (29%).

## Future Improvements
- Automate alert responses with scripts.
- Integrate additional threat detection rules.