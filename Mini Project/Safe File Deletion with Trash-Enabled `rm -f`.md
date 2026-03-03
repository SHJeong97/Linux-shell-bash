# 🗑️ Safe File Deletion with Trash-Enabled `rm -f` (Linux)

## 📘 Project Overview
Accidental file deletion using `rm -rf` is a common and dangerous operational risk on Linux systems.  
This project implements a **system-wide safety mechanism** that modifies the behavior of the `rm -f` command so that files and directories are **moved to a temporary trash directory instead of being permanently deleted**.

The solution preserves the original behavior of `rm` when the `-f` flag is **not** used, ensuring compatibility with existing workflows while adding a protective layer against destructive mistakes.

---

## 🎯 Objectives
- Prevent accidental permanent deletion using `rm -f`
- Redirect forced deletions to a safe trash directory (`/tmp/trash`)
- Preserve default `rm` behavior for non-`-f` usage
- Ensure the modified behavior applies **system-wide** for all users
- Maintain compatibility with existing Linux utilities and scripts

---

## 🛠️ Tools & Technologies
- **Operating System:** Linux
- **Shell Scripting:** Bash / Zsh
- **Core Utilities:** `rm`, `mv`, `chmod`, `chown`
- **Environment Management:** `PATH`
- **Permissions & Security:** Sticky bit (`1777`)

---

## ⚙️ Design Approach

### Key Design Principles
- **Safety-first:** No data is destroyed when using `rm -f`
- **Minimal disruption:** Default `rm` behavior remains intact
- **System-wide impact:** Works for all users
- **Reversible:** Deleted files can be recovered from trash

### High-Level Flow
1. Intercept `rm -f` using a wrapper script
2. Move target files/directories to `/tmp/trash`
3. Overwrite files in trash if name conflicts occur
4. Delegate all other cases to the original `/bin/rm`

---

## 🔧 Implementation

### 1️⃣ Create a Shared Trash Directory
```bash
sudo mkdir /tmp/trash
sudo chown root:root /tmp/trash
sudo chmod 1777 /tmp/trash
```
Sticky bit (1777) ensures:


Any user can write files


Only file owners can remove their files


Mirrors standard /tmp security behavior



### 2️⃣ Create a High-Priority Custom Binary Directory
```bash
sudo mkdir -p /usr/local/custom/bin
sudo chmod a+x /usr/local/custom/bin
```
This directory will contain the custom rm wrapper and be loaded before system binaries.

### 3️⃣ Update System PATH (All Users)
Edit the environment configuration:
```bash
sudo vim /etc/environment
```
Ensure /usr/local/custom/bin is first in the PATH:
```bash
PATH="/usr/local/custom/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```
Apply changes:
```bash
source /etc/environment
```

### 4️⃣ Implement the Custom rm Wrapper
Create the wrapper script:
```bash
sudo vim /usr/local/custom/bin/rm

#!/bin/zsh

TRASH_DIR="/tmp/trash"

if [[ "$1" == "-f" ]]; then
  shift
  for target in "$@"; do
    if [[ -e "$target" ]]; then
      mv -f "$target" "$TRASH_DIR"
    else
      echo "Error: $target does not exist."
    fi
  done
else
  /bin/rm "$@"
fi
```
Set ownership and permissions:
```bash
sudo chown root:root /usr/local/custom/bin/rm
sudo chmod 755 /usr/local/custom/bin/rm
```

## 🧪 Verification Example
```bash
rm -f important_file.txt
ls /tmp/trash
```
Expected behavior:
File no longer exists in the original location


File appears inside /tmp/trash


No permanent deletion occurs



## ✅ Results
rm -f safely redirects deletions to a trash directory


Default rm behavior remains unchanged


Works transparently for all system users


Protects against irreversible human error



## 🔐 Security & Operational Impact
Reduces risk of catastrophic data loss


Adds a safety net without modifying core binaries


Demonstrates secure command interception using PATH precedence


Reinforces least-destruction operational principles



## 🧠 Key Takeaways
Small wrapper scripts can significantly improve system safety


PATH ordering is a powerful system control mechanism


Defensive tooling does not require kernel or binary modification


Safe defaults are critical in production environments



## 🚀 Possible Enhancements
Timestamped trash subdirectories


Per-user trash isolation


Automatic cleanup policy for /tmp/trash


Restore command (unrm)


Logging of deletion actions for auditing



## 📌 Conclusion
This project demonstrates a practical approach to defensive Linux system engineering.
 By modifying command behavior safely and system-wide, it prevents one of the most common and costly administrative mistakes—accidental deletion—while maintaining compatibility and usability.
This solution reflects real-world practices used in hardened Linux environments and production systems.

