# 🗂️ Linux Forensics-Style File Collection (Time-Scoped Copy + Preserve Paths)

## 📘 Overview
This challenge simulates a common **IR / forensics** and **sysadmin** task: collecting system configuration files from a **specific time window** while preserving original directory structure.

Goal: **Copy every file under `/etc` that was last modified in 2022 into `/tmp/etc`**, keeping the same folder structure (e.g., `/etc/fstab` → `/tmp/etc/fstab`).  
Only **files** count — directory timestamps alone should not trigger copying.

---

## 🎯 Learning Objectives
- Use `find` to filter files by modification time (year-based time window)
- Preserve directory structure during bulk copy operations
- Handle restricted permissions safely using `sudo`
- Validate the result by ensuring output contains only the intended time-scoped files

---

## 🧠 Core Concept
Time-bound file collection is valuable for:
- Incident response: collecting configs modified during an intrusion window
- Auditing: reviewing what changed within a year/period
- Backups & migration: capturing only relevant modifications

---

## 🛠️ Implementation (collect_files.sh)

### 1) Script Location
`/home//collect_files.sh`

### 2) Script Code
```bash
#!/bin/bash

# Script: collect_files.sh
# Purpose: Copy /etc files last modified in 2022 into /tmp/etc while preserving directory structure.

source_dir="/etc"
target_dir="/tmp"
year="2022"

# Ensure target exists
mkdir -p "$target_dir"

# Copy only files modified during 2022, preserving full paths under /tmp
find "$source_dir" -type f -newermt "$year-01-01" ! -newermt "$year-12-31" \
  -exec cp --parents --dereference "{}" "$target_dir" \;

echo "File copying completed."

```

## 🔍 Step Summery
### ✅ Run the collection as root (permission-safe)
```bash
sudo sh /home/collect_files.sh

```

Why: /etc contains many root-owned files. Running with sudo ensures full coverage.

### ✅ Confirm files were copied into the expected base path
```bash
ls -alh /tmp/etc

```

Proves: Output directory exists and includes copied /etc structure.

### ✅ Verify the directory structure is preserved
```bash
find /tmp/etc -type f | head

```

Proves: Files exist inside nested paths, not flattened into one folder.

### ✅ Validate the time window (spot-check)
```bash
stat /tmp/etc/fstab 2>/dev/null | grep -E "Modify|Change"

```

Proves: A known file’s timestamp can be checked against the requirement.

### ✅ Ensure only 2022-modified files exist in /tmp/etc
```bash
find /tmp/etc -type f ! -newermt "2022-01-01" -o -newermt "2023-01-01"

Proves: Detects any files outs
```
ide the intended 2022 window (should return nothing meaningful).

## 🧬 MITRE ATT&CK Mapping (Defensive / IR Use Case)

| Tactic     | Technique ID | Technique Name                  | Why It Fits                                      |
| ---------- | ------------ | ------------------------------- | ------------------------------------------------ |
| Discovery  | T1083        | File and Directory Discovery    | Identifying files under `/etc` for collection    |
| Collection | T1005        | Data from Local System          | Gathering local configuration files for analysis |
| Collection | T1074.001    | Data Staged: Local Data Staging | Staging copies in `/tmp/etc` for export/review   |


## 🧩 NIST CSF Mapping
| Function     | Category | Subcategory          | Description                                                    |
| ------------ | -------- | -------------------- | -------------------------------------------------------------- |
| **Identify** | ID.AM    | Asset Management     | Locating configuration assets under `/etc`                     |
| **Detect**   | DE.AE    | Anomalies and Events | Supporting timeline-based review of system changes             |
| **Respond**  | RS.AN    | Analysis             | Collecting evidence for investigation and scoping              |
| **Recover**  | RC.IM    | Improvements         | Improving repeatable collection workflows for future incidents |


## 🛡️ Security & Operational Notes
Uses -type f to ensure only files are copied (not directories)


Uses cp --parents to preserve full directory paths beneath /tmp


Uses --dereference to copy real content rather than relying on links


Designed to run under sudo to avoid incomplete collection due to permissions



## ✅ Key Takeaway
Filtering by time boundaries + preserving directory structure is a core skill for:
incident response evidence handling


audit and compliance checks


system administration automation


This lab demonstrates how to build a reliable time-scoped file collection script using standard Linux tools.

