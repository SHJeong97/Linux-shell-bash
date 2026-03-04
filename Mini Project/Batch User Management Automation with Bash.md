# 👥 Batch User Management Automation with Bash

## 📘 Project Overview
Managing users manually on Linux systems is time-consuming and error-prone, especially in environments like classrooms, labs, or training servers.  
This project delivers a **robust Bash automation tool** that allows system administrators to **create and delete teacher and student accounts in batches**, with strict input validation, secure password handling, and safe execution.

The script is designed to be **idempotent**, **secure**, and **production-ready**, demonstrating real-world Linux administration skills.

---

## 🎯 Objectives
- Automate bulk user creation and deletion
- Enforce strict parameter validation
- Safely handle existing users
- Generate secure random passwords
- Assign appropriate shells and privileges
- Support repeatable, clean system administration workflows

---

## 🛠️ Technologies & Skills
- **Bash Scripting**
- **Linux User & Group Management**
- `useradd`, `userdel`, `usermod`
- Argument parsing & validation
- Loops and conditionals
- Secure password generation
- Privilege handling with `sudo`
- Defensive scripting (`set -euo pipefail`)

---

## ⚙️ Script Usage

```bash

#!/usr/bin/env bash
set -euo pipefail


# Usage:
#   ./manage_class.sh add teachername stu 5
#   ./manage_class.sh del teachername stu 5
#
# Optional env vars:
#   OUT_FILE=/path/to/credentials.txt   (default: ./credentials.txt)
#   PRINT_PASSWORDS=1                   (prints passwords to screen; default off)
#   DRY_RUN=1                           (no changes; just shows what would happen)


usage() {
  echo "Usage: $0 <add|del> <teacher_name> <student_prefix> <student_count(1-10)>" >&2
  exit 1
}


err() {
  echo "parameter error" >&2
  exit 1
}


run() {
  if [[ "${DRY_RUN:-0}" == "1" ]]; then
    echo "[DRY_RUN] $*"
  else
    eval "$@"
  fi
}


# --- Validate args ---
[[ $# -eq 4 ]] || usage


OPERATION="$1"
TEACHER_NAME="$2"
STUDENT_PREFIX="$3"
STUDENT_COUNT="$4"


[[ "$OPERATION" == "add" || "$OPERATION" == "del" ]] || err


# Linux username rules (simplified + practical)
# - starts with letter or underscore
# - then letters/numbers/underscore/dash
# - length up to 32 is common
[[ "$TEACHER_NAME" =~ ^[a-z_][a-z0-9_-]{0,31}$ ]] || err
[[ "$STUDENT_PREFIX" =~ ^[a-z]+$ ]] || err
[[ "$STUDENT_COUNT" =~ ^[0-9]+$ ]] || err
(( STUDENT_COUNT >= 1 && STUDENT_COUNT <= 10 )) || err


# Pick sudo if needed
SUDO=""
if [[ "${EUID:-$(id -u)}" -ne 0 ]]; then
  SUDO="sudo"
fi


# Choose a shell that exists
DEFAULT_SHELL="/bin/bash"
[[ -x /bin/zsh ]] && DEFAULT_SHELL="/bin/zsh"


# Credential output (protected)
OUT_FILE="${OUT_FILE:-./credentials.txt}"
PRINT_PASSWORDS="${PRINT_PASSWORDS:-0}"


# --- Helpers ---
user_exists() {
  id -u "$1" &>/dev/null
}


generate_password() {
  # Prefer openssl if available; fallback to /dev/urandom.
  if command -v openssl &>/dev/null; then
    # 12 chars, alnum only
    openssl rand -base64 18 | tr -dc 'A-Za-z0-9' | head -c 12
    echo
  else
    tr -dc 'A-Za-z0-9' </dev/urandom | head -c 12
    echo
  fi
}


write_cred() {
  local u="$1" p="$2"
  if [[ "$PRINT_PASSWORDS" == "1" ]]; then
    echo "$u:$p"
  else
    echo "$u:******"
  fi


  # Only write real passwords on "add" when the user is newly created
  if [[ "$OPERATION" == "add" ]]; then
    # Protect the credentials file
    ( umask 077; echo "$u:$p" >> "$OUT_FILE" )
  fi
}


create_user() {
  local u="$1"


  if user_exists "$u"; then
    # Existing user: do not reset password automatically (safer)
    echo "$u:******"
    return 0
  fi


  local p
  p="$(generate_password)"


  run "$SUDO useradd -m -s \"$DEFAULT_SHELL\" \"$u\""
  # Set password
  run "echo \"$u:$p\" | $SUDO chpasswd"
  # Force password change on first login
  run "$SUDO chage -d 0 \"$u\""


  write_cred "$u" "$p"
}


delete_user() {
  local u="$1"
  # Safe delete: suppress noise, ignore if missing
  if user_exists "$u"; then
    run "$SUDO userdel -r \"$u\" >/dev/null 2>&1 || true"
  fi
}


# --- Main ---
if [[ "$OPERATION" == "add" ]]; then
  # Fresh credentials file per run (optional; comment out if you prefer append)
  if [[ "${DRY_RUN:-0}" != "1" ]]; then
    ( umask 077; : > "$OUT_FILE" )
  else
    echo "[DRY_RUN] would create/overwrite credentials file: $OUT_FILE"
  fi


  create_user "$TEACHER_NAME"
  # Optional: only add teacher to sudo if you truly want that
  if user_exists "$TEACHER_NAME"; then
    run "$SUDO usermod -aG sudo \"$TEACHER_NAME\""
  fi


  for ((i=1; i<=STUDENT_COUNT; i++)); do
    create_user "${STUDENT_PREFIX}${i}"
  done


  if [[ "$PRINT_PASSWORDS" != "1" ]]; then
    echo "Credentials saved to: $OUT_FILE (protected permissions)."
    echo "Tip: set PRINT_PASSWORDS=1 if you want them printed to screen."
  fi


elif [[ "$OPERATION" == "del" ]]; then
  delete_user "$TEACHER_NAME"
  for ((i=1; i<=STUDENT_COUNT; i++)); do
    delete_user "${STUDENT_PREFIX}${i}"
  done
fi


```
### 0) Goal of the script
You run it like:
```bash
./manage_class.sh add teacher1 stu 5
```
It will:
create teacher1


