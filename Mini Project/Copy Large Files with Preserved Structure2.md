# 🗂️ Copy Large Files with Preserved Structure (Linux)

## 📘 Overview
This portfolio entry documents a LabEx challenge where the objective was to **copy only files larger than 10K** from `/etc` to `/tmp/etc`, while **preserving the original directory structure**.

This is a realistic task used in system administration and incident response workflows—collecting only “high-value” configuration files (often larger ones) while keeping paths intact for later review.

---

## 🎯 Objectives
- Copy **all files larger than 10K** under `/etc` into `/tmp/etc`
- Preserve directory structure (ex: `/etc/apt/trusted.gpg` → `/tmp/etc/apt/trusted.gpg`)
- Ensure **only** files larger than 10K exist in `/tmp/etc` after completion
- Run successfully even when files require elevated permissions (`sudo`)

---

## 🧠 Skills Practiced
- Linux file searching with `find`
- Size-based filtering using `-size +10k`
- Privileged file operations with `sudo`
- Structure-preserving copy using `cp --parents`
- Validation and output scoping

---

## 🛠️ Step Summery

### 1) Prepare script target directory
```bash
mkdir -p /tmp
```
Why: Ensures the destination base directory exists before copying.

### 2) Copy only files larger than 10K while preserving paths
```bash
find /etc -type f -size +10k -exec cp --parents --dereference "{}" /tmp \;
```
Why this works:
-type f → files only (no directories)


-size +10k → only files bigger than 10K


cp --parents → preserves the /etc/... structure under /tmp


--dereference → copies real file content (not symlinks)



### 3) Run the challenge script (as required)
```bash
sudo sh /home/labex/project/copy.sh
```
Why: Some /etc files are root-owned; sudo ensures full access and complete copy results.

## ✅ Script Used (copy.sh)
Location: /home/labex/project/copy.sh
```bash
#!/bin/bash

# This script copies files larger than 10K from a source directory to a target directory.

# Define the source directory and target directory
source_dir="/etc"
target_dir="/tmp"

# Create the target directory if it doesn't exist
mkdir -p "$target_dir"

# Use the find command to locate files larger than 10K and copy them to the target directory
find "$source_dir" -type f -size +10k -exec cp --parents --dereference "{}" "$target_dir" \;

echo "File copying complete."
```

## 🔍 Validation (Proof Checks)
Confirm copied structure exists
```bash
ls -alh /tmp/etc
```
Expected: /tmp/etc exists and contains subfolders from /etc.
Confirm no small files were copied
```bash
find /tmp/etc -type f -size -10k -print | head
```
Expected: No output (meaning no files smaller than 10K exist in /tmp/etc).
Spot-check copied large files
```bash
find /tmp/etc -type f -size +10k | head
```
Expected: Shows sample files successfully copied.

## 🧬 MITRE ATT&CK Mapping (Collection / Admin Context)
| Tactic     | Technique ID | Technique Name                  | Lab Relevance                    |
| ---------- | ------------ | ------------------------------- | -------------------------------- |
| Discovery  | T1083        | File and Directory Discovery    | Enumerated `/etc` and subfolders |
| Collection | T1005        | Data from Local System          | Collected local config files     |
| Collection | T1074.001    | Data Staged: Local Data Staging | Staged results in `/tmp/etc`     |


## 🧩 NIST CSF Mapping
| Function | Category | Subcategory                    | Description                                    |
| -------- | -------- | ------------------------------ | ---------------------------------------------- |
| Identify | ID.AM    | Asset Management               | Identify where key config data lives           |
| Protect  | PR.AC    | Access Control                 | Reinforces why `/etc` requires permissions     |
| Detect   | DE.CM    | Security Continuous Monitoring | Bulk copying can be monitored as suspicious    |
| Respond  | RS.AN    | Analysis                       | Supports evidence collection for investigation |
| Recover  | RC.IM    | Improvements                   | Improve repeatable collection workflows        |



## 🔐 Security Takeaway
Even simple Linux commands can perform powerful data collection. A defender should:
monitor access patterns to /etc


restrict permissions appropriately


log and alert on unusual bulk copy behavior


avoid leaving sensitive configs world-readable



## 📌 Key Takeaway
Using find with -size filtering and cp --parents enables clean, scoped collection of important system files while keeping the full directory context, which is essential for audits, incident response, and troubleshooting.

