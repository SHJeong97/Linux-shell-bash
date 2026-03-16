
# 🔐 Secure Password Generator (Bash)

## 📘 Project Overview
Strong passwords are still one of the simplest, highest-impact controls for reducing account takeover risk.  
This project implements a **lightweight Bash password generator** that produces a **12-character** password with enforced complexity rules and an approved special-character set.

It’s designed for **admin workflows**, quick provisioning, and repeatable secure defaults—without relying on external “password generator” tools.

---

## 🎯 Objectives
- Generate a **12-character** password on every execution
- Ensure the password includes **at least**:
  - 1 digit
  - 1 uppercase letter
  - 1 lowercase letter
  - 1 special character (restricted allowlist)
- Allow only these special characters: `><+-{}:.&;`
- Produce a **different output each run**
- Keep the script simple enough to audit quickly

---

## 🛠️ Technologies & Skills
- **Bash scripting**
- Randomized string generation using `$RANDOM`
- Character-class enforcement (upper/lower/digit/special)
- Safe string slicing and length handling
- Output consistency for automation workflows
- Operational security thinking (complexity + unpredictability)

---

## ⚙️ Usage

```bash
chmod +x genpass.sh
./genpass.sh
```
Minimal integration example (admin provisioning pattern):
```bash
PASS="$(./genpass.sh)"
echo "newuser:${PASS}" | sudo chpasswd
```

## 🧾 Full Implementation (genpass.sh)
```bash
#!/bin/bash

# Random Password Generator
# This script generates a random password that meets the specified requirements.

# Function to generate a random password
generate_password() {
  local length=12
  local password=''

  # Special characters (allowlist)
  local special_chars='><+-{}:.&;'
  local special_char="${special_chars:$RANDOM%${#special_chars}:1}"
  password+="$special_char"

  # Digits
  local digits='0123456789'
  local digit="${digits:$RANDOM%${#digits}:1}"
  password+="$digit"

  # Uppercase letters
  local upper_case='ABCDEFGHIJKLMNOPQRSTUVWXYZ'
  local upper="${upper_case:$RANDOM%${#upper_case}:1}"
  password+="$upper"

  # Lowercase letters
  local lower_case='abcdefghijklmnopqrstuvwxyz'
  local lower="${lower_case:$RANDOM%${#lower_case}:1}"
  password+="$lower"

  # Remaining characters
  local remaining_length=$((length - 4))
  local characters='><+-{}:.&;0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'
  local num_characters=${#characters}

  for ((i = 0; i < remaining_length; i++)); do
    local random_char="${characters:$RANDOM%$num_characters:1}"
    password+="$random_char"
  done

  # Shuffle the order of password characters
  password=$(echo "$password" | fold -w1 | shuf | tr -d '\n')

  echo "$password"
}

# Generate password and output
generate_password
```

## 🧩 Implementation Walkthrough
### 1️⃣ Function wrapper + fixed length target
```bash
generate_password() {
  local length=12
  local password=''
```
Keeps the logic contained and reusable.


length=12 is the required output size.


password starts empty and is built step-by-step.



### 2️⃣ Guarantee 1 allowed special character
```bash
local special_chars='><+-{}:.&;'
local special_char="${special_chars:$RANDOM%${#special_chars}:1}"
password+="$special_char"
```
Uses an allowlist so only approved symbols are used.


Ensures the final password always contains a special character.



### 3️⃣ Guarantee 1 digit
```bash
local digits='0123456789'
local digit="${digits:$RANDOM%${#digits}:1}"
password+="$digit"
```
Adds one numeric character to ensure at least one digit exists.



### 4️⃣ Guarantee 1 uppercase + 1 lowercase
```bash
local upper_case='ABCDEFGHIJKLMNOPQRSTUVWXYZ'
local upper="${upper_case:$RANDOM%${#upper_case}:1}"
password+="$upper"

local lower_case='abcdefghijklmnopqrstuvwxyz'
local lower="${lower_case:$RANDOM%${#lower_case}:1}"
password+="$lower"
```
Ensures a mix of case to improve strength against guessing patterns.



### 5️⃣ Fill remaining characters from the full allowed pool
```bash
local remaining_length=$((length - 4))
local characters='><+-{}:.&;0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'

for ((i = 0; i < remaining_length; i++)); do
  local random_char="${characters:$RANDOM%$num_characters:1}"
  password+="$random_char"
done
```
Completes the password to 12 total characters.


Uses a combined pool for better variety.



### 6️⃣ Shuffle final output to avoid predictable placement
```bash
password=$(echo "$password" | fold -w1 | shuf | tr -d '\n')
```
Prevents the first characters from always being “special-digit-upper-lower”.


Reduces pattern predictability.



## ✅ Results & Validation
Sample output
```bash
$ ./genpass.sh
2Dsxw9+xS:27
```
Quick checks (length + character classes)
```bash
PASS="$(./genpass.sh)"

# length should be 12 (exclude newline)
echo -n "$PASS" | wc -c

# must contain at least one of each class
echo "$PASS" | grep -Eq '[0-9]' && echo "digit: OK"
echo "$PASS" | grep -Eq '[A-Z]' && echo "uppercase: OK"
echo "$PASS" | grep -Eq '[a-z]' && echo "lowercase: OK"
echo "$PASS" | grep -Eq '[><+\-{}:.&;]' && echo "special: OK"
```

## 🧪 Validation Walkthrough
### 1️⃣ Confirm output length
Run the script and count characters (excluding newline).


Expected: 12 characters.


### 2️⃣ Confirm required character classes exist
Verify the output includes:


a digit


an uppercase letter


a lowercase letter


an allowed special character


### 3️⃣ Confirm variability across runs
Run the script multiple times.


Expected: passwords differ across executions.



## 🧭 MITRE ATT&CK Mapping
| Tactic            | Technique ID | Technique Name                   | Relevance                                                                                        |
| ----------------- | ------------ | -------------------------------- | ------------------------------------------------------------------------------------------------ |
| Credential Access | T1110        | Brute Force                      | Strong, diverse passwords increase resistance to guessing and password-spraying attempts.        |
| Initial Access    | T1078        | Valid Accounts                   | Reduces likelihood that weak credentials lead to successful authentication.                      |
| Credential Access | T1555        | Credentials from Password Stores | Promotes safer provisioning patterns (generate, apply, rotate) instead of reusing known strings. |


## 🛡️ NIST CSF Mapping
| Function | Category                                             | How This Project Supports It                                                               |
| -------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Protect  | Identity Management, Authentication & Access Control | Improves password quality used for account provisioning and access control hygiene.        |
| Protect  | Protective Technology                                | Enables repeatable, standardized password generation in operational workflows.             |
| Govern   | Risk Management Strategy                             | Supports consistent credential-strength practices that reduce authentication-related risk. |


## 📌 Conclusion
This project demonstrates practical security-minded scripting: enforcing password complexity, controlling allowed character sets, and producing repeatable output suitable for real admin workflows. It’s a small tool, but it targets a big risk area—weak credentials—using clear logic and auditable implementation.