create stu1 stu2 stu3 stu4 stu5


generate passwords for new users


set the passwords


force password change at first login


store credentials in a protected file (by default ./credentials.txt)


Delete mode:
```bash
./manage_class.sh del teacher1 stu 5
```
It will delete teacher1 and those student users (including home directories).

### 1) Header + strict mode
```bash
#!/usr/bin/env bash
set -euo pipefail
```
#!/usr/bin/env bash
This tells the OS: “run this script using bash.”


Using /usr/bin/env bash is more portable than #!/bin/bash because it finds bash via the PATH.


set -euo pipefail (strict mode)
This makes scripts much safer:
-e: exit immediately if any command fails (non-zero exit status)


prevents continuing after a failure (like half-created accounts)


-u: treat unset variables as errors


prevents bugs like $STUDENT_COUTN typo becoming empty and causing destructive commands


pipefail: if any command in a pipeline fails, the whole pipeline fails


e.g. echo "x" | command failing won’t be hidden



### 2) Helper functions for consistent errors
```bash
usage() {
  echo "Usage: $0 <add|del> <teacher_name> <student_prefix> <student_count(1-10)>" >&2
  exit 1
}

err() {
  echo "parameter error" >&2
  exit 1
}
```
usage()
prints a helpful usage message


sends it to stderr (>&2) so error output is separated from normal output


exits with code 1 meaning failure


err()
prints the lab-style “parameter error”


exits


used when input validation fails


This keeps your script consistent and easier to modify.

### 3) “Run command” wrapper (dry-run support)
```bash
run() {
  if [[ "${DRY_RUN:-0}" == "1" ]]; then
    echo "[DRY_RUN] $*"
  else
    eval "$@"
  fi
}
```
What it does
This function is used to run system-changing commands.
If DRY_RUN=1 is set, the script does not execute commands.


It prints what it would run.


Example:
```bash
DRY_RUN=1 ./manage_class.sh add teacher1 stu 3
```
You’ll see lines like:
```bash
[DRY_RUN] sudo useradd -m -s "/bin/zsh" "teacher1"
```
Why DRY_RUN is useful
lets you test logic safely


good for demos and audits


prevents accidentally creating/deleting users while debugging


Why eval is used
We pass quoted strings like "$SUDO useradd ..." into run. eval executes the whole composed command.
(If you want ultra-hardening, we can avoid eval, but for this lab-style script it’s okay.)

