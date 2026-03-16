# 🔎 Linux Config Discovery: Find /etc Files Containing a Target String

## 📘 Project Overview
During security assessments and incident response, you often need to quickly identify **where a specific keyword appears inside system configuration files**—for example a username, service account, hardcoded identifier, or environment-specific marker.

This project demonstrates a clean, repeatable approach to **discovering every file under `/etc` that contains a target string**, while handling noisy permission errors and producing a deduplicated path list suitable for reporting or follow-up review.

---

## 🎯 Objectives
- Search recursively under `/etc` for a target string found in file contents
- Output **only file paths** (not matching lines)
- Suppress permission noise while scanning protected directories
- Save results into a single output file for review and audit trail

---

## 🛠️ Technologies & Skills
- **Linux / Bash CLI**
- Recursive file search & content matching
- `grep` flags for operational triage:
  - `-r` recursive traversal
  - `-s` suppress errors (permission-denied noise)
  - `-l` output filenames only
- Output redirection and reproducible result capture

---

## ⚙️ Usage Example
```bash
sudo grep -rsl "TARGET_STRING" /etc > output

## 💻 Full Implementation
```bash
sudo grep -rsl "TARGET_STRING" /etc > /home/user/project/output
```

## 🔍 Implementation Walkthrough
### ✅ What the command is doing
sudo: ensures the scan can read protected system config files in /etc


grep: searches file contents for a pattern


-r: recursively scans /etc and all subdirectories


-s: suppresses error output (e.g., permission denied) so results stay clean


-l: prints only file paths that contain at least one match


> output: writes the final list to a report file for review


### ✅ Why this is security-relevant
This approach is commonly used to:
locate hardcoded identifiers or environment markers


confirm where a service account is referenced across configs


discover unexpected persistence or misconfigurations (e.g., suspicious references inside supervisor/system configs)


build a focused list of files for deeper auditing



### ✅ Results & Validation
The resulting file contains:
one file path per line


no duplicates


only files under /etc where the string appears in the file contents


Example output shape:
```bash
/etc/group
/etc/passwd
/etc/shadow
/etc/security/limits.conf
/etc/supervisor/conf.d/service.conf
...
```

## 🧪 Validation Walkthrough
### 1️⃣ Run the scan and save output
```bash
sudo grep -rsl "TARGET_STRING" /etc > /home/user/project/output
```
### 2️⃣ Confirm the output file exists and has content
```bash
ls -l /home/user/project/output
wc -l /home/user/project/output
```
### 3️⃣ Spot-check a few returned files to confirm true matches
```bash
head -n 10 /home/user/project/output
sudo grep -n "TARGET_STRING" "$(head -n 1 /home/user/project/output)"
```
### 4️⃣ Confirm no duplicates (optional)
```bash
sort /home/user/project/output | uniq -d
```
If this returns nothing, the file list contains no duplicates.

## 🗺️ MITRE ATT&CK Mapping
| Tactic    | Technique ID | Technique Name               | Why it applies                                                                        |
| --------- | ------------ | ---------------------------- | ------------------------------------------------------------------------------------- |
| Discovery | T1083        | File and Directory Discovery | Enumerates file locations to identify where relevant data/config references exist     |
| Discovery | T1005        | Data from Local System       | Identifies local configuration files containing target strings for follow-on analysis |


## 🧩 NIST CSF Mapping
| Function | Category                               | What this project supports                                                                    |
| -------- | -------------------------------------- | --------------------------------------------------------------------------------------------- |
| Identify | Asset Management (ID.AM)               | Improves visibility into what configuration files contain environment-specific identifiers    |
| Detect   | Security Continuous Monitoring (DE.CM) | Supports rapid validation of config references during investigation or triage                 |
| Respond  | Analysis (RS.AN)                       | Produces a documented artifact to guide deeper review of suspicious or sensitive config files |


## 📌 Conclusion
This project is a practical example of operational Linux discovery work that shows up in real security workflows. It converts an open-ended question—“where is this referenced in system configs?”—into a clean, auditable result that supports incident response, configuration review, and security hardening.

