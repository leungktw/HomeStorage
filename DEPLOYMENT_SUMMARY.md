# Home Storage App - Deployment Package Summary

Complete deployment package for Home Storage Inventory Management System.

---

## Package Contents

### Core Application Files

```
simple-image-app/
├── backend/
│   ├── app.py                          # Flask REST API (400 lines)
│   └── requirements.txt                # Python dependencies
├── frontend/
│   └── index.html                      # Web UI (2000+ lines)
└── deploy/                             # Deployment scripts
```

### Deployment Scripts

| File | Platform | Purpose |
|------|----------|---------|
| `deploy/install.sh` | Linux (Raspberry Pi/Ubuntu) | Automated installation |
| `deploy/backup.sh` | Linux | Database backup with rotation |
| `deploy/generate-cert.sh` | Linux | SSL certificate generation |
| `deploy/install.bat` | Windows | Automated installation |
| `deploy/backup.bat` | Windows | Database backup |
| `deploy/generate-cert.bat` | Windows | SSL certificate generation |
| `deploy/systemd/homestorage.service` | Linux | Systemd auto-start service |

### Documentation

| File | Description |
|------|-------------|
| `DEPLOY.md` | Complete deployment guide for all platforms |
| `INSTALL.md` | Quick start guide |
| `README.md` | User documentation |
| `requirements.md` | Technical requirements |
| `DEPLOYMENT_SUMMARY.md` | This file |

---

## Quick Start Guide

### Raspberry Pi / Ubuntu (Linux)

**Step 1: Copy Files**
```bash
scp -r simple-image-app admin@<ip>:/home/admin/
```

**Step 2: Run Installer**
```bash
cd /home/admin/simple-image-app
bash deploy/install.sh
```

**Step 3: Access App**
```
http://<ip>:5000
Password: 1234
```

---

### Windows

**Step 1: Copy Files**
```
Copy to: C:\Users\<YourName>\simple-image-app
```

**Step 2: Run Installer**
```cmd
cd C:\Users\<YourName>\simple-image-app
deploy\install.bat
```

**Step 3: Access App**
```
http://localhost:5000
Password: 1234
```

---

## Features

### Core Features
- CRUD operations for inventory items
- Image upload with automatic compression (<50KB)
- Full-text search with wildcard support
- Date filtering (items >= selected date)
- Password authentication (stored in database)
- SQLite database (single file, no server needed)

### UI Features
- Responsive design (desktop, tablet, mobile)
- Modern gradient UI with animations
- Image zoom (fullscreen view)
- Image download
- Confirmation dialogs for destructive actions
- Server connection status indicator

### Security Features
- Password-protected access
- Optional HTTPS (self-signed or CA certificates)
- Local network only (no internet exposure by default)

---

## Technical Specifications

### Backend
- **Framework**: Flask 3.0.0
- **Database**: SQLite 3.x
- **Image Processing**: Pillow 10.x
- **CORS**: Flask-CORS 4.0.0
- **Python**: 3.8+

### Frontend
- **HTML5**: Single page application
- **CSS3**: Flexbox, Grid, variables, animations
- **JavaScript**: Vanilla ES6+
- **Responsive**: 3 breakpoints (desktop, 600px, 360px)

### API Endpoints
```
GET  /api/health          - Health check
GET  /api/items           - Get all items
GET  /api/items/search    - Search with filters
POST /api/items           - Add item
PUT  /api/items/<id>      - Update item
DELETE /api/items/<id>    - Delete item
POST /api/items/batch-delete - Batch delete
GET  /api/stats           - Database statistics
GET  /api/password        - Get password
PUT  /api/password        - Update password
POST /api/compress-image  - Compress image
```

---

## System Requirements

### Minimum Hardware
- **CPU**: 1 GHz dual-core
- **RAM**: 512 MB
- **Storage**: 100 MB + database space
- **Network**: Ethernet or WiFi

### Recommended Hardware
- **CPU**: 1.5 GHz quad-core (Raspberry Pi 4 or better)
- **RAM**: 2 GB
- **Storage**: 16 GB microSD card
- **Network**: Gigabit Ethernet

### Operating Systems
- Raspberry Pi OS (32/64-bit)
- Ubuntu 20.04+
- Windows 10/11

---

## Resource Usage

### Memory
- **Idle**: ~50 MB RAM
- **Active**: ~80 MB RAM
- **Image compression**: ~120 MB RAM (brief spikes)

### CPU
- **Idle**: <1%
- **Active**: 5-20%
- **Image compression**: 30-60% (brief spikes)

### Storage
- **Application**: ~5 MB
- **Database**: 1-5 KB per record (with compressed images)
- **Backups**: ~50 KB each (auto-managed)

---

