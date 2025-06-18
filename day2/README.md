
# 🛡️ Day 2: Splunk SOC Lab – Real-Time Dashboard for Threat Monitoring

## 📘 Overview

On **Day 2** of my Splunk journey for the **Certified SOC Analyst (CSA)** exam, I focused on creating a real-time monitoring dashboard using Splunk Enterprise. The objective was to visualize and analyze Apache web server logs to identify:

- 👤 Top client IP addresses  
- ❌ 404 error spikes  
- 🔐 Access to admin paths (e.g., `/wp-admin`)  

---

## 🎯 Objective

Build a **real-time Splunk dashboard** to assist in **Security Operations Center (SOC)** threat detection using live Apache web traffic.

---

## 🧪 Lab Environment

| Component              | Details                                               |
|------------------------|-------------------------------------------------------|
| 💻 Host Machine        | Windows 10 Laptop with Splunk Enterprise              |
| 🖥️ VM                  | Ubuntu Desktop in VirtualBox                          |
| 🌐 Web Server          | Apache2                                               |
| 🔁 Log Forwarder       | Splunk Universal Forwarder                            |
| 🌍 Web Access Tools    | Firefox, `curl`                                       |
| 📊 Splunk Web URL      | [http://localhost:8000](http://localhost:8000)        |

---

## ⚙️ Setup Instructions

### ✅ Start Splunk Enterprise (on Host - Windows)
```bash
"C:\Program Files\Splunk\bin\splunk" start
```

### ✅ Start Universal Forwarder & Apache (on VM - Ubuntu)
```bash
/opt/splunkforwarder/bin/splunk start
sudo systemctl start apache2
```

---

## 🔁 Simulate Web Traffic

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

## 🔍 Splunk Search Queries

### 🔝 Top Client IPs (Pie Chart)
```spl
index=main sourcetype=access_combined 
| top limit=10 clientip
```

### ❌ 404 Errors Over Time (Line Chart)
```spl
index=main sourcetype=access_combined status=404 
| timechart count
```

### 🔐 Admin Path Access (Table)
```spl
index=main sourcetype=access_combined uri_path IN ("/admin*", "/wp-admin*") 
| stats count by uri_path
```

---

## 📊 Dashboard Creation

### Dashboard Name: `SOC Web Monitoring`

| Panel               | Visualization | Purpose                                |
|---------------------|---------------|----------------------------------------|
| Top Client IPs      | Pie Chart     | Detect frequent and possibly malicious visitors |
| 404 Errors Over Time| Line Chart    | Identify scanning activity or broken links |
| Admin Path Access   | Table         | Spot unauthorized attempts on admin endpoints |

---

## 📸 Screenshots

> 📎 *Replace image paths after uploading screenshots in your repo.*

### 🔍 Dashboard Overview  
![Dashboard Overview](screenshots/dashboard_overview.png)

### 📊 Top Client IPs  
![Top Client IPs](screenshots/top_client_ips.png)

### 📈 404 Errors Over Time  
![404 Errors](screenshots/404_errors.png)

---

## 🧠 Key Findings

- **Frequent Client IPs** revealed potential attackers or scanners repeatedly accessing the server.
- **Spikes in 404 errors** indicated probable automated scanning tools or broken scripts.
- **Access to `/admin`, `/wp-admin`** endpoints highlighted suspicious probing behavior.

---

## 🚀 Next Steps

- ⚠️ **Add Splunk alerts** for real-time notification on 404 spikes or admin access.
- 🛠️ **Simulate attacks** like SQL injection, XSS to enhance SOC response strategy.
- 🌍 **Enhance dashboard** with GeoIP, user agents, and more advanced analytics.

---

## 📁 Directory Structure

```
SOC-Splunk-Lab-Day2/
├── traffic_simulation.sh
├── screenshots/
│   ├── dashboard_overview.png
│   ├── top_client_ips.png
│   └── 404_errors.png
└── README.md
```

---

## 🙌 Final Thoughts

This dashboard represents a foundational step in operationalizing Splunk for real-time threat detection in SOC environments. It bridges basic log monitoring with actionable insights—empowering faster incident response.

> ✅ Stay tuned for **Day 3** where I’ll explore **alerting and automation with Splunk**.

---
