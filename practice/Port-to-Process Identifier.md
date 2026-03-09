# 🔎 Port-to-Process Identifier (Linux) — Bash Utility

## 📘 Project Overview
When troubleshooting services in Linux, a common blocker is **“port already in use.”**  
This project delivers a lightweight Bash utility that takes a port number and returns:

- ✅ the **full path** of the program listening on that port, or  
- ✅ `OK` if nothing is listening

This is designed for fast triage during system administration, incident response, and lab troubleshooting.

---

## 🎯 Objectives
- Accept a **port number** as input
- Detect whether the port is actively listening
- Return **full binary path** of the listening program when present
- Return **OK** when the port is unused
- Keep output minimal and automation-friendly

---

## 🛠️ Technologies & Skills
- **Bash Scripting**
- **Linux Process & Port Enumeration**
- `lsof`, `ps`, `awk`, `sed`
- Input validation
- Defensive scripting patterns
- Operational troubleshooting workflow

---

## ⚙️ Script Usage

```bash
sh get.sh <port>
```
🧪 Examples
Port is in use
sh get.sh 3000
/usr/lib/code-server/lib/node
```
Port is not in use
sh get.sh 43000
OK
```

📌 Step Summery
```bash
#!/bin/bash

# get.sh - Get Running Program on a Specified Port
# This script checks if a program is running on a specified port. If no program is running, it prints "OK".

# Check if the port number is provided as an argument
if [ -z "$1" ]; then
  echo "Please provide a port number."
  exit 1
fi

# Get the port number
port=$1

# Check if the port is in use
process=$(lsof -i :$port -sTCP:LISTEN -Fp | sed 's/^p//')

# Check if a program is running
if [ -z "$process" ]; then
  echo "OK"
else
  # Get the full path of the program
  path=$(ps -p $process -o args=)
  echo "$path" | awk '{print $1}'
fi

```
## 🧾 Breakdown
### 1) Declare the interpreter
```bash
#!/bin/bash
```
Ensures the script runs using Bash, so behavior is consistent across environments.


### 2) Validate that a port was provided
```bash
if [ -z "$1" ]; then
  echo "Please provide a port number."
  exit 1
fi
```
$1 is the first argument (the port).


If it’s missing, the script prints a message and exits with a non-zero status.


### 3) Store the input into a variable
```bash
port=$1
```
Makes the rest of the script easier to read ($port instead of repeating $1).


### 4) Identify the listening process (PID) using lsof
```bash
process=$(lsof -i :$port -sTCP:LISTEN -Fp | sed 's/^p//')
```
lsof -i :$port finds processes tied to that port


-sTCP:LISTEN limits results to listening sockets only


-Fp outputs only the PID field in the format p1234


sed 's/^p//' removes the leading p so the variable contains only 1234


### 5) If no PID is found, print OK
```bash
if [ -z "$process" ]; then
  echo "OK"
```
When lsof returns nothing, process is empty → the port is unused.


### 6) If PID exists, resolve the program path and print it
```bash
else
  path=$(ps -p $process -o args=)
  echo "$path" | awk '{print $1}'
fi
```
ps -p $process -o args= pulls the full command line for that PID


awk '{print $1}' prints only the first token (typically the executable path)


Final output becomes the full path to the program listening on the port




## 🏆 What This Demonstrates
Practical Linux troubleshooting and enumeration


Clean Bash scripting with focused output


Ability to turn common operational needs into reusable tooling


Security-minded system visibility workflow



## 🚀 Possible Enhancements
Support protocol filters (TCP/UDP)


Show PID + user + command line (optional verbose mode)


Accept multiple ports in one run


JSON output for SIEM ingestion


Fallback to ss when lsof is unavailable



## 📌 Conclusion
This utility is a small but realistic operations tool: simple input, clear output, and direct value in troubleshooting and security validation. It reflects the type of scripting and host visibility work expected in junior security and Linux administration roles.

