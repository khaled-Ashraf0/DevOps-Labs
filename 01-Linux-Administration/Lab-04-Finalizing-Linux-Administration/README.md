# Lab 04: Finalizing Linux Administration & Bash Automation

End-to-end Linux system administration capstone scenario covering user and group provisioning, directory ACLs (`770`), SSH port hardening, `nginx` & `firewalld` management, automated disk health monitoring in Bash, `cron`/`at` job scheduling, and privilege auditing[cite: 20, 21, 22].

---

## Part 1: User, Group, and Permission Management

### Task 1.1: User and Group Provisioning
Create the `sec-audit` and `analytics` groups, provision `audit-lead` and `data-analyst` with `/bin/bash` shells, and set their initial passwords[cite: 21].

```bash
# 1. Create system groups
sudo groupadd sec-audit
sudo groupadd analytics

# 2. Create user accounts assigned to their respective primary groups
sudo useradd -m -s /bin/bash -g sec-audit audit-lead
sudo useradd -m -s /bin/bash -g analytics data-analyst

# 3. Set initial passwords
sudo passwd audit-lead
sudo passwd data-analyst
```

### Task 1.2: Directory Ownership & Access Control
Create `/srv/analytics_data`, assign ownership to `audit-lead:analytics`, and restrict permissions to `770` (full access for owner and group, none for others)[cite: 21].

```bash
sudo mkdir -p /srv/analytics_data
sudo chown audit-lead:analytics /srv/analytics_data
sudo chmod 770 /srv/analytics_data

# Verify directory ownership and permissions
ls -ld /srv/analytics_data
```
<img width="958" height="172" alt="Screenshot 2026-11-28 010938" src="https://github.com/user-attachments/assets/a60c5d2c-ecc6-4be5-a9df-11e3c0aea1af" />

---

## Part 2: SSH Hardening, Service Control & Firewalld

### Task 2.1: Secure SSH Configuration
Change the SSH daemon listening port from `22` to `2223` and restart the SSH service[cite: 21].

**1. Edit SSH configuration file:**
```bash
sudo vim /etc/ssh/sshd_config
```

**2. Update/uncomment the Port directive:**
```text
Port 2223
```

**3. Validate syntax and restart SSH service:**
```bash
sudo sshd -t
sudo systemctl restart ssh
```
*(Note: On Ubuntu 22.04+, if `ssh.socket` is active, reload daemon and restart `ssh.socket` as well: `sudo systemctl daemon-reload && sudo systemctl restart ssh.socket`)*

### Task 2.2: Package Installation & Service Lifecycle
Install `nginx` and `firewalld`, then start and enable both services at boot[cite: 21].

```bash
sudo apt update -y
sudo apt install -y nginx firewalld

sudo systemctl enable --now nginx
sudo systemctl enable --now firewalld
```

### Task 2.3: Firewall Ingress Rules
Permit inbound traffic on custom SSH port `2223/tcp` and HTTP port `80/tcp`, then reload `firewalld` permanently[cite: 21].

```bash
sudo firewall-cmd --permanent --add-port=2223/tcp
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify active rules
sudo firewall-cmd --list-all
```
<img width="566" height="312" alt="Screenshot 2026-11-28 011441" src="https://github.com/user-attachments/assets/7182dd2f-2367-4a8c-96e5-26dc7d426205" />

---

## Part 3: Intermediate Bash Automation Scripting

### Task 3.1: Disk Health Script Construction (`/usr/local/bin/disk_checker.sh`)
Monitor root filesystem (`/`) usage against an `80%` threshold and append timestamped logs and errors to `/var/log/disk_health.log`[cite: 21].

**1. Create the script file:**
```bash
sudo vim /usr/local/bin/disk_checker.sh
```

