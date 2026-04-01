# Home Storage App - Raspberry Pi Installation Guide

Complete guide for deploying the Home Storage Inventory Management System to Raspberry Pi.

## Quick Start (Automated Installation)

For a fully automated installation, run this single command after copying files to your Raspberry Pi:

```bash
cd /home/pi/simple-image-app
bash deploy/install.sh
```

The automated installer will:
- Update system packages
- Install all dependencies
- Create directory structure
- Setup systemd service
- Configure firewall (optional)
- Display access information

---

## Manual Installation Steps

### Prerequisites

- Raspberry Pi (any model, Pi 3 or later recommended)
- Raspberry Pi OS (32-bit or 64-bit)
- Network connection
- SD card with at least 4GB storage

### 1. Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Python and Dependencies

Python 3 comes pre-installed on Raspberry Pi OS. Install required packages:

```bash
sudo apt install -y python3-pip python3-venv git
```

### 3. Clone or Copy Application

**Option A: Clone from Git (if you have the code in a repository)**
```bash
cd /home/pi
git clone <your-repository-url> simple-image-app
```

**Option B: Manual copy (SCP from your development machine)**
```bash
# From your Windows/Mac/Linux machine
scp -r simple-image-app pi@<raspberry-pi-ip>:/home/pi/
```

### 4. Install Python Dependencies

```bash
cd /home/pi/simple-image-app/backend
pip3 install -r requirements.txt
```

### 5. Initialize Database

Run the application once to create the database:

```bash
python3 app.py
```

Press `Ctrl+C` to stop it after you see the database initialization message.

### 6. Set Up Systemd Service (Auto-start on Boot)

Copy the service file:

```bash
sudo cp /home/pi/simple-image-app/deploy/systemd/homestorage.service /etc/systemd/system/
```

Edit the service file if your username is not `pi`:

```bash
sudo nano /etc/systemd/system/homestorage.service
```

Reload systemd and enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable homestorage
sudo systemctl start homestorage
```

Check service status:

```bash
sudo systemctl status homestorage
```

### 7. Configure Firewall (Optional)

If you have a firewall enabled, allow port 5000:

```bash
sudo apt install -y ufw
sudo ufw allow 5000/tcp
sudo ufw enable
```

### 8. Access the Application

Find your Raspberry Pi's IP address:

```bash
hostname -I
```

Access the application from any device on the same network:

```
http://<raspberry-pi-ip>:5000
```

Default login password: `1234`

## Maintenance

### View Application Logs

```bash
# Real-time logs
journalctl -u homestorage -f

# Today's logs
journalctl -u homestorage --since today

# Last 50 lines
journalctl -u homestorage -n 50
```

### Restart the Service

```bash
sudo systemctl restart homestorage
```

### Stop the Service

```bash
sudo systemctl stop homestorage
```

### Disable Auto-start

```bash
sudo systemctl disable homestorage
```

### Backup Database

Run the backup script:

```bash
cd /home/pi/simple-image-app
chmod +x deploy/backup.sh
./deploy/backup.sh
```

Backups are stored in `/home/pi/simple-image-app/backups/` with timestamps. The script automatically keeps only the last 10 backups.

**Automated Daily Backup (Optional):**

Add to crontab for daily 2 AM backups:

```bash
crontab -e
```

Add this line:

```
0 2 * * * /home/pi/simple-image-app/deploy/backup.sh
```

### Restore from Backup

```bash
# Stop the service
sudo systemctl stop homestorage

# Copy backup to database location
cp /home/pi/simple-image-app/backups/data_backup_YYYYMMDD_HHMMSS.db /home/pi/simple-image-app/backend/data.db

# Restart service
sudo systemctl start homestorage
```

## Troubleshooting

### Service Won't Start

Check logs for errors:

```bash
journalctl -u homestorage -n 50 --no-pager
```

Common issues:
- Missing dependencies: `pip3 install -r /home/pi/simple-image-app/backend/requirements.txt`
- Database file permissions: `sudo chown pi:pi /home/pi/simple-image-app/backend/data.db`
- Port already in use: Change port in `app.py` line 269

### Can't Access from Network

1. Verify service is running: `sudo systemctl status homestorage`
2. Check IP address: `hostname -I`
3. Test locally: `curl http://localhost:5000/api/health`
4. Check firewall: `sudo ufw status`

### Database Issues

Reset database (WARNING: deletes all data):

```bash
rm /home/pi/simple-image-app/backend/data.db
sudo systemctl restart homestorage
```

## Production Considerations

### Change Default Password

After first login, immediately change the password from the default `1234` using the password change feature in the app.

### Static IP Address (Recommended)

Edit DHCP configuration:

```bash
sudo nano /etc/dhcpcd.conf
```

Add at the bottom (adjust for your network):

```
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

Reboot: `sudo reboot`

### HTTPS Setup (Optional but Recommended)

For production use with sensitive data, enable HTTPS:

**Option 1: Self-signed certificate (quick setup)**

1. Install OpenSSL (if not already installed):
   ```bash
   sudo apt install -y openssl
   ```

2. Generate a self-signed certificate:
   ```bash
   cd /home/pi/simple-image-app/backend
   openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes \
     -subj "/C=CN/ST=Local/L=Local/O=HomeStorage/CN=localhost"
   ```

3. Edit `app.py` and set `USE_HTTPS = True` (line 378):
   ```python
   USE_HTTPS = True  # Set to True to enable HTTPS
   ```

4. Restart the service:
   ```bash
   sudo systemctl restart homestorage
   ```

5. Access via: `https://<raspberry-pi-ip>:5000`

   Note: Browsers will show a security warning for self-signed certificates. Add an exception to proceed.

**Option 2: Let's Encrypt certificate (recommended for production)**

Set up nginx as a reverse proxy with Let's Encrypt certificates:

```bash
sudo apt install -y nginx certbot python3-certbot-nginx
```

Then configure nginx to proxy requests to the Flask app and obtain SSL certificates from Let's Encrypt.

### Remote Access from Internet (Optional)

If you need to access the app from outside your home network:

1. **Port forwarding**: Forward port 5000 (or 443 for HTTPS) in your router
2. **Dynamic DNS**: Use a service like No-IP or DuckDNS for dynamic IP addresses
3. **Security**: Ensure strong password and consider HTTPS

**Warning**: Exposing the app to the internet without proper security measures is not recommended as the current authentication is basic client-side only.

## File Structure

```
/home/pi/simple-image-app/
├── backend/
│   ├── app.py              # Flask application
│   ├── requirements.txt    # Python dependencies
│   └── data.db             # SQLite database (created automatically)
├── frontend/
│   └── index.html          # Web interface
├── deploy/
│   ├── systemd/
│   │   └── homestorage.service  # Systemd service file
│   └── backup.sh           # Database backup script
└── INSTALL.md              # This file
```

## Support

For issues or questions, check the application logs first using `journalctl -u homestorage`.
