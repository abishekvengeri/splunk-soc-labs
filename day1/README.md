Splunk SOC Labs: Day 1 - Log Ingestion and Threat Detection
Overview
This repository documents my two-week journey to master Splunk for Security Operations Center (SOC) tasks and prepare for the Certified SOC Analyst (CSA) exam. On Day 1, I configured the Splunk Universal Forwarder on an Ubuntu Desktop VM to send Apache web server logs to Splunk Enterprise on my Windows laptop. I performed searches to detect suspicious activity (e.g., 404 errors, admin path requests) and visualized results, simulating real-world SOC threat detection.
Day 1: Forwarding Apache Logs and Detecting Suspicious Activity
Objective
Set up log forwarding from Apache on an Ubuntu VM to Splunk Enterprise, analyze logs for potential threats (e.g., scanning attempts), and create visualizations to aid SOC analysis.
Environment

Host: Windows laptop running Splunk Enterprise (accessible at http://localhost:8000).
VM: Ubuntu Desktop in VirtualBox with Splunk Universal Forwarder and Apache web server.
Tools: Splunk Enterprise, Splunk Universal Forwarder, Apache, Firefox, curl.

Setup Instructions

Verify Splunk Enterprise (Windows):

Started Splunk: "C:\Program Files\Splunk\bin\splunk" start.
Enabled receiving on port 9997: Settings > Forwarding and Receiving > Receive Data.
Accessed Splunk at http://localhost:8000.


Verify Splunk Universal Forwarder (Ubuntu VM):

Started Forwarder: /opt/splunkforwarder/bin/splunk start.
Configured to forward to Windows host (e.g., 192.168.56.1:9997):/opt/splunkforwarder/bin/splunk add forward-server 192.168.56.1:9997 -auth admin:<password>




Install Apache (Ubuntu VM):

Installed Apache: sudo apt install apache2 -y.
Started service: sudo systemctl start apache2.
Verified at http://localhost in Firefox.


Configure Log Forwarding:

Monitored Apache logs: /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/access.log -sourcetype access_combined -index main.
Restarted Forwarder: /opt/splunkforwarder/bin/splunk restart.



Lab Steps

Generate Logs:

Visited http://localhost in Firefox and requested non-existent URLs (e.g., /test123, /wp-admin).
Used curl: curl http://localhost/nonexistent.


Splunk Searches:

Viewed all events:index=main sourcetype=access_combined


Detected 404 errors:index=main sourcetype=access_combined status=404


Identified admin path requests:index=main sourcetype=access_combined uri_path IN ("/admin*", "/wp-admin*")




Visualization:

Created a column chart for 404 errors by uri_path.
Saved as "404 Error URLs".



Screenshots

Search: 404 Errors:
Search: Admin Path Requests:
Visualization: 404 Errors by URI Path:

Findings

Identified multiple 404 errors, indicating potential scanning attempts.
Detected requests to /wp-admin, a common target for attackers.
Visualized 404 errors to prioritize suspicious URLs for SOC analysis.

Next Steps

Build a Splunk dashboard to monitor real-time threats.
Simulate attacks (e.g., XSS) and detect them in Splunk.

Repository Structure

screenshots/: Contains Splunk search and visualization images.
day1/: Notes and scripts from this lab.

Usage
Clone this repository and follow the setup instructions to replicate the lab. Use Splunk to analyze Apache logs and practice SOC threat detection.
Contact
Connect with me on LinkedIn for feedback or collaboration: [Insert LinkedIn Profile Link].
#Splunk #SOCAnalyst #Cybersecurity #CSACertification
