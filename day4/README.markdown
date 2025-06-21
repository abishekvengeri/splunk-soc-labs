# Splunk SOC Lab: Day 4 - Detecting SQL Injection Attacks

## Overview
This repository documents Day 4 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab simulates a SQL injection attack on an Apache web server, detects it using Splunk searches, and visualizes results in the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Simulate SQL injection attacks on an Apache web server.
- Detect attacks using Splunk searches.
- Update the "SOC Web Monitoring" dashboard for visualization.
- Document findings for CSA exam preparation.

## Environment
- **Host**: Windows laptop with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder and Apache.
- **Tools**: Splunk, Apache, `curl`.

## Prerequisites
- Splunk Enterprise installed on Windows.
- Splunk Universal Forwarder and Apache installed on Ubuntu Desktop VM.
- VirtualBox with host-only network adapter (e.g., VM IP: `192.168.56.101`).
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
   - In Splunk, run: `index=main sourcetype=access_combined uri=*`.
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

2. **Create Splunk Search**:
   - In Splunk > **Search & Reporting**, run:
     ```spl
     index=main sourcetype=access_combined uri="*id=*" OR uri="*query=*" OR uri="*search=*" | table _time, clientip, uri, status | eval attack_type="SQL Injection"
     ```
   - Save As > Report: "SQL Injection Attempts" (Table).

3. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > From Report > "SQL Injection Attempts".
     - Set Title: "SQL Injection Attempts", Time Range: Last 15 minutes.
     - Save.

4. **Analyze Logs**:
   - Run:
     ```spl
     index=main sourcetype=access_combined uri="*id=*" OR uri="*query=*" OR uri="*search=*" | stats count by clientip, uri | sort -count
     ```
   - Save As > Report: "SQL Injection Analysis" (Table).

## Results
- Successfully detected SQL injection patterns (e.g., `id=1 OR 1=1`, `query=UNION SELECT`) in Apache logs.
- Visualized attacks in the "SOC Web Monitoring" dashboard.
- Analyzed top attacking IPs and payloads.

## Screenshots
- `day4_sql_search.png`: SQL injection search results.
- `1.png`: Updated dashboard with SQL injection panel.
- `2.png`: Log analysis of attack patterns.

## Notes
- Ensure Apache logs to `/var/log/apache2/access.log` with `combined` format.
- Use a broad time range (e.g., Last 60 minutes) if logs are delayed.
- This lab aligns with CSA exam topics: incident detection (26%) and response (29%).

## Future Improvements
- Create alerts for SQL injection attempts.
- Integrate threat intelligence feeds.