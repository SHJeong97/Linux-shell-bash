# 🔐 Passwordless SSH (Key-Based) Login for Local Administration

## 📘 Project Overview
Operational Linux environments often require administrators to access nodes quickly and reliably without weakening security through reusable passwords.  
This project implements **key-based SSH authentication** for a local account, enabling **passwordless login to `localhost`** while enforcing correct SSH file permissions and ensuring the SSH service is properly restarted.

The result is a clean, repeatable baseline for **secure interactive access** in lab clusters, jump hosts, or single-node admin workflows.

---

## 🎯 Objectives
- Generate an SSH key pair with **no passphrase**
- Configure the local account to accept the public key via `authorized_keys`
- Enforce secure SSH directory/file permissions
- Restart the SSH service so configuration changes are applied cleanly
- Validate passwordless SSH access to `localhost`

---

## 🛠️ Technologies & Skills
- **Linux / OpenSSH**
- SSH key lifecycle (generate, authorize, validate)
- Secure permission controls (`chmod 700/600`)
- Service management (`service ssh restart`)
- Operational access hardening (reducing password-based login dependence)

---

## ⚙️ Usage Example
```bash
ssh USERNAME@localhost
```
## 💻 Full Implementation
Replace USERNAME with your local account name.
```bash


# 1) Create SSH directory with secure permissions
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# 2) Generate a strong key pair (no passphrase)
# RSA 4096 is widely supported; ED25519 is also a great choice if your environment supports it.
ssh-keygen -t rsa -b 4096 -C "USERNAME@localhost" -N "" -f ~/.ssh/id_rsa

# 3) Authorize the public key for local SSH login
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# (Optional but recommended) Ensure private key permissions are tight
chmod 600 ~/.ssh/id_rsa

# 4) Restart SSH service to apply cleanly
sudo service ssh restart

# 5) Test: should not prompt for password
ssh USERNAME@localhost
```

## 🔍 Implementation Walkthrough
Key decisions
Key-based auth vs passwords: reduces exposure to password reuse, guessing, or shoulder-surfing for routine administrative access.


No passphrase requirement: useful for controlled environments (e.g., local automation/lab clusters) while still relying on private key protection via file permissions.


Permission enforcement: OpenSSH will ignore insecure SSH files; correct permissions are both a security control and an operational requirement.


What each block does
mkdir -p ~/.ssh + chmod 700 ~/.ssh
 Ensures only the account owner can read/write inside the SSH configuration directory.


ssh-keygen ... -N ""
 Creates a key pair explicitly without a passphrase to enable non-interactive login.


cat id_rsa.pub >> authorized_keys + chmod 600 authorized_keys
 Registers the public key as an accepted login credential and restricts file access.


sudo service ssh restart
 Restarts the SSH daemon to ensure clean state and consistent authentication behavior.


ssh USERNAME@localhost
 Confirms end-to-end passwordless authentication is working.



## ✅ Results & Validation
SSH login to localhost succeeds without a password prompt


SSH key materials and authorization files exist with secure permissions:


~/.ssh is 700


~/.ssh/authorized_keys is 600


private key is not world-readable



## 🧪 Validation Walkthrough
### 1️⃣ Confirm key + authorization files exist
```bash


ls -la ~/.ssh
```
What to look for:
id_rsa (private key)


id_rsa.pub (public key)


authorized_keys (contains the public key)


### 2️⃣ Confirm permissions are correct (OpenSSH-safe)
```bash


stat -c "%a %n" ~/.ssh ~/.ssh/authorized_keys ~/.ssh/id_rsa
```
Expected:
```bash


700 ~/.ssh


600 ~/.ssh/authorized_keys


600 ~/.ssh/id_rsa (recommended)
```
### 3️⃣ Restart SSH service and confirm it’s running
```bash


sudo service ssh restart
sudo service ssh status
```
Expected:
Service shows active/running.


### 4️⃣ Validate passwordless login behavior
```bash


ssh USERNAME@localhost
```
Expected:
You get a shell immediately without a password prompt.



## 🗺️ MITRE ATT&CK Mapping
| Tactic            | Technique ID | Technique Name       | Why it applies                                                                                           |
| ----------------- | ------------ | -------------------- | -------------------------------------------------------------------------------------------------------- |
| Lateral Movement  | T1021.004    | Remote Services: SSH | SSH is a primary admin access and lateral movement channel; this project hardens how access is performed |
| Credential Access | T1110        | Brute Force          | Reduces reliance on password-based authentication workflows, shrinking brute-force exposure              |


## 🧩 NIST CSF Mapping
| Function | Category | What this project supports |                                                                                              |
| -------- | -------- | -------------------------- | -------------------------------------------------------------------------------------------- |
| Protect  | PR.AC    | Access Control             | Enforces stronger authentication practices and controlled SSH access paths                   |
| Protect  | PR.PT    | Protective Technology      | Uses standard platform security capabilities (OpenSSH + file permissions + service controls) |
| Respond  | RS.AN    | Analysis                   | Produces a repeatable validation method to confirm secure remote access configuration        |


## 📌 Conclusion
This project demonstrates practical access hardening for Linux administration by replacing routine password-based SSH workflows with key-based authentication, enforcing correct permissions, and validating service behavior. It’

