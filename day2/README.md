
#  Day 2: Splunk SOC Lab – Real-Time Dashboard for Threat Monitoring


On **Day 2** of my Splunk journey . I focused on creating a real-time monitoring dashboard using Splunk Enterprise. The objective was to visualize and analyze Apache web server logs to identify:

-  Top client IP addresses  
-  404 error spikes  
-  Access to admin paths (e.g., `/wp-admin`)  

---

##  Objective

Build a **real-time Splunk dashboard** to assist in **Security Operations Center (SOC)** threat detection using live Apache web traffic.

---

## 🧪 Lab Environment

| Component              | Details                                               |
|------------------------|-------------------------------------------------------|
|  Host Machine        | Windows 10 Laptop with Splunk Enterprise              |
| 🖥 VM                  | Ubuntu Desktop in VirtualBox                          |
|  Web Server          | Apache2                                               |
|  Log Forwarder       | Splunk Universal Forwarder                            |
|  Web Access Tools    | Firefox, `curl`                                       |
|  Splunk Web URL      | [http://localhost:8000](http://localhost:8000)        |

---

##  Setup Instructions


##  Simulate Web Traffic

I used a shell script to generate various Apache access logs by visiting valid, invalid, and sensitive endpoints.

### `traffic_simulation.sh`
```bash
#!/bin/bash
urls=("http://localhost" "http://localhost/test123" "http://localhost/admin" "http://localhost/wp-admin" "http://localhost/nonexistent")
for i in {1..20}; do
  url=${urls[$((RANDOM % ${#urls[@]}))]}
  curl -s $url
  sleep 1
done
```

---

##  Splunk Search Queries

###  Top Client IPs 
```spl
index=main sourcetype=access_combined 
| top limit=10 clientip
```

###  404 Errors Over Time 
```spl
index=main sourcetype=access_combined status=404 
| timechart count
```

###  Admin Path Access 
```spl
index=main sourcetype=access_combined uri_path IN ("/admin*", "/wp-admin*") 
| stats count by uri_path
```

---

##  Dashboard Creation

### Dashboard Name: `SOC Web Monitoring`

| Panel               | Visualization | Purpose                                |
|---------------------|---------------|----------------------------------------|
| Top Client IPs      | Pie Chart     | Detect frequent and possibly malicious visitors |
| 404 Errors Over Time| Line Chart    | Identify scanning activity or broken links |
| Admin Path Access   | Table         | Spot unauthorized attempts on admin endpoints |

---


### 🔍 Dashboard Overview  
![Dashboard Overview](dashboard_overview.mp4)


