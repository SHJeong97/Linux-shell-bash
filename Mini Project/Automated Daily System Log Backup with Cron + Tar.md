# 🗓️ Automated Daily System Log Backup with Cron + Tar

## 📘 Project Overview
Operational log retention is a daily responsibility in real environments—whether for troubleshooting, audits, or incident response.  
This project implements a **scheduled, reliable daily backup** for key system logs using **cron** and **tar**, producing a date-stamped archive every night and overwriting safely when a backup already exists.

The end result is a lightweight, repeatable backup pattern that can be deployed on Linux servers with minimal dependencies.

---

## 🎯 Objectives
- Schedule an automated daily backup job at a fixed time
- Archive multiple log files into a single `.tar` bundle
- Use consistent, date-based naming (`YYYY-MM-DD.tar`)
- Store backups in a dedicated directory
- Overwrite prior backups with the same name to keep runs deterministic

---

## 🛠️ Technologies & Skills
- **Linux Job Scheduling:** `cron`, `crontab`
- **Archiving:** `tar`
- **Operational automation**
- **Log handling & retention basics**
- Defensive task design (predictable naming, consistent output paths)

---

## ⚙️ Implementation
### 1️⃣ Prepare the Backup Destination
```bash
mkdir -p /home/USER/project/backup
```
### 2️⃣ Add the Cron Job (Daily at 02:00)
Edit the user’s crontab:
```bash
crontab -e
```
Add this line:

```bash
0 2 * * * tar -cf /home/USER/project/backup/$(date +\%F).tar -C /var/log dpkg.log bootstrap.log fontconfig.log
```
Why this works
0 2 * * * → runs at 02:00 every day


$(date +\%F) → expands to YYYY-MM-DD (cron requires % to be escaped)


-cf → creates the archive and overwrites the file if it already exists


-C /var/log → changes directory so files are stored cleanly as relative paths



## ▶️ Usage Example
Run the same command manually to validate behavior:
```bash
tar -cf /home/USER/project/backup/$(date +\%F).tar -C /var/log dpkg.log bootstrap.log fontconfig.log
```
List archive contents:
```bash
tar -tf /home/USER/project/backup/$(date +\%F).tar
```

## 🧭 MITRE ATT&CK Mapping
| Tactic      | Technique ID | Technique Name                    | Why It Fits                                                        |
| ----------- | -----------: | --------------------------------- | ------------------------------------------------------------------ |
| Collection  |        T1005 | Data from Local System            | Scheduled collection of local log artifacts for retention/analysis |
| Execution   |        T1059 | Command and Scripting Interpreter | Uses standard shell commands executed via scheduled task           |
| Persistence |    T1053.003 | Scheduled Task / Cron             | Cron-based recurring job demonstrates scheduled execution behavior |


## 🧩 NIST CSF Mapping
| Function | Category | What This Project Supports                                                   |
| -------- | -------- | ---------------------------------------------------------------------------- |
| Identify | ID.AM    | Highlights operational asset importance: logs as security-relevant data      |
| Protect  | PR.IP    | Establishes a repeatable process for preserving critical system records      |
| Detect   | DE.CM    | Improves availability of logs needed to detect abnormal activity             |
| Respond  | RS.AN    | Enables faster investigation by ensuring logs are consistently retained      |
| Recover  | RC.IM    | Supports recovery improvements by keeping artifacts for post-incident review |


A daily cron schedule executes at 02:00


A date-stamped archive is created in the backup directory:


YYYY-MM-DD.tar


The archive contains exactly the intended log files:


dpkg.log, bootstrap.log, fontconfig.log


Re-running the job overwrites the existing archive for that date (expected)



## 🔎 Validation Walkthrough
Confirm cron is active
```bash

 systemctl status cron
```
Confirm the job is installed
```bash

 crontab -l
```
Create a test backup immediately (manual run)
```bash

 tar -cf /home/USER/project/backup/$(date +\%F).tar -C /var/log dpkg.log bootstrap.log fontconfig.log
```
Verify the archive exists
```bash

 ls -l /home/USER/project/backup/
```
Verify archive contents
```bash
 tar -tf /home/USER/project/backup/$(date +\%F).tar
```

## 📌 Conclusion
This project demonstrates a practical, operations-ready automation pattern: predictable scheduling + deterministic output + clean archival of system logs.
 It reflects core habits expected in infrastructure, SRE, and junior security roles—ensuring that critical operational data is consistently preserved for troubleshooting and security review.

