
# 👤 New User Account Provisioning with Sudo Access (Linux)

## 📘 Project Overview
On internal engineering and testing servers, onboarding a new teammate safely and consistently is a core operational task. This project demonstrates a clean, repeatable workflow to provision a new Linux user with a defined home directory, a preferred default shell, and controlled administrative access via the sudo group.

The goal is not “just create a user,” but to do it in a way that’s **auditable**, **policy-aligned**, and **easy to verify**.

---

## 🎯 Objectives
- Create a new user account: `jane`
- Ensure the home directory is set to: `/home/jane`
- Set the default shell to: `/bin/zsh`
- Grant administrative privileges via sudo group membership

---

## 🛠️ Technologies & Skills
- **Linux user & group management**
- `adduser`, `usermod`, `id`, `getent`, `groups`
- Shell configuration (`/bin/zsh`)
- Privilege management with sudo group
- Operational verification and access control hygiene

---

## 🔧 Implementation

### Account Provisioning Commands
```bash
sudo adduser jane
sudo usermod -d /home/jane jane
sudo usermod -s /bin/zsh jane
sudo usermod -aG sudo jane
```
What Each Command Does (Brief)
adduser jane
 Creates the account and initializes standard user configuration.


usermod -d /home/jane jane
 Ensures the account uses the required home directory path.


usermod -s /bin/zsh jane
 Sets Zsh as the default login shell for the user.


usermod -aG sudo jane
 Adds the user to the sudo group, enabling administrative actions per sudo policy.



## ⚙️ Usage Example
After provisioning, the account can be validated and used:
```bash
su - jane
whoami
```

## 🧭 MITRE ATT&CK Mapping
| Tactic               | Technique ID | Technique Name                                           | Why It Fits                                                                                                                     |
| -------------------- | -----------: | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Persistence          |        T1078 | Valid Accounts                                           | Creating and managing legitimate accounts is directly tied to how “valid accounts” are used and controlled in real environments |
| Privilege Escalation |    T1548.003 | Abuse Elevation Control Mechanism: Sudo and Sudo Caching | Sudo group membership is a common privilege boundary; this work demonstrates controlled assignment and verification             |
| Defense Evasion      |        T1098 | Account Manipulation                                     | Establishes awareness of how account properties (groups/shell/home) can be modified—and why verification matters                |


## 🧩 NIST CSF Mapping

| Function | Category | What This Project Supports                                                             |
| -------- | -------- | -------------------------------------------------------------------------------------- |
| Protect  | PR.AC    | Access control: user provisioning, privilege assignment, and group-based authorization |
| Protect  | PR.PT    | Controlled administrative tooling via sudo and standardized user configuration         |
| Identify | ID.AM    | Asset management practices via consistent account setup and validation checks          |


## ✅ Results & Validation
User account jane created successfully


Home directory configured as /home/jane


Default shell set to /bin/zsh


Sudo privileges enabled through sudo group membership



## 🔎 Validation Walkthrough
1) Confirm the account exists
```bash
getent passwd jane
```
Expected: a passwd entry is returned for jane.
2) Verify home directory and default shell
```bash
getent passwd jane | awk -F: '{print "home:", $6, "\nshell:", $7}'
```
Expected:
home: /home/jane


shell: /bin/zsh


3) Confirm sudo group membership
```bash
id jane
groups jane
```
Expected: sudo appears in the group list.
4) Validate sudo access (policy check)
```bash
sudo -l -U jane
```
Expected: sudo policy output shows the user is allowed per system rules.

## 📌 Conclusion
This project showcases practical identity and access management at the Linux host level: creating accounts with explicit configuration, granting admin rights through group policy, and validating outcomes with standard system queries. These are foundational skills for secure operations, onboarding workflows, and controlled administrative access in production-like environments.