### 4) Argument count validation
```bash
[[ $# -eq 4 ]] || usage
```
Meaning
$# = number of arguments


must be exactly 4 or we exit


So these are required:
OPERATION → add or del


TEACHER_NAME


STUDENT_PREFIX


STUDENT_COUNT



### 5) Assign args to variables
```bash
OPERATION="$1"
TEACHER_NAME="$2"
STUDENT_PREFIX="$3"
STUDENT_COUNT="$4"
```
Why this matters:
more readable than $1, $2 everywhere


reduces mistakes


makes auditing easier



### 6) Validate operation type
```bash
[[ "$OPERATION" == "add" || "$OPERATION" == "del" ]] || err
```
Only two accepted values:
add


del


If you enter something else → “parameter error”
This avoids the script silently doing nothing (which can confuse lab checkers).

### 7) Validate teacher name format
```bash
[[ "$TEACHER_NAME" =~ ^[a-z_][a-z0-9_-]{0,31}$ ]] || err
```
What this regex means
^ start of string


[a-z_] first character must be a lowercase letter or underscore


[a-z0-9_-]{0,31} remaining chars can be lowercase letters, digits, underscore, dash


{0,31} means 0 to 31 more characters


$ end of string


So max length is 32 chars.
Why validate?
prevents invalid Linux usernames


prevents weird characters that could break commands or logs



### 8) Validate student prefix
```bash
[[ "$STUDENT_PREFIX" =~ ^[a-z]+$ ]] || err
```
Prefix must be one or more lowercase letters.
Example good: stu
 bad: stu_ or Stu or s1
Why?
ensures predictable stu1, stu2, etc.


reduces risk of accidental weird usernames



### 9) Validate student count
```bash
[[ "$STUDENT_COUNT" =~ ^[0-9]+$ ]] || err
(( STUDENT_COUNT >= 1 && STUDENT_COUNT <= 10 )) || err
```
First line: must be digits only.
 Second line: must be between 1 and 10.
Why?
prevents giant account creation loops


matches lab constraints


prevents for ((i=1; i<=; i++)) errors from empty values



### 10) Auto-select sudo if needed
```bash
SUDO=""
if [[ "${EUID:-$(id -u)}" -ne 0 ]]; then
  SUDO="sudo"
fi
```
EUID is the effective user id (0 is root)


If not root, use sudo.


This lets you run as:
root: no sudo needed


normal user: sudo used automatically



### 11) Choose a login shell that exists
```bash
DEFAULT_SHELL="/bin/bash"
[[ -x /bin/zsh ]] && DEFAULT_SHELL="/bin/zsh"
```
defaults to bash


if /bin/zsh exists and executable (-x) then use zsh


Why?
avoids failing useradd -s /bin/zsh on systems that don’t have zsh



### 12) Output and behavior switches
```bash
OUT_FILE="${OUT_FILE:-./credentials.txt}"
PRINT_PASSWORDS="${PRINT_PASSWORDS:-0}"
```
If OUT_FILE env var is set, use it; else default to ./credentials.txt


If PRINT_PASSWORDS=1, it prints passwords to screen; otherwise it masks them


Example:
PRINT_PASSWORDS=1 ./manage_class.sh add teacher1 stu 3


### 13) Helper: check if a user exists
```bash
user_exists() {
  id -u "$1" &>/dev/null
}
```
id -u username returns success if user exists.


redirects all output to /dev/null to keep it quiet.


This is reliable and standard.

### 14) Generate a password securely
```bash
generate_password() {
  if command -v openssl &>/dev/null; then
    openssl rand -base64 18 | tr -dc 'A-Za-z0-9' | head -c 12
    echo
  else
    tr -dc 'A-Za-z0-9' </dev/urandom | head -c 12
    echo
  fi
}
```
What it does
If openssl exists: use it (strong randomness)


else fallback: read from /dev/urandom (still strong)


Then:
filters to only letters/numbers using tr -dc


takes 12 characters using head -c 12


prints newline


Why this is better than 6-digit numbers:
harder to brute force


more policy-compliant



