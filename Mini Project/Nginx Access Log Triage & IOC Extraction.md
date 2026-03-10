# 📊 Nginx Access Log Triage & IOC Extraction (Bash)

## 📘 Project Overview
Security and operations teams often need quick, repeatable answers from web server logs during incident triage: **who is hitting the server most**, **which clients are unusually active**, **what endpoints are being targeted**, and **which requests are producing errors**.

This project delivers a lightweight, command-line–driven log analysis workflow for **Nginx access logs**, producing clean output files that can be fed into incident notes, detection rules, or follow-up investigations. The focus is on **actionable results** and **consistent formatting** for operational use.

---

## 🎯 Objectives
- Identify the **top client IPs** by request volume for a specific date window
- Extract IPs that exceed an **activity threshold** (≥ 10 requests) for a defined day
- Determine the **top requested endpoints** while excluding common static resources
- List all unique **404 (Not Found) endpoints** for follow-up remediation or detection
- Save results into **four separate output artifacts** suitable for reporting

---

## 🛠️ Technologies & Skills
- **Linux CLI** log triage
- **Nginx access log** field awareness
- Text processing: `grep`, `awk`, `sort`, `uniq`
- Frequency analysis and ranking (Top-N)
- Filtering noisy/static resources using regex
- Output artifact generation for operational workflows

---

## ⚙️ Usage (Minimal)
Run the analysis commands from the directory containing `access.log`.  
Outputs are written into four files:

- `output1` → Top 5 IPs for a specified date
- `output2` → IPs with ≥ 10 hits for a specified date
- `output3` → Top 10 request paths (static filtered)
- `output4` → Unique request paths returning 404

---

## 🔧 Implementation

### 1️⃣ Top 5 Client IPs for a Specific Date (Daily Spike Check)
Goal: identify the highest-volume sources for a single day (useful for spotting brute force, scraping, or scanning bursts).

```bash
grep '10/Apr/2015' access.log \
  | awk '{print $1}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -5 \
  | awk '{print $2}' \
  > output1
```

### 2️⃣ IPs With ≥ 10 Requests (Threshold-Based IOC Candidate List)
Goal: extract repeat/high-frequency IPs during a specific day for triage, allow/deny review, or enrichment.
```bash
grep '11/Apr/2015' access.log \
  | awk '{print $1}' \
  | sort \
  | uniq -c \
  | awk '$1 >= 10 {print $2}' \
  > output2
```

### 3️⃣ Top 10 Most Requested Endpoints (Noise-Reduced)
Goal: identify the most targeted endpoints while removing common static resources and typical noise.
```bash
grep -vE '(/robots.txt|\.js|\.css|\.png)' access.log \
  | awk '{print $7}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -10 \
  | awk '{print $2}' \
  > output3
```

### 4️⃣ Unique 404 Request Paths (Broken Links + Recon Targeting)
Goal: extract unique missing paths for remediation, alerting, or threat hunting (common recon behavior produces many 404s).
```bash
grep ' 404 ' access.log \
  | awk '{print $7}' \
  | sort \
  | uniq \
  > output4
```

## 🧾 Code Breakdown (Brief)
🔹 Why this works operationally
grep narrows the dataset (time slice or status code)


awk '{print $1}' extracts the client IP field (common Nginx format)


awk '{print $7}' extracts the request path field


sort | uniq -c converts raw logs into frequency counts


sort -rn | head -N produces ranked Top-N results


Static filtering reduces noise so results reflect meaningful application routes



## 🧪 Results & Validation
### ✅ Output Artifacts
output1 — Top 5 IP addresses (one per line)


output2 — IP addresses with ≥ 10 hits (one per line)


output3 — Top 10 request paths (one per line, static excluded)


output4 — Unique 404 request paths (one per line, no duplicates)


### ✅ Results & Validation
Collected a date-scoped IP leaderboard to identify high-volume sources (output1)


Generated an activity-threshold list to isolate repeat clients (output2)


Extracted request path popularity with noise reduction for static assets (output3)


Captured unique 404 endpoints for follow-up validation and detection (output4)


Ensured each file matched operational reporting requirements:


One value per line


No empty lines


Ranked outputs where applicable


De-duplicated output where required



## 🧩 MITRE ATT&CK Mapping (Relevant)
| Tactic         | Technique ID | Technique Name            | How It Relates                                                                    |
| -------------- | ------------ | ------------------------- | --------------------------------------------------------------------------------- |
| Reconnaissance | T1595        | Active Scanning           | High-frequency IPs and repeated endpoint hits can indicate scanning behavior      |
| Discovery      | T1046        | Network Service Discovery | Repeated access patterns can align with service probing and enumeration behaviors |
| Collection     | T1005        | Data from Local System    | Log analysis supports identifying access patterns tied to attempted data access   |


## 🛡️ NIST CSF Mapping
| Function | Category | What This Project Supports                                                |
| -------- | -------- | ------------------------------------------------------------------------- |
| Identify | ID.AM    | Asset & service visibility through endpoint access patterns               |
| Detect   | DE.AE    | Detect anomalies via traffic spikes, repeat IPs, and unusual 404 patterns |
| Respond  | RS.AN    | Triage support through ranked indicators and scoped evidence outputs      |


## 📌 Conclusion
This project demonstrates practical incident triage workflow skills: extracting high-signal indicators from web logs, producing clean operational artifacts, and supporting detection and response decisions with consistent outputs.

