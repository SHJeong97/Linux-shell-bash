# ⏰ Bash Task Scheduler using Cron

## 📘 Project Overview
This project implements a **command-line task scheduler** using **Bash scripting and Linux cron**.  
The script allows users to **list**, **add**, and **remove scheduled tasks** through a simple interactive menu.

The goal of this project is to demonstrate practical Linux automation skills, user input handling, and real-world task scheduling using `crontab`.

---

## 🎯 Objectives
- Build a Bash script from scratch
- Interact with the system `crontab`
- Schedule tasks using human-readable intervals
- Manage cron jobs safely without overwriting existing tasks
- Create a reusable, menu-driven CLI tool

---

## 🛠️ Tools & Technologies
- **Language:** Bash
- **Scheduler:** cron / crontab
- **Environment:** Linux (Ubuntu-based)
- **Commands Used:** `crontab`, `grep`, `read`, `case`, `chmod`

---

## ⚙️ Core Features
- 📋 List all scheduled cron tasks
- ➕ Add new tasks (hourly / daily / weekly)
- ❌ Remove tasks by command name
- 🔁 Interactive menu loop
- 🧠 Preserves existing cron jobs safely

---

## 🧠 Script Design

### List Scheduled Tasks
```bash
list_tasks() {
  echo "Scheduled tasks:"
  crontab -l
  echo
}
```
Displays the current user’s cron jobs


Uses crontab -l to avoid direct file manipulation



### Add a New Task
```bash
add_task() {
  read -p "Enter the command or script to be executed: " command
  read -p "Enter the schedule (hourly, daily, weekly): " schedule
  read -p "Enter any additional parameters: " parameters

  case $schedule in
    hourly) cron_schedule="0 * * * *" ;;
    daily)  cron_schedule="0 0 * * *" ;;
    weekly) cron_schedule="0 0 * * 0" ;;
    *)
      echo "Invalid schedule."
      return
      ;;
  esac

  (
    crontab -l 2>/dev/null
    echo "$cron_schedule $command $parameters"
  ) | crontab -

  echo "Task scheduled successfully."
}
```
Key concepts demonstrated:
User input handling


Mapping human-friendly schedules to cron syntax


Safely appending tasks without deleting existing cron jobs


Error suppression when no crontab exists



### Remove a Task
```bash
remove_task() {
  read -p "Enter the command or script to be removed: " command
  crontab -l | grep -v "$command" | crontab -
  echo "Task removed successfully."
}
```
Removes tasks based on command matching


Demonstrates filtering with grep -v


Highlights risks of pattern-based deletion (discussed in improvements)



### Interactive Menu Loop
```bash
while true; do
  echo "Task Scheduler"
  echo "1. List scheduled tasks"
  echo "2. Add a task"
  echo "3. Remove a task"
  echo "4. Exit"
  read -p "Enter your choice: " choice

  case $choice in
    1) list_tasks ;;
    2) add_task ;;
    3) remove_task ;;
    4) break ;;
    *) echo "Invalid choice." ;;
  esac
done
```
Continuous menu-driven interface


Uses case for clean control flow


Easy to extend with new features



## 🚀 How to Run
```bash
chmod +x task_scheduler.sh
./task_scheduler.sh
```

## 🔐 Security & Operations Relevance
This project reflects real-world Linux administration tasks, including:
Automation of repetitive jobs


Safe interaction with system schedulers


CLI tooling for DevOps and SOC environments


Understanding cron misconfigurations and risks


Improper cron usage is a common source of:
Privilege escalation


Persistence mechanisms


Misconfigured automation



## 🧪 Limitations & Future Improvements
Improve task removal with indexed selection


Add input validation for commands


Support custom cron expressions


Add logging of task changes


Implement role-based execution (sudo checks)



## 📌 Key Takeaway
This project demonstrates how Bash scripting + cron can be combined to create practical automation tools.
 It highlights core Linux skills used in system administration, DevOps, and cybersecurity operations.

