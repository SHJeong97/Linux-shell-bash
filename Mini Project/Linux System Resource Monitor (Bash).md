# 🖥️ Linux System Resource Monitor (Bash)

## 📘 Project Overview
This project demonstrates the creation of a **Linux system monitoring tool** using a Bash shell script.  
The script continuously monitors **CPU, memory, and disk usage** in real time and triggers alerts when predefined thresholds are exceeded.

The goal of this project is to strengthen foundational Linux scripting skills while building a practical, extensible monitoring utility commonly used in system administration and security operations.

---

## 🎯 Objectives
- Monitor system resource utilization using native Linux tools
- Implement threshold-based alerting logic
- Use Bash functions, loops, and command substitution
- Display real-time system statistics in a terminal interface

---

## 🛠️ Tools & Technologies
- **Language:** Bash
- **Commands Used:** `top`, `free`, `df`, `awk`, `grep`, `tput`
- **Environment:** Linux (Ubuntu-based lab)

---

## ⚙️ Core Features
- Real-time monitoring of:
  - CPU usage
  - Memory usage
  - Disk usage
- Configurable threshold values
- Visual alerting using colored terminal output
- Continuous monitoring loop with periodic refresh

---

## 🧠 Script Design

### Threshold Configuration
```bash
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80
```
Defines alert thresholds (in percentage) for system resources.

### Alert Function
```bash
send_alert() {
  echo "$(tput setaf 1)ALERT: $1 usage exceeded threshold! Current value: $2%$(tput sgr0)"
}
```
Centralized alert logic


Uses colored output to highlight critical conditions


Easily extendable for logging or notifications



### CPU Monitoring Logic
```bash
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
cpu_usage=${cpu_usage%.*}

if ((cpu_usage >= CPU_THRESHOLD)); then
  send_alert "CPU" "$cpu_usage"
fi
```
Extracts real-time CPU usage


Combines user and system CPU percentages


Compares against defined threshold



### Memory Monitoring Logic
```bash
memory_usage=$(free | awk '/Mem/ {printf("%3.1f", ($3/$2) * 100)}')
memory_usage=${memory_usage%.*}

if ((memory_usage >= MEMORY_THRESHOLD)); then
  send_alert "Memory" "$memory_usage"
fi
```
Calculates memory usage percentage


Uses free and awk for accurate metrics



### Disk Monitoring Logic
```bash
disk_usage=$(df -h / | awk '/\// {print $(NF-1)}')
disk_usage=${disk_usage%?}

if ((disk_usage >= DISK_THRESHOLD)); then
  send_alert "Disk" "$disk_usage"
fi
```
Monitors root filesystem usage


Extracts percentage cleanly for comparison



### Continuous Monitoring Loop
```bash
while true; do
  clear
  echo "Resource Usage:"
  echo "CPU: $cpu_usage%"
  echo "Memory: $memory_usage%"
  echo "Disk: $disk_usage%"
  sleep 2
done
```
Refreshes output every 2 seconds


Provides real-time visibility into system health



## 🔐 Security & Operations Relevance
This project reflects real-world skills used in:
System administration


SOC monitoring


Infrastructure health checks


Incident response preparation


Baseline system performance analysis


Monitoring system resources is a foundational defensive capability for detecting:
Resource exhaustion attacks


Misbehaving processes


Early indicators of compromise



## 🚀 Possible Enhancements
Log alerts to a file


Email or Slack notifications


Per-process monitoring


Historical usage tracking


Integration with cron or systemd


Threshold configuration via CLI arguments



## 📌 Key Takeaway
This project shows how simple Bash scripting, combined with native Linux utilities, can create powerful monitoring tools.
 It reinforces the importance of visibility, automation, and proactive alerting in secure and reliable system operations.

