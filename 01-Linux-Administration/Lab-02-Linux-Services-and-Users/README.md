# Lab 02: Linux Package, Service & User Management

Hands-on administration tasks on **Ubuntu Server** covering APT package removal and cleanup, `systemd` service auditing, OpenSSH server deployment, and user password policy management.

---

## 1. Package Removal & Dependency Cleanup (`apt`)

```bash
# Purge nginx along with its configuration files
sudo apt purge -y nginx

# Remove unused residual dependencies
sudo apt autoremove -y
```

---

## 2. Auditing `systemd` Services

```bash
# Export all currently active services to a text file
systemctl list-units --type=service --state=active > active.txt

# Inspect the top 10 active services
head -n 10 active.txt

# Export any failed services to verify system health
systemctl list-units --type=service --state=failed > failed.txt
cat failed.txt
```

---

## 3. Installing & Enabling OpenSSH Server

```bash
# Update local package index and install openssh-server
sudo apt update && sudo apt install -y openssh-server

# Enable SSH service to start on boot and start it immediately
sudo systemctl enable --now ssh

# Verify service status and listening port (22)
sudo systemctl status ssh
```

---

## 4. User Account Password & Status Management

```bash
# Check the current password lock status of 'devops_user' (L = Locked)
sudo passwd -S devops_user

# Set a compliant password (minimum 8 characters)
sudo passwd devops_user

# Verify the account status changed to active password (P = Password set)
sudo passwd -S devops_user
```
