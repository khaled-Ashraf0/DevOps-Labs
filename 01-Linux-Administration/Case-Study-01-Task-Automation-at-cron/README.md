# Case Study 01: Automated Job Scheduling & Lifecycle Management (`atd` & `cron`)

End-to-end administration case study on **Ubuntu Server** implementing one-time background task execution via `at`, recurring log automation via `crontab`, queue auditing, and schedule cleanup.

---

## 1. Inspect & Enable `atd` Daemon

```bash
# Verify 'at' utility installation
which at

# Check status of the deferred execution scheduler (atd)
sudo systemctl status atd

# Ensure the service is enabled and running
sudo systemctl enable --now atd
```

---

## 2. Schedule & Audit One-Time Background Jobs (`at`, `atq`, `atrm`)

```bash
# Schedule a non-interactive one-time task 2 minutes from current time
echo "date >> /tmp/at-test.txt" | at now + 2 minutes

# List queued one-time jobs to inspect Job ID and scheduled execution time
atq

# Verify execution output after the 2-minute window elapses
ls -l /tmp/at-test.txt
cat /tmp/at-test.txt
```

### Canceling Queued Jobs Before Execution

```bash
# Schedule a test job 5 minutes from now
echo "echo 'Cancelled task' > /tmp/cancel.txt" | at now + 5 minutes

# Capture the Job ID from the active queue
atq

# Remove the pending job from the queue before execution (e.g., Job ID 7)
atrm 7

# Confirm the queue is empty
atq
```

---

## 3. Inspect `cron` Daemon & Configure Recurring Schedules

```bash
# Verify that the cron service is active and running
sudo systemctl status cron

# Schedule a high-frequency job running every minute (* * * * *)
(crontab -l 2>/dev/null; echo "* * * * * date >> /tmp/cron-test.txt") | crontab -

# Schedule an off-peak daily maintenance job at 02:00 AM (0 2 * * *)
(crontab -l 2>/dev/null; echo "0 2 * * * echo 'Daily maintenance completed' >> /tmp/daily-task.txt") | crontab -

# Inspect active user crontab entries
crontab -l
```

### Validating Recurring Execution Output

```bash
# Confirm timestamps are appended every minute
cat /tmp/cron-test.txt
```

---

## 4. Crontab Cleanup & Verification

```bash
# Purge test cron entries to prevent log pollution
crontab -r

# Confirm no active cron jobs remain for the user
crontab -l
```