**2. Script content:**
```bash
#!/bin/bash

LOG_FILE="/var/log/disk_health.log"
CURRENT_TIME=$(date "+%Y-%m-%d %H:%M:%S")
THRESHOLD=80

check_disk_space() {
    local usage
    usage=$(df -h / | awk 'NR==2 {gsub("%",""); print $5}')

    if [ "$usage" -gt "$THRESHOLD" ]; then
        echo "[$CURRENT_TIME] High Disk Usage Warning: Root partition at ${usage}%"
    else
        echo "[$CURRENT_TIME] Disk Usage Normal: Root partition at ${usage}%"
    fi
}

check_disk_space >> "$LOG_FILE" 2>> "$LOG_FILE"
```

**3. Set executable permissions & test run:**
```bash
sudo chmod +x /usr/local/bin/disk_checker.sh
sudo /usr/local/bin/disk_checker.sh
cat /var/log/disk_health.log
```
<img width="731" height="210" alt="Screenshot 2026-11-28 012204" src="https://github.com/user-attachments/assets/24bb8c7e-b798-4879-ad39-30b8fa54f93c" />

---

## Part 4: Task Scheduling (`cron` & `at`)

### Task 4.1: Recurring Task Automation with Cron
Schedule `/usr/local/bin/disk_checker.sh` to run automatically every night at 11:00 PM.

**1. Open root crontab:**
```bash
sudo crontab -e
```

**2. Add the cron schedule entry:**
```cron
0 23 * * * /usr/local/bin/disk_checker.sh
```

### Task 4.2: One-Time Task Scheduling with At
Schedule a one-time compressed archive of `/var/log` to `/tmp/logs_backup.tar.gz` 15 minutes from execution[cite: 22].

```bash
# Ensure atd service is running
sudo systemctl enable --now atd

# Schedule one-time backup job
echo "tar -czf /tmp/logs_backup.tar.gz /var/log" | sudo at now + 15 minutes

# Verify scheduled job queue
sudo atq
```
<img width="1012" height="155" alt="Screenshot 2026-11-28 012611" src="https://github.com/user-attachments/assets/4be8b64e-119f-4a28-838d-9d3bf39e8ed7" />

---

## Part 5: Permission Validation & Verification Checklist

Verification matrix validating access control boundaries across `data-analyst`, `audit-lead`, and unprivileged users[cite: 22]:
```bash
| Test Case | User Context | Command Executed | Expected Behavior | Actual Behavior |
| :--- | :--- | :--- | :--- | :--- |
| **Test Case 1** | `data-analyst` | `touch /srv/analytics_data/report.csv` | **Success:** `data-analyst` belongs to owning group `analytics` with `rwx` (`770`) permissions[cite: 21, 22]. | File `/srv/analytics_data/report.csv` created without errors[cite: 22]. |
| **Test Case 2** | `data-analyst` | `systemctl restart nginx` | **Denied:** Non-root user without `sudo` privileges cannot manage system services[cite: 21, 22]. | Prompted for authentication / `Interactive authentication required` (Access Denied)[cite: 22]. |
| **Test Case 3** | `audit-lead` | `ssh -p 2223 audit-lead@localhost` | **Success:** SSH daemon listens on port `2223` and firewall permits `2223/tcp`[cite: 21, 22]. | Authenticated and opened an interactive shell session on port `2223`[cite: 22]. |
| **Test Case 4** | `audit-lead` | `sudo /usr/local/bin/disk_checker.sh` | **Denied (unless in sudoers):** Standard user not in `sudo` group cannot run root commands[cite: 21, 22]. | `audit-lead is not in the sudoers file. This incident will be reported.`[cite: 22] |
| **Test Case 5** | Unprivileged User (`other`) | `cat /srv/analytics_data/report.csv` | **Denied:** Directory permissions (`770`) block all access for `others`[cite: 21, 22]. | `cat: /srv/analytics_data/report.csv: Permission denied`[cite: 22]. |
```
<img width="1532" height="270" alt="Screenshot 2026-11-28 013145" src="https://github.com/user-attachments/assets/ea2c4a2b-da24-4ca3-850b-94bece2f2244" />

