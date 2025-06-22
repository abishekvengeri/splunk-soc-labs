# Splunk SOC Lab: Day 5 - Creating Alerts for SQL Injection Detection

## Overview
This repository documents Day 5 of a two-week Splunk lab series. The lab focuses on configuring Splunk alerts to automatically detect SQL injection attacks on an Apache web server . The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Environment
- **Host**: Windows laptop with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder and Apache (e.g., VM IP: `192.168.56.101`).
- **Tools**: Splunk, Apache, `curl`.


## Lab Steps
1. **Simulate SQL Injection**:
   - On Ubuntu VM, create and run `simulate_sql_injection.sh`:
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


## Results
- Configured a Splunk alert to automatically detect SQL injection attempts.
- Verified alert triggering with simulated attacks.


- **Alert Configuration**:
  ![Alert Config](1.png)
- **Triggered Alert**:
  ![Triggered Alert](2.png)
