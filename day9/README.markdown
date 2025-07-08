# Splunk SOC Lab: Day 9 - Enhancing Threat Detection with a New Alert and Script Action

## Overview
This repository documents Day 9 of  Splunk lab series . The lab focuses on creating the "Coordinated Login Attack Detection" alert, adding a custom script action to log details, testing the automation,. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Create a new alert to detect coordinated login attacks.
- Add a custom script action to log alert details.
- Test the automation with simulated traffic.


## Environment
- **Host**: Windows laptop with Splunk Enterprise .
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder and Apache 
- **Tools**: Splunk, Apache, curl.

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
   - Wait 15 minutes .
   - Check log: `cat /var/log/alerts.log` .
   - Alert ran successfully  with results, 
## Screenshots
- **Alert Configuration**:
  ![Alert Config](alert.png)
- **Alert action config**:
  ![Triggered Alert](alertaction.png)
