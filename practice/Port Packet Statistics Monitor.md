# 📡 Port Packet Statistics Monitor (3-Second Window)

## 📘 Project Overview
Operational teams often need a fast way to confirm whether a service port is actively exchanging traffic—especially during incident triage, change windows, or service validation.  
This project delivers a lightweight **port-focused packet counter** that runs for **exactly 3 seconds** and reports how many packets were observed on a specified port.

The result is a practical “quick signal” tool: *Is there traffic on this port right now, and roughly how much?*

---

## 🎯 Objectives
- Accept a target **port number** as input
- Capture traffic for a **fixed 3-second interval**
- Count observed packets on that port (ingress/egress)
- Output a single, consistent result line: `Packages: <count>`

---

## 🛠️ Technologies & Skills
- **Shell scripting (Zsh/Bash compatible)**
- **Network monitoring fundamentals**
- `tcpdump`, `timeout`, `wc`
- Argument validation and safe exits
- Minimal operational tooling design

---

## 🔧 Implementation
### Script: `netcheck.sh`
```bash
#!/usr/bin/env zsh

# netcheck.sh - Count packets observed on a specified port over a 3-second window

# Validate input
if [[ $# -ne 1 ]]; then
  echo "Usage: $0 <port>"
  exit 1
fi

port="$1"

# Basic numeric port validation (1–65535)
if ! [[ "$port" =~ '^[0-9]+$' ]] || (( port < 1 || port > 65535 )); then
  echo "Usage: $0 <port>"
  exit 1
fi

# Capture for 3 seconds and count packet lines
# Note: tcpdump may require elevated privileges depending on the system configuration.
count=$(timeout 3 tcpdump -i any -n "port ${port}" 2>/dev/null | wc -l | tr -d ' ')

echo "Packages: ${count}"
```
## ⚙️ Usage Example
```bash
chmod +x netcheck.sh
sudo ./netcheck.sh 22
```
Example output:
```bash
Packages: 2
```

## 🧭 MITRE ATT&CK Mapping
| Tactic     | Technique ID | Technique Name                       | Why It Fits                                                                                    |
| ---------- | -----------: | ------------------------------------ | ---------------------------------------------------------------------------------------------- |
| Collection |        T1040 | Network Sniffing                     | Captures/observes network traffic on a specified port to derive activity metrics               |
| Discovery  |        T1049 | System Network Connections Discovery | Helps validate whether a port is actively communicating (operational visibility during triage) |
| Execution  |        T1059 | Command and Scripting Interpreter    | Uses shell tooling to run packet capture and compute counts                                    |


## 🧩 NIST CSF Mapping
| Function | Category | What This Project Supports                                                                 |
| -------- | -------- | ------------------------------------------------------------------------------------------ |
| Detect   | DE.CM    | Quick verification of network activity on a target port during monitoring or investigation |
| Respond  | RS.AN    | Supports triage by confirming whether a suspected service is active and exchanging traffic |
| Protect  | PR.PT    | Helps validate expected port behavior after changes (basic control verification)           |


## ✅ Results & Validation
Script accepts exactly one port argument and rejects invalid values


Runs for a fixed 3-second observation window


Outputs a single standardized line: Packages: <count>


Produces consistent results suitable for fast operational checks



## 🔎 Validation Walkthrough
Confirm the script runs and prints a result
```bash
sudo ./netcheck.sh 22
```
Confirm the fixed-time behavior
```bash
time sudo ./netcheck.sh 22
```
Expected: real runtime is approximately ~3 seconds.
Validate input handling
```bash
./netcheck.sh
./netcheck.sh abc
./netcheck.sh 70000
```
Expected: usage message and clean exit.

## 📌 Conclusion
This project demonstrates practical network monitoring craftsmanship: a small, reliable script that provides immediate insight into whether a port is actively carrying traffic. It’s the kind of lightweight operational tool that fits well in incident response workflows, service validation routines, and day-to-day system administration.

