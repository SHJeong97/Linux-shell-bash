# 🖥️ Linux Server Information Retrieval with Bash (getinfo.sh)

## 📘 Project Overview
System administrators often need a quick, repeatable way to collect **core server metrics** without installing extra tooling.  
This project delivers a Bash script that prints a **standardized 8-line system summary** covering CPU, memory, disk, system architecture, process count, package count, and IP address.

The output format is strict and consistent, making it useful for **baseline snapshots, audits, documentation, and troubleshooting**.

---

## 🎯 Objectives
- Retrieve essential Linux server system metrics using built-in utilities
- Output results in a **strict format and order**
- Practice reliable parsing with `awk`, `sed`, `wc`
- Produce consistent results with **no parameters required**

---

## 🛠️ Technologies & Skills
- **Bash Scripting**
- Linux system introspection (`/proc`, `getconf`, `ip`, `df`, `free`, `ps`)
- Text processing with `awk`, `sed`, `wc`
- Output formatting consistency
- Practical operations scripting

---

## ⚙️ Script Usage

```bash
sh getinfo.sh
```
Expected output format (8 lines, ordered):
```bash
cpu num: 48
memory total: 15 G
memory free: 1257 M
disk size: 11G
system bit: 64
process: 40
software num: 195
ip: 10.2.X.X
```

## 🧾 Step Summery
Script Used (getinfo.sh)
```bash
#!/bin/bash

# getinfo.sh - Linux System Information Script
# This script retrieves CPU, memory, disk, and other information of a Linux server.proc/

# Function: Retrieve CPU information
cpu_num=$(grep -c '^processor' /proc/cpuinfo)

# Function: Retrieve total memory size (in GB)
memory_total=$(free -g | awk '/^Mem:/ {print $2}')

# Function: Retrieve available memory size (in MB)
memory_free=$(free -m | awk '/^Mem:/ {print $4}')

# Function: Retrieve total disk size of the root filesystem (in GB)
disk_size=$(df -h / | awk '/\// {print $2}')

# Function: Retrieve system bit
system_bit=$(getconf LONG_BIT)

# Function: Retrieve the number of currently running processes
process=$(ps -ef | wc -l)

# Function: Retrieve the number of installed software packages
software_num=$(dpkg-query -f '${binary:Package}\n' -W | wc -l)

# Function: Retrieve the IP address of eth0
ip=$(ip addr show eth0 | awk '/inet / {print $2}' | sed 's|/.*||')

# Output information
echo "cpu num: $cpu_num"
echo "memory total: $memory_total G"
echo "memory free: $memory_free M"
echo "disk size: $disk_size"
echo "system bit: $system_bit"
echo "process: $((process - 1))"
echo "software num: $software_num"
echo "ip: $ip"
```

## 🔧 Implementation
### 1️⃣ Create the Script File
Navigate to your working directory and create the script file:
```bash

touch getinfo.sh
```
This prepares an executable container for the required logic.

### 2️⃣ Add the Required System Queries
Open the file and implement each metric using standard Linux sources and commands:
CPU count from /proc/cpuinfo


Memory totals from free


Disk size from df


System bit from getconf


Running processes from ps


Installed software count from dpkg-query


IP address from ip addr


This keeps the solution lightweight and dependency-free.

### 3️⃣ Apply the Exact Output Format
The lab requires:
8 lines total


exact label order


a space after every colon


memory units in G and M


Example formatting rules applied:
```bash
echo "cpu num: $cpu_num"
echo "memory total: $memory_total G"
echo "memory free: $memory_free M"
```

### 4️⃣ Make the Script Executable
```bash
chmod +x getinfo.sh
```
This allows direct execution if needed, while still supporting:
```bash
sh getinfo.sh
```

### 5️⃣ Run and Validate Output
Execute the script:
```bash
sh getinfo.sh
```
Confirm the results match the required structure:
exactly 8 lines


labels match exactly


spacing matches exactly (: )


units match required output




## 🔐 Security & Operational Value
Useful for quick baseline checks during audits or troubleshooting


Helps confirm expected capacity (CPU/RAM/Disk) and system architecture


Provides a simple indicator of process activity and package footprint


Produces standardized output suitable for logs and reports



## 🏆 What This Demonstrates
Practical Bash automation for Linux administration workflows


Strong command-line parsing and formatting discipline


Ability to convert operational requirements into repeatable tooling


Comfort using core Linux inspection commands



## 🚀 Possible Enhancements
Add hostname + kernel version output


Auto-detect primary network interface (not only eth0)


Support RPM-based systems for package counting


Add JSON mode for logging pipelines



## 📌 Conclusion
This project demonstrates operations-ready scripting: consistent output, reliable parsing, and meaningful system context in a single command. It fits naturally into real-world admin, documentation, and security baseline workflows.

