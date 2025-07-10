# Splunk SOC Lab: Day 10 - Automating Alert Escalation and Enhancing Dashboard Insights

## Overview
This repository documents Day 10 of a two-week Splunk lab series to prepare for the Certified SOC Analyst (CSA) exam. The lab focuses on automating alert escalation via email and enhancing the "SOC Web Monitoring" dashboard with a threat summary panel, addressing previous automation issues.

## Objectives
- Automate alert escalation via email.
- Enhance the dashboard with a threat summary panel.

## Environment
- **Host**: Windows laptop with Splunk Enterprise (`http://<new_windows_ip>:8000`).
- **VM**: Ubuntu Desktop (VirtualBox) with Splunk Universal Forwarder, Apache, and mailutils (named "abishekv-virtualbox", e.g., VM IP: `192.168.56.101`).
- **Tools**: Splunk, Apache, mailutils.

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
1. **Ensure Mail Configuration**:
   - Verify mail name:
     ```bash
     cat /etc/mailname  # Should show "abishekv-virtualbox"
     ```
   - Configure Gmail:
     ```bash
     sudo postconf -e "relayhost = [smtp.gmail.com]:587"
     sudo postconf -e "smtp_sasl_auth_enable = yes"
     sudo postconf -e "smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd"
     sudo postconf -e "smtp_sasl_security_options = noanonymous"
     sudo postconf -e "smtp_tls_security_level = encrypt"
     echo "[smtp.gmail.com]:587 your.email@gmail.com:your-app-password" | sudo tee /etc/postfix/sasl_passwd
     sudo chmod 600 /etc/postfix/sasl_passwd
     sudo postmap /etc/postfix/sasl_passwd
     sudo service postfix restart
     ```

2. **Create Automation Script**:
   - Create:
     ```bash
     sudo nano /usr/local/bin/escalate_alert_auto.sh
     ```
   - Add:
     ```bash
     #!/bin/bash
     TIMESTAMP=$(date -u +"%Y-%m-%d %H:%M:%S UTC")
     URI_PATH=$1
     UNIQUE_IPS=$2
     COUNT=$3
     EMAIL="your.email@gmail.com"
     SUBJECT="Automated Alert: Coordinated Login Attack Detected"
     BODY="Timestamp: $TIMESTAMP\nURI: $URI_PATH\nUnique IPs: $UNIQUE_IPS\nFailed Attempts: $COUNT"
     echo -e "$BODY" | mail -s "$SUBJECT" "$EMAIL"
     echo "[$TIMESTAMP] Automated email sent for URI: $URI_PATH, IPs: $UNIQUE_IPS, Attempts: $COUNT" >> /var/log/alerts.log
     ```
   - Make executable:
     ```bash
     sudo chmod +x /usr/local/bin/escalate_alert_auto.sh
     sudo cp /usr/local/bin/escalate_alert_auto.sh /opt/splunkforwarder/bin/
     sudo chmod +x /opt/splunkforwarder/bin/escalate_alert_auto.sh
     ```

3. **Update `alert_actions.conf`**:
   - Edit:
     ```bash
     sudo nano /opt/splunkforwarder/etc/apps/search/local/alert_actions.conf
     ```
   - Add:
     ```
     [escalate_alert_auto]
     is_custom = 1
     label = Escalate Alert Auto
     icon_path = alert_icon.png
     entity = [script]
     command = /opt/splunkforwarder/bin/escalate_alert_auto.sh $result.uri_path$ $result.unique_ips$ $result.count$
     ```
   - Restart Forwarder:
     ```bash
     sudo /opt/splunkforwarder/bin/splunk restart
     ```

4. **Update Alert**:
   - In **Alerts** > "Coordinated Login Attack Detection" > **Edit**.
   - Add "Escalate Alert Auto" to trigger actions.
   - Save.

5. **Test Automation**:
   - Run simulation: `sudo ./simulate_failed_logins.sh`.
   - Wait for next run (e.g., 09:00 PM IST).
   - Check email and `/var/log/alerts.log`.

6. **Enhance Dashboard**:
   - In **Dashboards** > "SOC Web Monitoring" > **Edit**.
   - Add Panel > Create New.
   - Search: `index=main sourcetype=access_combined status=401 | stats count as total_attempts, dc(clientip) as unique_ips by _time span=1h | eval threat_level=if(unique_ips > 2 AND total_attempts > 5, "High", "Low") | table _time, total_attempts, unique_ips, threat_level`.
   - Panel Title: "Threat Summary".
   - Format: Table.
   - Save.

## Adding Screenshots to Repository
1. **Save Screenshots**:
   - `day10_script_config.png`: Updated `alert_actions.conf`.
   - `day10_email_log.png`: Email output or log.
   - `day10_dashboard.png`: Updated dashboard.
2. **Add to GitHub**:
   - Create a folder:
     ```bash
     mkdir -p screenshots/day10
     mv day10_script_config.png day10_email_log.png day10_dashboard.png screenshots/day10/
     ```
   - Add and commit:
     ```bash
     git add screenshots/day10/*.png
     git commit -m "Add Day 10 screenshots"
     git push origin main
     ```
3. **Link in README**:
   - Embed screenshots (update when available).

## Results
- Successfully automated alert escalation via email.
- Enhanced dashboard with a threat summary panel.

## Troubleshooting
- Check Splunk logs: `index=_internal sourcetype=splunkd component=Script` for errors.
- Verify script permissions and Gmail credentials.

## Future Improvements
- Explore automated mitigation scripts.
- Integrate with SIEM tools.