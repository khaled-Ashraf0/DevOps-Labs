# Case Study 02: Dynamic Ingress Traffic Control & Perimeter Hardening (`firewalld`)

Cross-distribution (`RHEL` / `Rocky Linux 9` & `Ubuntu 22.04+`) dynamic firewall administration covering trust zone inspection, runtime vs. permanent ingress rules, service/port revocation, `ufw` conflict prevention, and web-tier perimeter hardening.

---

## Part 1: Inspect `firewalld` Daemon State

Verify package installation, inspect service status via `systemd`, and ensure `firewalld` is running and enabled on boot.

```bash
# 1. Verify package installation (RHEL/Rocky vs. Ubuntu/Debian)
rpm -q firewalld              # RHEL / Rocky Linux
dpkg -l | grep firewalld      # Ubuntu / Debian

# 2. Check service status
sudo systemctl status firewalld

# 3. Start and enable at boot if inactive
sudo systemctl enable --now firewalld
```

---

## Part 2: Audit Firewall Trust Zones

List pre-configured zones, identify the active zone bound to the network interface, and inspect its runtime parameters.

```bash
# 1. Display all available zones
sudo firewall-cmd --get-zones

# 2. Identify active zone(s) and bound interface(s)
sudo firewall-cmd --get-active-zones

# 3. Inspect active zone detailed runtime configuration
sudo firewall-cmd --list-all
```

---

## Part 3: Runtime Service Rule Injection

Inject a temporary (`runtime`) traffic exception for the `http` service and verify its presence in the live active zone.

```bash
# 1. Allow HTTP service in runtime configuration
sudo firewall-cmd --add-service=http

# 2. Verify live active zone rules
sudo firewall-cmd --list-services
```

---

## Part 4: Runtime Port Rule Injection

Allow inbound TCP traffic explicitly on custom application port `8080/tcp` in the live runtime engine.

```bash
# 1. Allow inbound TCP traffic on port 8080
sudo firewall-cmd --add-port=8080/tcp

# 2. Verify port in runtime configuration
sudo firewall-cmd --list-ports
```

---

## Part 5: Ingress Rule Revocation

Remove the `http` service exception from the live firewall configuration and verify that it is no longer permitted.

```bash
# 1. Revoke HTTP service exception from live runtime rules
sudo firewall-cmd --remove-service=http

# 2. Confirm HTTP is removed from active zone
sudo firewall-cmd --list-services
```

---

## Part 6: Persistent Policy Configuration

Configure permanent rules written to disk that persist across firewall reloads and system reboots.

```bash
# 1. Permanently add HTTPS service
sudo firewall-cmd --permanent --add-service=https

# 2. Reload firewall engine to apply disk rules to live runtime
sudo firewall-cmd --reload

# 3. Verify HTTPS remains active after reload
sudo firewall-cmd --list-all
```

---

## Part 7: RHEL / Rocky Linux 9 Administration Practice

Execute the end-to-end `firewalld` lifecycle on a RHEL-based node:

```bash
# 1. Verify active zone (typically 'public')
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all

# 2. Apply permanent service & custom port rules
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp

# 3. Reload and verify persistence
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

---

## Part 8: Ubuntu Administration Practice

Deploy `firewalld` on Ubuntu while disabling `ufw` to avoid iptables/nftables rule conflicts:

```bash
# 1. Disable default UFW firewall to prevent rule conflicts
sudo ufw disable
sudo systemctl stop ufw
sudo systemctl disable ufw

# 2. Install firewalld via apt
sudo apt update -y
sudo apt install -y firewalld

# 3. Start and enable firewalld daemon
sudo systemctl enable --now firewalld

# 4. Validate zones and test rule lifecycle
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --add-service=http
sudo firewall-cmd --remove-service=http
```

---

## Final Challenge: Complete Web Tier Perimeter Hardening

Build, apply, and audit the final production web-tier firewall baseline:

* **Allowed Services:** `ssh`, `http`, `https`
* **Allowed Ports:** `8080/tcp`

```bash
# 1. Apply all target services and ports permanently
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp

# 2. Reload firewalld to enforce permanent baseline
sudo firewall-cmd --reload

# 3. Final audit & state verification
sudo firewall-cmd --list-all
```

**Expected Audit Output (`sudo firewall-cmd --list-all`):**
```text
public (active)
  target: default
  interfaces: ens33
  services: ssh http https
  ports: 8080/tcp
```
<img width="451" height="227" alt="image" src="https://github.com/user-attachments/assets/1c3519c9-b829-4c84-8e20-921ffb91179a" />
