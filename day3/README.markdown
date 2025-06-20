# Day 3: Splunk SOC Lab - Automated Threat Detection with Alerts

## Overview
On Day 3 of my Splunk journey for the Certified SOC Analyst (CSA) exam, I updated the Splunk Universal Forwarder to a new IP and troubleshooted alerts for excessive 404 errors and admin path requests.

## Objective
Fix Forwarder IP, configure alerts, and resolve triggering issues.

## Environment
- **Host**: Windows with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop in VirtualBox with Universal Forwarder, Apache.
- **Tools**: Splunk, Apache, Firefox, curl.

## Setup
1. **Updated Forwarder**:
   ```bash
   /opt/splunkforwarder/bin/splunk add forward-server <new_windows_ip>:9997 -auth admin:<password>
   /opt/splunkforwarder/bin/splunk restart
   ```
2. **Verified**:
   - Splunk: `"C:\Program Files\Splunk\bin\splunk" start`.
   - Forwarder, Apache: `/opt/splunkforwarder/bin/splunk start`, `sudo systemctl start apache2`.

## Steps
1. **Configured Alerts**:
   - Excessive 404 Errors:
     ```
     index=main sourcetype=access_combined status=404 | stats count | where count > 5
     ```
     - Alert: “Excessive 404 Errors”, every 5 minutes.
   - Admin Path Requests:
     ```
     index=main sourcetype=access_combined uri_path IN ("/admin*", "/wp-admin*")
     ```
     - Alert: “Admin Path Requests”, real-time.
2. **Simulated Attack**:
   ```bash
   #!/bin/bash
   for i in {1..30}; do
     curl -s http://localhost/nonexistent$i
     curl -s http://localhost/wp-admin
     sleep 0.2
   done
   ```
3. **Troubleshoot**:
   - Checked logs: `index=main sourcetype=access_combined`.
   - Adjusted 404 threshold to >5.
   - Verified alerts: `index=_internal "Excessive 404 Errors" OR "Admin Path Requests"`.

## Screenshots
- **404 Alert Config**: ![404 Alert](screenshots/day3_404_alert.png)
- **Admin Path Alert Config**: ![Admin Alert](screenshots/day3_admin_alert.png)
- **Triggered Alert**: ![Triggered Alert](screenshots/day3_triggered_alert.png)

## Findings
- Restored log flow with new IP.
- Alerts triggered after tuning threshold and increasing log volume.

## Next Steps
- Simulate SQL injection.
- Add threat intelligence.

## Repository
See [README.md](../../README.md).