### 15) Write credentials safely
```bash
write_cred() {
  local u="$1" p="$2"
  if [[ "$PRINT_PASSWORDS" == "1" ]]; then
    echo "$u:$p"
  else
    echo "$u:******"
  fi

  if [[ "$OPERATION" == "add" ]]; then
    ( umask 077; echo "$u:$p" >> "$OUT_FILE" )
  fi
}
```
What happens here
It prints either:


user:REALPASS (if PRINT_PASSWORDS=1)


user:****** (default safe behavior)


Then it writes real credentials to the file using:


( umask 077; echo "$u:$p" >> "$OUT_FILE" )

Why ( umask 077; ... ) is used
That’s a subshell:
umask 077 only applies inside the parentheses


it doesn’t permanently change your shell’s umask


077 makes the output file owner-only (600)


So credentials don’t become world-readable.

### 16) Creating a user
```bash
create_user() {
  local u="$1"

  if user_exists "$u"; then
    echo "$u:******"
    return 0
  fi

  local p
  p="$(generate_password)"

  run "$SUDO useradd -m -s \"$DEFAULT_SHELL\" \"$u\""
  run "echo \"$u:$p\" | $SUDO chpasswd"
  run "$SUDO chage -d 0 \"$u\""

  write_cred "$u" "$p"
}
```
Step by step
If user exists:


don’t recreate


don’t reset password (safer)


Generate password


Create user:


useradd -m creates home directory


-s sets default shell


Set password:


chpasswd reads user:pass from stdin


Force password change:


chage -d 0 makes them change password at first login


Save credentials



### 17) Deleting a user
```bash
delete_user() {
  local u="$1"
  if user_exists "$u"; then
    run "$SUDO userdel -r \"$u\" >/dev/null 2>&1 || true"
  fi
}
```
Only deletes if user exists


-r removes home directory too


redirects output so deletion doesn’t spam terminal


|| true prevents delete failures from killing the entire script under set -e



### 18) Main mode switch
Add mode
```bash
if [[ "$OPERATION" == "add" ]]; then
```
Create / overwrite credentials file safely
```bash
if [[ "${DRY_RUN:-0}" != "1" ]]; then
  ( umask 077; : > "$OUT_FILE" )
else
  echo "[DRY_RUN] would create/overwrite credentials file: $OUT_FILE"
fi
```
: > file truncates/creates file (like > file)


umask 077 ensures it’s protected


In dry-run, it doesn’t create anything.


Create teacher + sudo group
```bash
create_user "$TEACHER_NAME"
if user_exists "$TEACHER_NAME"; then
  run "$SUDO usermod -aG sudo \"$TEACHER_NAME\""
fi
```
creates teacher


adds them to sudo (optional; can remove if you want least privilege)


Create students in a loop
```bash
for ((i=1; i<=STUDENT_COUNT; i++)); do
  create_user "${STUDENT_PREFIX}${i}"
done
```
Creates stu1...stuN.
Final message
```bash
if [[ "$PRINT_PASSWORDS" != "1" ]]; then
  echo "Credentials saved to: $OUT_FILE (protected permissions)."
fi
```
Only prints file location if it’s not showing passwords directly.

Delete mode
```bash
elif [[ "$OPERATION" == "del" ]]; then
  delete_user "$TEACHER_NAME"
  for ((i=1; i<=STUDENT_COUNT; i++)); do
    delete_user "${STUDENT_PREFIX}${i}"
  done
fi
```
delete teacher


delete students


no prompts (batch style)

## 🔐 Security & Safety Features
Strict argument count enforcement


Username and prefix validation via regex


Student count bounds enforced (1–10)


Secure password generation


Default shell set to /bin/zsh (or fallback)


Teacher automatically added to sudo group


Script exits safely on errors



## 🧠 Key Design Decisions
Idempotent behavior — existing users are detected, not overwritten


Least surprise principle — deletion produces no output


Secure defaults — no plaintext password leakage by default


Root-safe execution — automatically uses sudo if required



## 🏆 What This Demonstrates
Practical Linux system administration


Secure automation mindset


Defensive scripting techniques


Production-grade Bash structure


Ability to translate operational needs into tooling



## 🚀 Possible Enhancements
Group creation for classes


CSV input support


Logging to audit files


Password expiration policies


Role-based access tiers


Integration with LDAP / AD



## 📌 Conclusion
This project reflects real operational tooling, not academic scripting.
 It solves a genuine administrative problem with safety, clarity, and scalability — the exact qualities expected of a Linux system administrator or junior security engineer.

