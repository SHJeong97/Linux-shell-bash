# 📧 Linux Mail Server Setup with Postfix

## 📘 Project Overview
This project demonstrates the installation, configuration, and validation of a **local mail server using Postfix** on a Linux system.  
The goal is to understand how email is routed internally, how mail transfer agents (MTAs) operate, and how system users receive email.

By completing this project, I configured a functional Postfix mail server, mapped email addresses to local users, and verified successful email delivery through command-line testing.

---

## 🎯 Objectives
- Install and configure the Postfix Mail Transfer Agent (MTA)
- Edit Postfix configuration files using Vim
- Create and manage local Linux users for mail delivery
- Configure virtual email address mapping
- Send and receive test emails from the command line
- Validate mail flow and delivery

---

## 🛠️ Tools & Technologies
- **Operating System:** Linux (Ubuntu-based)
- **Mail Server:** Postfix
- **Mail Utilities:** mailutils
- **Editor:** Vim
- **Commands Used:** `postfix`, `postmap`, `mail`, `useradd`, `service`

---

## ⚙️ Architecture Overview
- **Mail Server Type:** Internet Site
- **Domain:** `labex.io`
- **MTA:** Postfix
- **Mailbox Type:** Local user mailbox
- **Email Routing:** Virtual alias mapping

---

## 🔧 Implementation Steps

### 1️⃣ Install Postfix
```bash
sudo apt-get update
sudo apt-get install postfix

```

Configuration choices during install:
Mail server type: Internet Site


System mail name: labex.io



### 2️⃣ Configure Postfix
Edited the main Postfix configuration file:
```bash
sudo vim /etc/postfix/main.cf

```

Key configuration changes:
```bash
myhostname = labex.io
alias_maps = hash:/etc/postfix/virtual

```

These settings define the mail server identity and enable virtual address mapping.

### 3️⃣ Create a Local Mail User
```bash
sudo useradd -m -d /home/master master
sudo passwd master

```

This user acts as the mailbox owner for incoming mail.

### 4️⃣ Configure Email Address Mapping
Mapped an email address to the local user:
```bash
echo "master@labex.io    master" | sudo tee -a /etc/postfix/virtual

```

Apply the mapping:
```bash
sudo postmap /etc/postfix/virtual
sudo service postfix restart

```

This allows Postfix to route mail for master@labex.io to the local master user.

### 5️⃣ Send a Test Email
Installed mail utilities:
```bash
sudo apt-get install mailutils

```

Sent a test message:
```bash
echo "Hello, this is a test email." | mail -s "Test Email" master@labex.io

```


### 6️⃣ Verify Email Delivery
Switched to the mailbox user and checked mail:
```bash
su master
mail

```

The test email was successfully received, confirming correct mail routing and delivery.

## ✅ Results
Postfix mail server installed and running


Domain-based email routing configured


Local mailbox delivery verified


Successful end-to-end email transmission tested



## 🔐 Security & Operations Considerations
Local-only mail delivery limits external attack surface


Virtual alias mapping reduces misdelivery risks


Understanding mail routing helps identify:


Mail relay abuse


Misconfigured MTAs


Privilege escalation via mail services



## 🧠 Key Takeaways
Email delivery relies on precise server identity and routing rules


Postfix uses mapping databases to route mail efficiently


Even basic mail servers require careful configuration to avoid abuse


Mail services remain a critical component of Linux infrastructure



## 🚀 Future Improvements
Enable TLS for encrypted mail transfer


Configure SMTP authentication (SASL)


Add spam filtering (SpamAssassin)


Enable logging and monitoring


Support external mail relay



## 📌 Conclusion
This project provided hands-on experience with Linux mail server administration, reinforcing foundational skills in system configuration, service management, and troubleshooting.
 Understanding how mail servers work is essential for system administrators, DevOps engineers, and cybersecurity professionals alike.

