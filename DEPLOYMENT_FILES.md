# Home Storage App - Deployment Package Summary

This document lists all files included in the deployment package for Raspberry Pi.

---

## Complete File Structure

```
simple-image-app/
│
├── backend/
│   ├── app.py                          # Flask REST API server
│   ├── requirements.txt                # Python dependencies (Flask, Flask-CORS, Pillow)
│   └── data.db                         # SQLite database (created on first run)
│
├── frontend/
│   └── index.html                      # Web interface with responsive design
│
├── deploy/
│   ├── systemd/
│   │   └── homestorage.service         # Systemd service for auto-start
│   │
│   ├── install.sh                      # Automated installation script
│   ├── backup.sh                       # Database backup script
│   ├── generate-cert.sh                # SSL certificate generator
│   ├── generate-cert.bat               # SSL generator for Windows (testing)
│   └── CHECKLIST.md                    # Deployment checklist
│
├── backups/                            # Backup storage (created on first backup)
├── ssl/                                # SSL certificates (created when HTTPS enabled)
│
├── DEPLOY.md                           # Full deployment guide
├── INSTALL.md                          # Quick installation guide
├── README.md                           # User documentation
├── requirements.md                     # Requirements documentation
└── DEPLOYMENT_FILES.md                 # This file
```

---

## File Descriptions

### Core Application Files

| File | Purpose | Size |
|------|---------|------|
| `backend/app.py` | Flask REST API with SQLite, image compression, password management | ~12KB |
| `backend/requirements.txt` | Python package dependencies | ~100 bytes |
| `frontend/index.html` | Complete web UI with all features | ~180KB |

### Deployment Scripts

| File | Purpose | Platform |
|------|---------|----------|
| `deploy/install.sh` | Automated one-command installer | Linux/Raspberry Pi |
| `deploy/backup.sh` | Database backup with rotation | Linux/Raspberry Pi |
| `deploy/generate-cert.sh` | SSL certificate generator | Linux/Raspberry Pi |
| `deploy/generate-cert.bat` | SSL certificate generator | Windows (testing) |
| `deploy/systemd/homestorage.service` | Systemd service configuration | Linux/Raspberry Pi |

### Documentation

| File | Purpose |
|------|---------|
| `DEPLOY.md` | Complete deployment guide with all steps |
| `INSTALL.md` | Quick start installation guide |
| `README.md` | User documentation and feature list |
| `requirements.md` | Technical requirements and revision history |
| `deploy/CHECKLIST.md` | Step-by-step deployment checklist |
| `DEPLOYMENT_FILES.md` | This file - deployment package summary |

---

## Quick Deploy Commands

### Option 1: Automated Installation

Copy files to Raspberry Pi, then run:

```bash
cd /home/pi/simple-image-app
bash deploy/install.sh
```

### Option 2: Manual Installation

Follow the step-by-step guide in `DEPLOY.md`.

---

## System Requirements

### Minimum Hardware
- Raspberry Pi 3 Model B (or newer)
- 8GB microSD card
- 5V 2.5A power supply
- Network connection (Ethernet or WiFi)

### Recommended Hardware
- Raspberry Pi 4 Model B (2GB+)
- 16GB+ microSD card (Class 10)
- Official Raspberry Pi power supply
- Ethernet connection for stability

### Software
- Raspberry Pi OS (32-bit or 64-bit)
- Python 3.7+
- Network access for initial setup

---

## Resource Usage

### Memory
- Idle: ~50MB RAM
- Active: ~80MB RAM
- With image compression: ~120MB RAM

### Storage
- Application: ~5MB
- Database: Varies (estimate 1-5KB per record with compressed images)
- System overhead: ~10MB
- **Total minimum**: ~20MB (excluding OS)

### CPU
- Idle: <1%
- Active usage: 5-20%
- Image compression: 30-60% (brief spikes)

---

## Network Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 5000 | HTTP | Web interface (default) |
| 5000 | HTTPS | Web interface (when HTTPS enabled) |
| 22 | SSH | Remote administration |

---

## Default Credentials

| Service | Username | Password | Notes |
|---------|----------|----------|-------|
| Web App | (none) | `1234` | Change after first login |
| SSH | `pi` | (set during OS install) | Use SSH keys in production |
| Database | N/A | N/A | SQLite file-based |

---

## Backup Strategy

### Automated Backups
```bash
# Add to crontab for daily 2 AM backups
crontab -e
0 2 * * * /home/pi/simple-image-app/deploy/backup.sh
```

### Manual Backup
```bash
/home/pi/simple-image-app/deploy/backup.sh
```

### Backup Location
`/home/pi/simple-image-app/backups/`

### Retention Policy
- Last 10 backups automatically retained
- Older backups automatically deleted

---

## Security Considerations

### Current Security Level: Basic

The application includes:
- Password authentication (stored in database)
- Optional HTTPS with self-signed certificate
- Local network only (no internet exposure by default)

### Recommended Additional Security

For production deployment:
1. Change default password immediately
2. Enable HTTPS with trusted certificate (Let's Encrypt)
3. Use SSH key authentication instead of password
4. Configure firewall (UFW)
5. Regular system updates
6. Regular database backups
7. Consider VPN for remote access instead of port forwarding

### NOT Recommended
- Exposing the app directly to the internet without proper security
- Using self-signed certificates for sensitive data
- Default passwords in production

---

## Update Procedures

### Application Update
```bash
cd /home/pi/simple-image-app
# If using Git:
git pull
# Or copy new files manually

sudo systemctl restart homestorage
```

### System Update
```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

### Python Package Update
```bash
cd /home/pi/simple-image-app/backend
pip3 install --upgrade -r requirements.txt
sudo systemctl restart homestorage
```

---

## Monitoring and Logging

### View Logs
```bash
# Real-time
journalctl -u homestorage -f

# Last 100 lines
journalctl -u homestorage -n 100

# Today's logs
journalctl -u homestorage --since today
```

### Check Service Status
```bash
sudo systemctl status homestorage
```

### Test API Health
```bash
curl http://localhost:5000/api/health
```

---

## Contact and Support

### Log Files Location
- Systemd logs: `journalctl -u homestorage`
- Application logs: Included in systemd logs
- Backup logs: Console output from backup script

### Diagnostic Commands
```bash
# Service status
sudo systemctl status homestorage

# Network status
hostname -I
ip addr show

# Disk usage
df -h

# Memory usage
free -h

# Process status
ps aux | grep python
```

---

## Version Information

| Component | Version |
|-----------|---------|
| Flask | 3.0.0 |
| Flask-CORS | 4.0.0 |
| Pillow | 10.1.0 |
| Python | 3.7+ |
| SQLite | 3.x (included with Python) |

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-03-30 | 1.0 | Initial release |
| 2026-03-31 | 1.1 | Added client-server architecture |
| 2026-03-31 | 1.2 | Added image compression (<50KB) |
| 2026-03-31 | 1.3 | Added password in database |
| 2026-03-31 | 1.4 | Added HTTPS support |
| 2026-03-31 | 1.5 | Added Zoom/Download functions |
| 2026-03-31 | 1.6 | Complete deployment package |