## Deployment Checklist

### Pre-Deployment
- [ ] Files transferred to target machine
- [ ] Python 3.8+ installed
- [ ] Network access configured
- [ ] Administrator/root access available

### Installation
- [ ] Dependencies installed
- [ ] Database initialized
- [ ] Service/daemon configured
- [ ] Service started successfully
- [ ] Web interface accessible

### Post-Installation
- [ ] Default password changed
- [ ] HTTPS enabled (optional)
- [ ] Automated backups configured
- [ ] Firewall rules configured
- [ ] Documentation stored safely

### Verification
- [ ] Add record with image works
- [ ] Search function works
- [ ] Image compression works (<50KB)
- [ ] Image zoom/download works
- [ ] Password change works
- [ ] Service survives reboot

---

## Common Issues & Solutions

### Python 3.13 + Pillow Build Error
**Issue**: `KeyError: '_version_'` during Pillow installation

**Solution**:
```bash
# Use system package instead
sudo apt install python3-pillow
pip3 install --break-system-packages flask flask-cors
```

### Externally Managed Environment (PEP 668)
**Issue**: `error: externally-managed-environment`

**Solution**:
```bash
pip3 install --break-system-packages <package>
```

### Service Won't Start (Wrong Paths)
**Issue**: User is `admin` but service expects `pi`

**Solution**:
```bash
sudo sed -i 's|/home/pi|/home/admin|g' /etc/systemd/system/homestorage.service
sudo sed -i 's/User=pi/User=admin/g' /etc/systemd/system/homestorage.service
sudo systemctl daemon-reload
sudo systemctl restart homestorage
```

### Missing Module (flask_cors)
**Issue**: `ModuleNotFoundError: No module named 'flask_cors'`

**Solution**:
```bash
cd /path/to/backend
pip3 install --break-system-packages flask-cors
sudo systemctl restart homestorage
```

---

## Maintenance Commands

### Linux (systemd)
```bash
# Service management
sudo systemctl start homestorage
sudo systemctl stop homestorage
sudo systemctl restart homestorage
sudo systemctl status homestorage

# Logs
journalctl -u homestorage -f      # Follow logs
journalctl -u homestorage -n 50   # Last 50 lines

# Backup
/home/admin/simple-image-app/deploy/backup.sh

# Network
hostname -I           # Get IP address
```

### Windows
```cmd
# If using NSSM service
nssm start HomeStorage
nssm stop HomeStorage
nssm restart HomeStorage

# Backup
C:\Users\<Name>\simple-image-app\deploy\backup.bat

# Network
ipconfig            # Get IP address
```

---

## Backup Strategy

### Automated Backups (Linux)
```bash
crontab -e
# Add: 0 2 * * * /home/admin/simple-image-app/deploy/backup.sh
```

### Automated Backups (Windows)
- Use Task Scheduler
- Trigger: Daily at 2:00 AM
- Action: `deploy\backup.bat`

### Manual Backup
```bash
# Linux
/home/admin/simple-image-app/deploy/backup.sh

# Windows
deploy\backup.bat
```

### Backup Retention
- Last 10 backups automatically retained
- Older backups automatically deleted

---

## HTTPS Configuration

### Self-Signed Certificate (10 years, maintenance-free)
```bash
# Linux
cd /home/admin/simple-image-app/ssl
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem \
  -days 3650 -nodes -subj "/C=CN/ST=Home/L=Home/O=HomeStorage/CN=<ip>"

# Windows
deploy\generate-cert.bat
```

### Enable HTTPS
Edit `backend/app.py`, line ~378:
```python
USE_HTTPS = True
```

Restart service after changing.

---

## Support Resources

### Log Files
- **Linux**: `journalctl -u homestorage`
- **Windows**: Event Viewer → Applications

### Database Location
- **Linux**: `/home/admin/simple-image-app/backend/data.db`
- **Windows**: `C:\Users\<Name>\simple-image-app\backend\data.db`

### Query Database
```bash
python3 << EOF
import sqlite3
conn = sqlite3.connect('backend/data.db')
conn.row_factory = sqlite3.Row
cursor = conn.execute('SELECT * FROM items ORDER BY id DESC LIMIT 10')
for row in cursor:
    print(dict(row))
conn.close()
EOF
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-30 | Initial release |
| 1.1 | 2026-03-31 | Client-server architecture |
| 1.2 | 2026-03-31 | Image compression (<50KB) |
| 1.3 | 2026-03-31 | Password in database |
| 1.4 | 2026-03-31 | HTTPS support |
| 1.5 | 2026-03-31 | Zoom/Download functions |
| 1.6 | 2026-03-31 | Complete deployment package |

---

## License

This is a private project for home use. No license restrictions.
