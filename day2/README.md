Day 2: Splunk SOC Lab - Real-Time Dashboard for Threat Monitoring
Overview
On Day 2 of my Splunk journey for the Certified SOC Analyst (CSA) exam, I created a Splunk dashboard to monitor Apache web logs in real-time, visualizing top client IPs, 404 errors, and admin path requests.
Objective
Build a Splunk dashboard for real-time SOC threat detection.
Environment

Host: Windows laptop with Splunk Enterprise (http://localhost:8000).
VM: Ubuntu Desktop in VirtualBox with Splunk Universal Forwarder and Apache.
Tools: Splunk Enterprise, Universal Forwarder, Apache, Firefox, curl.

Setup Instructions

Verified Splunk Enterprise: "C:\Program Files\Splunk\bin\splunk" start.
Confirmed Forwarder and Apache: /opt/splunkforwarder/bin/splunk start, sudo systemctl start apache2.

Lab Steps

Generated Web Traffic:

Visited http://localhost, requested /test123, /wp-admin.

Ran:
#!/bin/bash
urls=("http://localhost" "http://localhost/test123" "http://localhost/admin" "http://localhost/wp-admin" "http://localhost/nonexistent")
for i in {1..20}; do
  url=${urls[$((RANDOM % ${#urls[@]}))]}
  curl -s $url
  sleep 1
done




Created Searches:

Top Client IPs: index=main sourcetype=access_combined | top limit=10 clientip (saved as report, pie chart).
404 Errors Over Time: index=main sourcetype=access_combined status=404 | timechart count (saved as report, line chart).
Admin Paths: index=main sourcetype=access_combined uri_path IN ("/admin*", "/wp-admin*") | stats count by uri_path (saved as report, table).


Built Dashboard:

Created “SOC Web Monitoring” dashboard with panels: Top Client IPs (pie), 404 Errors Over Time (line), Admin Path Requests (table).



Screenshots

Dashboard Overview:


Top Client IPs:


404 Errors Over Time:



Findings

Top IPs showed frequent visitors, potential threat sources.
404 spikes indicated scanning activity.
Admin path requests (e.g., /wp-admin) flagged attacks.

Next Steps

Add alerts for automated detection.
Simulate specific attacks (e.g., SQL injection).

Repository
See README.md for the full journey.
