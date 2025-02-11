# **Root Cause Analysis (RCA) Guide**

## **Introduction**
Root Cause Analysis (RCA) is a structured approach used in Linux troubleshooting to identify the underlying cause of system failures, performance issues, and unexpected behaviors. By diagnosing and resolving the root cause rather than just treating symptoms, administrators can prevent recurring issues and improve system stability.

---

## **1. The RCA Process in Linux Troubleshooting**

### **Step 1: Identify the Problem (Symptom Analysis)**
- **Observe:** Identify unexpected behaviors (e.g., high CPU, network failures, service crashes).
- **Ask questions:** When did the issue start? What changed? Who reported it?
- **Reproduce the issue:** If possible, recreate the failure in a controlled environment.

### **Step 2: Gather Data**
Use system logs, monitoring tools, and diagnostic commands to collect evidence.

#### **Useful Commands:**
```bash
dmesg | tail -50        # View recent kernel logs
journalctl -xe          # Check system logs for failures
systemctl status <service> # Check the status of a service
ps aux --sort=-%cpu     # Find high CPU usage processes
netstat -tulnp          # Check open ports and listening services
df -h                   # Check disk space usage
```

---

## **2. Common RCA Scenarios with Examples**

### **2.1 High CPU Usage**
#### **Symptoms:**
- System is slow or unresponsive.

#### **Diagnosis:**
```bash
top  # View real-time CPU utilization
ps aux --sort=-%cpu | head -5  # Identify top CPU-consuming processes
strace -p <PID>  # Trace system calls of a process
```

#### **Root Cause & Resolution:**
- A new deployment introduced an infinite loop in the application code.
- **Fix:** Roll back the deployment and debug the code.

---

### **2.2 Service Downtime**
#### **Symptoms:**
- Web application is down, returning a `502 Bad Gateway` error.

#### **Diagnosis:**
```bash
systemctl status nginx
journalctl -u nginx --since "1 hour ago"
```

#### **Root Cause & Resolution:**
- The SSL certificate expired, and automatic renewal failed.
- **Fix:** Renew the certificate manually and fix the renewal script.
```bash
certbot renew --force-renewal
systemctl restart nginx
```

---

### **2.3 Disk Space Full**
#### **Symptoms:**
- Application crashes due to lack of space.

#### **Diagnosis:**
```bash
df -h  # Check disk usage
du -sh /var/log  # Check log size
```

#### **Root Cause & Resolution:**
- Excessive logs accumulated.
- **Fix:** Implement log rotation using `logrotate`.

---

## **3. Setting Up SSL Expiration Alerts**
To avoid downtime due to expired SSL certificates, set up an alert system.

### **Using OpenSSL to Check SSL Expiry:**
```bash
openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -enddate
```

### **Automate SSL Expiry Monitoring with a Cron Job:**
Create a script (`ssl_check.sh`):
```bash
#!/bin/bash
EXPIRY_DATE=$(openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
EXPIRY_TIMESTAMP=$(date -d "$EXPIRY_DATE" +%s)
CURRENT_TIMESTAMP=$(date +%s)
DAYS_LEFT=$(( (EXPIRY_TIMESTAMP - CURRENT_TIMESTAMP) / 86400 ))
if [ "$DAYS_LEFT" -lt 10 ]; then
    echo "SSL certificate for example.com expires in $DAYS_LEFT days!" | mail -s "SSL Expiry Alert" admin@example.com
fi
```

### **Schedule the Check in Crontab:**
```bash
crontab -e
```
Add the following line to check SSL expiry daily:
```bash
0 0 * * * /path/to/ssl_check.sh
```

---

## **4. Preventing Future Issues**
- **Implement Monitoring:** Use tools like Prometheus, Grafana, or Nagios.
- **Automate Backups & Alerts:** Regular snapshots and email notifications.
- **Apply Security Patches:** Keep software up to date to prevent vulnerabilities.

---

## **Conclusion**
Root Cause Analysis is a critical skill for Linux administrators. By systematically identifying, diagnosing, and resolving issues, you can maintain system reliability and prevent recurring problems. Document your RCA findings to build a knowledge base for future troubleshooting.

For more best practices, refer to Linux documentation and system logs regularly.

