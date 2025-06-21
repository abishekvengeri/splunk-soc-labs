# Splunk SOC Lab: Day 4 - Detecting SQL Injection Attacks

## Overview
This repository documents Day 4 of a two-week Splunk lab series. The lab simulates a SQL injection attack on an Apache web server, detects it using Splunk searches, and visualizes results in the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

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
- VirtualBox with host-only network adapter .

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
