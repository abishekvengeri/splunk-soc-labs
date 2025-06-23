# Splunk SOC Lab: Day 6 - Detecting Cross-Site Scripting (XSS) Attacks

## Overview
This repository documents Day 6 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab focuses on simulating and detecting XSS attacks on an Apache web server using Splunk, with visualization in the "SOC Web Monitoring" dashboard. The setup uses Splunk Enterprise on Windows and a Splunk Universal Forwarder on an Ubuntu Desktop VM with Apache.

## Objectives
- Simulate XSS attacks on an Apache web server.
- Detect attacks using Splunk searches.
- Update the "SOC Web Monitoring" dashboard with visualization.

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
1. **Simulate XSS Attacks**:
   - On Ubuntu VM, create and run `simulate_xss_attack.sh`:
     ```bash
     #!/bin/bash
     logfile="/tmp/xss_attack_log.txt"
     echo "Starting XSS simulation at $(date)" > "$logfile"
     payloads=(
       "/?q=<script>alert('xss')</script>"
       "/?search=<img src=x onerror=alert('xss')>"
       "/?input=<script src='http://malicious.com/xss.js'></script>"
       "/?id=1 onload=alert('xss')"
       "/?param=<svg onload=alert('xss')>"
     )
     for i in {1..100}; do
       url="http://127.0.0.1:80${payloads[$((RANDOM % ${#payloads[@]}))]}"
       response=$(curl -s -o /dev/null -w "%{http_code}" "$url")
       echo "[$(date)] URL: $url | Status: $response" >> "$logfile"
       sleep 0.05
     done
     echo "Simulation complete. Check $logfile for details." >> "$logfile"
     ```
   - Run: `chmod +x simulate_xss_attack.sh; ./simulate_xss_attack.sh`.
   - Check: `cat /tmp/xss_attack_log.txt`.

2. **Create Splunk Search**:
   - In Splunk > **Search & Reporting**, run:
     ```spl
     index=main sourcetype=access_combined uri="*<script>*" OR uri="*<img*onerror=*" OR uri="*.js*" | table _time, clientip, uri, status | eval attack_type="XSS"
     ```
   - Save As > Report: "XSS Attempts" (Table).

3. **Update Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**:
     - Add Panel > From Report > "XSS Attempts".
     - Set Title: "XSS Attempts", Time Range: Last 15 minutes.
     - Save.

4. **Analyze Logs**:
   - Run:
     ```spl
     index=main sourcetype=access_combined uri="*<script>*" OR uri="*<img*onerror=*" OR uri="*.js*" | stats count by clientip, uri | sort -count
     ```
   - Save As > Report: "XSS Analysis" (Table).

## Adding Screenshots to Repository
1. **Save Screenshots**:
   - `day6_search.png`: XSS search results.
   - `day6_dashboard.png`: Updated dashboard with XSS panel.
   - `day6_log_analysis.png`: Log analysis of XSS patterns.
2. **Add to GitHub**:
   - Create a folder (e.g., `screenshots`):
     ```bash
     mkdir screenshots
     mv day6_search.png day6_dashboard.png day6_log_analysis.png screenshots/
     ```
   - Add and commit:
     ```bash
     git add screenshots/*.png
     git commit -m "Add Day 6 screenshots"
     git push origin main
     ```
3. **Link in README**:
   - Embed screenshots using Markdown (see Screenshots section).

## Results
- Successfully detected XSS attack patterns (e.g., `<script>`, `<img onerror>`) in Apache logs.
- Updated the "SOC Web Monitoring" dashboard to include XSS monitoring.
- Analyzed top client IPs and payloads for attack trends.

## Screenshots
- **XSS Search Results**:
  ![XSS Search](screenshots/day6_search.png)
- **Updated Dashboard**:
  ![Dashboard](screenshots/day6_dashboard.png)
- **Log Analysis**:
  ![Log Analysis](screenshots/day6_log_analysis.png)

## Notes
- Ensure Apache logs to `/var/log/apache2/access.log` with `combined` format.
- Use a broad time range (e.g., Last 60 minutes) if logs are delayed.
- This lab aligns with CSA exam topics: incident detection (26%) and response (29%).

## Future Improvements
- Create alerts for XSS attacks.
- Integrate threat intelligence feeds.