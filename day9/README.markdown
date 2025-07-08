# Splunk SOC Lab: Day 9 - Enhancing Threat Detection with a New Alert and Script Action

## Overview
This repository documents Day 9 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab focuses on creating the "Coordinated Login Attack Detection" alert, adding a custom script action to log details, testing the automation, and updating the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Create a new alert to detect coordinated login attacks.
- Add a custom script action to log alert details.
- Test the automation with simulated traffic.
- Update the dashboard with real-time alert status.

## Environment
- **Host**: Windows laptop with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder and Apache (e.g., VM IP: `192.168.56.101`).
- **Tools**: Splunk, Apache, curl.

## Prerequisites
- Splunk Enterprise installed on Windows.
- Splunk Universal Forwarder and Apache installed on Ubuntu Desktop VM.
- VirtualBox with host-only network adapter configured.
- Apache logging to `/var/log/apache2/access.log` with `sourcetype=access_combined` from Day 7.
- "SOC Web Monitoring" dashboard from previous tasks.

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
     index=main sourcetype=access_combined
     ```
   - Ensure logs appear.

## Lab Steps
1. **Create Simulated Failed Login Data**:
   - On Ubuntu VM, create or verify:
     ```bash
     sudo nano /usr/local/bin/simulate_failed_logins.sh
     ```
   - Add:
     ```bash
     #!/bin/bash
     logfile="/var/log/apache2/access.log"
     ips=("192.168.56.102" "192.168.56.103" "192.168.56.104" "192.168.56.105" "192.168.56.106")
     for i in {1..100}; do
       ip=${ips[$((RANDOM % ${#ips[@]}))]}
       echo "$(date -u +'%d/%b/%Y:%H:%M:%S +0000') - - [$ip] \"POST /login HTTP/1.1\" 401 1234 \"-\" \"-\"" >> "$logfile"
       sleep 0.05
     done
     ```
   - Run: `sudo chmod +x /usr/local/bin/simulate_failed_logins.sh; sudo ./simulate_failed_logins.sh`.

2. **Create the "Coordinated Login Attack Detection" Alert**:
   - Go to **Settings** > **Searches, reports, and alerts** > **New Search**.
   - Name: "Coordinated Login Attack Detection".
   - Search: `index=main sourcetype=access_combined | stats dc(clientip) as unique_ips, count by uri_path | where unique_ips > 2 AND count > 5`.
   - Schedule: Cron Schedule with `0,15,30,45 * * * *` (every 15 minutes).
   - Trigger Condition: Number of results > 0.
   - Permissions: Shared in App > `search`.
   - Save as alert.

3. **Create Custom Script for Alert Action**:
   - On Ubuntu VM, create:
     ```bash
     sudo nano /usr/local/bin/log_alert.sh
     ```
   - Add:
     ```bash
     #!/bin/bash
     LOG_FILE="/var/log/alerts.log"
     TIMESTAMP=$(date -u +"%Y-%m-%d %H:%M:%S UTC")
     URI_PATH=$1
     UNIQUE_IPS=$2
     COUNT=$3
     echo "[$TIMESTAMP] Alert for URI: $URI_PATH, Unique IPs: $UNIQUE_IPS, Failed Attempts: $COUNT" >> "$LOG_FILE"
     ```
   - Run: `sudo chmod +x /usr/local/bin/log_alert.sh; sudo cp /usr/local/bin/log_alert.sh /opt/splunkforwarder/bin/; sudo chmod +x /opt/splunkforwarder/bin/log_alert.sh; sudo touch /var/log/alerts.log; sudo chmod 664 /var/log/alerts.log`.

4. **Configure Alert Action Manually**:
   - Edit `/opt/splunkforwarder/etc/apps/search/local/alert_actions.conf`:
     ```bash
     sudo nano /opt/splunkforwarder/etc/apps/search/local/alert_actions.conf
     ```
   - Add:
     ```
     [log_alert]
     is_custom = 1
     label = Log Alert
     icon_path = alert_icon.png
     entity = [script]
     command = /opt/splunkforwarder/bin/log_alert.sh $result.uri_path$ $result.unique_ips$ $result.count$
     ```
   - Restart Forwarder: `/opt/splunkforwarder/bin/splunk restart`.

5. **Test the Automation**:
   - Run simulation: `sudo ./simulate_failed_logins.sh`.
   - Wait 15 minutes (e.g., until 8:30 PM IST on Jul 8, 2025).
   - Check log: `cat /var/log/alerts.log` (currently empty, under troubleshooting).
   - Alert ran successfully from 8:15 PM to 8:45 PM IST with 26 results, but script action not logging.

6. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > Create New.
     - Search: `| inputlookup /var/log/alerts.log | table _time, uri_path, unique_ips, count`.
     - Panel Title: "Alert Status".
     - Format: Table.
     - Save (pending log data).

## Adding Screenshots to Repository
1. **Save Screenshots**:
   - `day9_alert_config.png`: Alert configuration.
   - `day9_alert_action_config.png`: `alert_actions.conf` content.
   - `day9_alert_log.png`: Script output log (to be updated).
   - `day9_dashboard.png`: Updated dashboard (to be updated).
2. **Add to GitHub**:
   - Create a folder:
     ```bash
     mkdir screenshots
     mv day9_alert_config.png day9_alert_action_config.png day9_alert_log.png day9_dashboard.png screenshots/
     ```
   - Add and commit:
     ```bash
     git add screenshots/*.png
     git commit -m "Add Day 9 screenshots"
     git push origin main
     ```
3. **Link in README**:
   - Embed screenshots (update when available).

## Results
- Successfully created the "Coordinated Login Attack Detection" alert, triggered at 8:15 PM IST with 26 results.
- Manually configured `alert_actions.conf` for script action.
- Script works manually (`/opt/splunkforwarder/bin/log_alert.sh "/login" 3 6`), but automation not logging (under investigation).
- Dashboard update pending log data.

## Troubleshooting
- Checked Splunk logs: `index=_internal sourcetype=splunkd component=Script` for errors.
- Verified permissions: `sudo chown splunk:splunk /var/log/alerts.log; sudo chmod 664 /var/log/alerts.log`.
- Next: Debug script action with `/tmp/debug.log` (see updated `log_alert.sh`).

## Future Improvements
- Resolve script action issue for automated logging.
- Add escalation scripts for Day 10.