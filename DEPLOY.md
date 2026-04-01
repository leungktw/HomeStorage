# Home Storage App - Complete Deployment Guide

Deploy the Home Storage Inventory Management System on Raspberry Pi, Ubuntu, or Windows.

---

## Table of Contents

1. [File Structure](#file-structure)
2. [Raspberry Pi Deployment](#raspberry-pi-deployment)
3. [Ubuntu Deployment](#ubuntu-deployment)
4. [Windows Deployment](#windows-deployment)
5. [Post-Installation Configuration](#post-installation-configuration)
6. [Maintenance Guide](#maintenance-guide)

---

## File Structure

```
simple-image-app/
├── backend/
│   ├── app.py                          # Flask REST API server
│   └── requirements.txt                # Python dependencies
├── frontend/
│   └── index.html                      # Web interface
├── deploy/
│   ├── systemd/
│   │   └── homestorage.service         # Systemd service (Linux only)
│   ├── install.sh                      # Automated installer (Linux)
│   ├── backup.sh                       # Backup script (Linux)
│   ├── generate-cert.sh                # SSL generator (Linux)
│   ├── install.bat                     # Windows installer
│   ├── backup.bat                      # Windows backup
│   └── generate-cert.bat               # Windows SSL generator
├── DEPLOY.md                           # This file
├── README.md                           # User documentation
└── requirements.md                     # Technical requirements
```

---

## Raspberry Pi Deployment

### Prerequisites

- Raspberry Pi 3 or later
- Raspberry Pi OS (32-bit or 64-bit)
- 8GB+ microSD card
- Network connection

### Step 1: Transfer Files

**Option A: SCP from another computer**
```bash
# From the computer with the files
scp -r simple-image-app pi@<raspberry-pi-ip>:/home/admin/
```

**Option B: Git clone**
```bash
cd /home/admin
git clone <repository-url> simple-image-app
```

### Step 2: Run Automated Installer

```bash
cd /home/admin/simple-image-app
bash deploy/install.sh
```

The installer will:
- Update system packages
- Install Python dependencies
- Create directory structure
- Setup systemd service
- Display access information

### Step 3: Fix Paths (if user is not "pi")

```bash
# Replace 'pi' with your username
sudo sed -i 's|/home/pi|/home/admin|g' /etc/systemd/system/homestorage.service
sudo sed -i 's/User=pi/User=admin/g' /etc/systemd/system/homestorage.service
sudo systemctl daemon-reload
```

### Step 4: Install Python Dependencies

```bash
# Install Pillow from system packages (avoids build errors)
sudo apt install -y python3-pillow

# Install Flask packages
pip3 install --break-system-packages flask==3.0.0 flask-cors==4.0.0
```

### Step 5: Start Service

```bash
sudo systemctl enable homestorage
sudo systemctl start homestorage
sudo systemctl status homestorage
```

### Step 6: Access Application

```
http://<raspberry-pi-ip>:5000
Default password: 1234
```

### Step 7: Enable HTTPS (Optional)

```bash
# Generate 10-year self-signed certificate
cd /home/admin/simple-image-app/ssl
sudo openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 3650 -nodes \
  -subj "/C=CN/ST=Home/L=Home/O=HomeStorage/CN=$(hostname -I | awk '{print $1}')"
sudo chmod 600 key.pem
sudo chmod 644 cert.pem
sudo chown admin:admin *.pem

# Enable HTTPS in app
sed -i 's/USE_HTTPS = False/USE_HTTPS = True/' /home/admin/simple-image-app/backend/app.py

# Restart service
sudo systemctl restart homestorage
```

Access via: `https://<raspberry-pi-ip>:5000`

---

## Ubuntu Deployment

### Prerequisites

- Ubuntu 20.04 or later
- Python 3.8+
- Network access

### Step 1: Transfer Files

```bash
# Copy files to Ubuntu
cp -r simple-image-app /home/<username>/
cd /home/<username>/simple-image-app
```

### Step 2: Install Dependencies

```bash
# Update system
sudo apt update

# Install Python and system packages
sudo apt install -y python3 python3-pip python3-pillow openssl

# Install Flask packages
pip3 install --break-system-packages -r backend/requirements.txt
```

### Step 3: Initialize Database

```bash
cd backend
python3 app.py &
sleep 2
kill %1
```

### Step 4: Setup Systemd Service

```bash
# Update service file with correct username
sed -i "s|/home/pi|/home/$USER|g" deploy/systemd/homestorage.service
sed -i "s/User=pi/User=$USER/g" deploy/systemd/homestorage.service

# Copy and enable service
sudo cp deploy/systemd/homestorage.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable homestorage
sudo systemctl start homestorage
sudo systemctl status homestorage
```

### Step 5: Access Application

```
http://<ubuntu-ip>:5000
Default password: 1234
```

### Step 6: Enable HTTPS (Optional)

```bash
# Create SSL directory
mkdir -p ssl
cd ssl

# Generate certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 3650 -nodes \
  -subj "/C=CN/ST=Home/L=Home/O=HomeStorage/CN=$(hostname -I | awk '{print $1}')"
chmod 600 key.pem
chmod 644 cert.pem

# Enable HTTPS
cd ../backend
sed -i 's/USE_HTTPS = False/USE_HTTPS = True/' app.py

# Restart service
sudo systemctl restart homestorage
```

---

## Windows Deployment

### Prerequisites

- Windows 10/11
- Python 3.8+ from https://python.org
- Administrator privileges

### Step 1: Install Python

1. Download Python from https://python.org/downloads/
2. Run installer
3. ✓ Check "Add Python to PATH"
4. Click "Install Now"

### Step 2: Transfer Files

Copy `simple-image-app` folder to:
```
C:\Users\<YourUsername>\simple-image-app
```

### Step 3: Run Windows Installer

Open Command Prompt as Administrator:

```cmd
cd C:\Users\<YourUsername>\simple-image-app
deploy\install.bat
```

### Step 4: Manual Installation (if needed)

```cmd
# Navigate to backend
cd C:\Users\<YourUsername>\simple-image-app\backend

# Install dependencies
pip install -r requirements.txt

# Initialize database
python app.py
```

Leave this window open to keep the server running, or install as Windows Service (see below).

### Step 5: Install as Windows Service (Optional)

Install NSSM (Non-Sucking Service Manager):

```cmd
# Download NSSM from https://nssm.cc/download
# Extract to C:\nssm

# Create Windows service
C:\nssm\nssm.exe install HomeStorage "C:\Python39\python.exe" "C:\Users\<YourUsername>\simple-image-app\backend\app.py"

# Set service directory
C:\nssm\nssm.exe set HomeStorage AppDirectory "C:\Users\<YourUsername>\simple-image-app\backend"

# Start service
C:\nssm\nssm.exe start HomeStorage
```

### Step 6: Access Application

```
http://localhost:5000
or
http://<your-windows-ip>:5000
Default password: 1234
```

To find your IP:
```cmd
ipconfig
```

### Step 7: Enable HTTPS (Optional)

Generate certificate:

```cmd
cd C:\Users\<YourUsername>\simple-image-app\ssl
deploy\generate-cert.bat
```

Enable HTTPS in app:

Edit `backend\app.py`, find line ~378:
```python
USE_HTTPS = True  # Change from False to True
```

Restart the service/server.

Access via: `https://localhost:5000` or `https://<your-windows-ip>:5000`

---

## Post-Installation Configuration

### Change Default Password

1. Login with password `1234`
2. Click logout button (top right)
3. Click "Change Password"
4. Enter current password and new password
5. New password is now stored in database

### Setup Automated Backups

**Linux (Raspberry Pi/Ubuntu):**

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /home/admin/simple-image-app/deploy/backup.sh
```

**Windows:**

1. Open Task Scheduler
2. Create Basic Task
3. Name: "Home Storage Backup"
4. Trigger: Daily at 2:00 AM
5. Action: Start a program
   - Program: `C:\Users\<Username>\simple-image-app\deploy\backup.bat`
6. Finish

### Find Your IP Address

**Raspberry Pi/Ubuntu:**
```bash
hostname -I
```

**Windows:**
```cmd
ipconfig
```

---

## Maintenance Guide

### Service Management

**Linux (systemd):**
```bash
sudo systemctl start homestorage
sudo systemctl stop homestorage
sudo systemctl restart homestorage
sudo systemctl status homestorage
```

**Windows:**
```cmd
# If using NSSM service
nssm start HomeStorage
nssm stop HomeStorage
nssm restart HomeStorage

# Or via Services app (services.msc)
```

### View Logs

**Linux:**
```bash
journalctl -u homestorage -f      # Real-time
journalctl -u homestorage -n 50   # Last 50 lines
```

**Windows:**
- Check Event Viewer → Applications
- Or view console output if running manually

### Backup Database

**Linux:**
```bash
/home/admin/simple-image-app/deploy/backup.sh
```

**Windows:**
```cmd
C:\Users\<Username>\simple-image-app\deploy\backup.bat
```

### Restore from Backup

**Linux:**
```bash
sudo systemctl stop homestorage
cp /home/admin/simple-image-app/backups/data_backup_YYYYMMDD_HHMMSS.db \
   /home/admin/simple-image-app/backend/data.db
sudo systemctl start homestorage
```

**Windows:**
```cmd
# Stop service first
net stop HomeStorage

copy C:\Users\<Username>\simple-image-app\backups\data_backup_*.db \
     C:\Users\<Username>\simple-image-app\backend\data.db

net start HomeStorage
```

### Update Application

**Linux:**
```bash
cd /home/admin/simple-image-app
git pull  # if using Git
sudo systemctl restart homestorage
```

**Windows:**
```cmd
cd C:\Users\<Username>\simple-image-app
git pull  # if using Git
nssm restart HomeStorage
```

### Troubleshooting

**Service won't start:**
```bash
# Linux
journalctl -u homestorage -n 100 --no-pager

# Windows
# Check Event Viewer or run manually:
python backend\app.py
```

**Can't access from network:**
1. Verify service is running
2. Check firewall settings
3. Verify IP address
4. Test locally: `curl http://localhost:5000/api/health`

**Database issues:**
```bash
# Check database integrity
python3 -c "
import sqlite3
conn = sqlite3.connect('backend/data.db')
cursor = conn.execute('PRAGMA integrity_check')
print('Database:', cursor.fetchone()[0])
conn.close()
"
```

---

## Quick Reference

### Default Credentials
- Login password: `1234` (change after first login)
- SSH (Linux): `pi` / `<your-password>` or `admin` / `<your-password>`

### Ports
- HTTP: 5000
- HTTPS: 5000
- SSH: 22

### Important Paths

**Linux:**
- Application: `/home/admin/simple-image-app/`
- Database: `/home/admin/simple-image-app/backend/data.db`
- Backups: `/home/admin/simple-image-app/backups/`
- Logs: `journalctl -u homestorage`

**Windows:**
- Application: `C:\Users\<Username>\simple-image-app\`
- Database: `C:\Users\<Username>\simple-image-app\backend\data.db`
- Backups: `C:\Users\<Username>\simple-image-app\backups\`

---

## Support

For issues, check:
1. Service status
2. Application logs
3. Database integrity
4. Network connectivity

See troubleshooting section above for specific commands.
