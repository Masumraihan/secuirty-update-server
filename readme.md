Here is a clean, professional, production-ready **README.md** you can keep in your repo or internal documentation to secure any DigitalOcean droplet from brute-force attacks, bot scans, and unauthorized SSH access.

---

# 🛡️ **Droplet Security Hardening Guide**

### Protecting Ubuntu + DigitalOcean Droplets from Brute-Force Attacks

*Last updated: 2025*

This guide explains how to fully secure a DigitalOcean droplet using **Fail2Ban**, **UFW firewall**, **SSH hardening**, and optional advanced protections.
These steps prevent brute-force attacks, bot scanning, and unauthorized access.

---

## 📌 **1. Update & Upgrade Server**

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 📌 **2. Enable UFW Firewall**

Allow essential ports only:

```bash
sudo ufw allow 22        # SSH
sudo ufw allow 80        # HTTP
sudo ufw allow 443       # HTTPS
```

Enable firewall:

```bash
sudo ufw enable
```

Check status:

```bash
sudo ufw status
```

---

## 📌 **3. Install & Configure Fail2Ban**

Fail2Ban blocks IPs that attempt repeated failed logins.

### Install:

```bash
sudo apt install fail2ban -y
```

### Create jail override:

```bash
sudo nano /etc/fail2ban/jail.local
```

Add:

```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 1h
findtime = 10m
```

Restart:

```bash
sudo systemctl restart fail2ban
```

Check status:

```bash
sudo fail2ban-client status sshd
```

---

## 📌 **4. Disable SSH Password Login (Critical)**

This step makes brute-force attacks nearly impossible.

### Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and update:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

**Important:** Ensure your SSH key works before disabling passwords.

---

## 📌 **5. Optional Hardening (Highly Recommended)**

### 🔒 **5.1 Disable root login**

In `/etc/ssh/sshd_config`:

```
PermitRootLogin no
```

Restart SSH after editing.

---

### 🔒 **5.2 Limit SSH to your own IP (Maximum Security)**

Replace `<YOUR_IP>`:

```bash
sudo ufw allow from <YOUR_IP> to any port 22
sudo ufw delete allow 22
```

Now only your IP can SSH into the server.

---

### 🔒 **5.3 Change SSH Port (Optional Security by Obscurity)**

Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Change:

```
Port 2222
```

Allow new port:

```bash
sudo ufw allow 2222
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

Reconnect using:

```bash
ssh -p 2222 user@your_ip
```

---

## 📌 **6. Monitor Authentication Logs**

Check recent failed SSH attempts:

```bash
sudo cat /var/log/auth.log | grep "Failed"
```

Real-time monitoring:

```bash
sudo tail -f /var/log/auth.log
```

---

## 📌 **7. Create Automatic MongoDB Backups (Optional but Recommended)**

Create backup directory:

```bash
sudo mkdir -p /var/backups/mongodb
```

Backup script:

```bash
sudo nano /usr/local/bin/mongodb-backup.sh
```

Add:

```bash
#!/bin/bash

MONGO_URI="mongodb://127.0.0.1:27017"
BACKUP_PATH="/var/backups/mongodb"
DATE=$(date +"%Y-%m-%d_%H-%M")
FILENAME="mongodb_backup_$DATE.gz"
DAYS_TO_KEEP=7

mongodump --uri="$MONGO_URI" --archive="$BACKUP_PATH/$FILENAME" --gzip
find $BACKUP_PATH -type f -name "*.gz" -mtime +$DAYS_TO_KEEP -delete
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/mongodb-backup.sh
```

Cron job (daily at 2AM):

```bash
sudo crontab -e
```

Add:

```
0 2 * * * /usr/local/bin/mongodb-backup.sh >> /var/log/mongodb-backup.log 2>&1
```

---

## 📌 **8. Upload Backups to DigitalOcean Spaces (Optional)**

Install AWS CLI:

```bash
sudo apt install awscli -y
```

Configure:

```bash
aws configure
```

Add Spaces endpoint:

```bash
nano ~/.aws/config
```

```
[default]
region = us-east-1
s3 =
    endpoint_url = https://sgp1.digitaloceanspaces.com
```

Upload backup in script:

```bash
aws s3 cp "$BACKUP_PATH/$FILENAME" s3://YOUR_SPACE_NAME/
```

---

## 📌 **9. Verify Everything is Secure**

### Check SSH ports:

```bash
sudo ss -tulnp | grep ssh
```

### Check Fail2Ban:

```bash
sudo fail2ban-client status sshd
```

### Check firewall:

```bash
sudo ufw status
```

---

# ✅ **Your Droplet is Now Fully Protected**

After applying these steps, your server is protected from:

* Brute-force SSH attacks
* Port scanning bots
* Unauthorized login attempts
* Root login vulnerabilities
* Weak password sign-ins
* Accidental data loss (with backups)

---

If you want, I can also generate:

📄 `SECURITY.md`
📄 `server-setup.md`
📄 `recovery-guide.md`
📄 Ansible script to automate everything

Just tell me!
