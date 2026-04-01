# Home Storage App

A web-based home storage inventory management system with client-server architecture.

---

## Quick Start

### Login
- Default password: **1234**
- Change password via the key icon (🔑) in the top-right corner

### Access the Application
- **Local**: http://localhost:5000
- **Network**: http://<server-ip>:5000
- **HTTPS**: https://<server-ip>:5000 (if enabled)

---

## Features

### Main Screen
- **Form Fields:**
  - Category
  - Item Description
  - Zone
  - Location
  - Date In
  - Image (with camera icon to upload)

- **Buttons:**
  - **加 (Add)** - Add a new record
  - **搜 (Search)** - Search records matching form criteria
  - **改 (Edit)** - Edit existing record
  - **删 (Delete)** - Delete a record
  - **清 (Clear)** - Clear all form fields

### Image Features
- **Upload**: Click camera icon to upload images
- **Auto-compression**: Images automatically compressed to <50KB before saving
- **Zoom**: Click 🔍 Zoom button to view image fullscreen
- **Download**: Click ⬇️ Download button to save image to your device
- **Preview**: Images displayed in form when record is loaded

### Item List Screen (Search Results)
- Displays matching records in a table with columns: Category, Item, Zone, Location, Date
- Result count badge shows number of matching records
- **選 (Select)** - Copy selected record back to Main Screen
- **回 (Return)** - Return to Main Screen without changes

---

## Search Functionality

The search uses **wildcard matching** with space-separated terms:
- Enter search terms in any combination of fields
- Spaces separate multiple terms (AND logic)
- Example: Searching "a b" in Item field finds records containing both "a" AND "b"
- **Date filter**: Shows items with date >= selected date (if Date In field is filled)

### Search Fields
| Field | Match Type |
|-------|------------|
| Category | Wildcard (contains) |
| Item | Wildcard (contains) |
| Zone | Wildcard (contains) |
| Location | Wildcard (contains) |
| Date In | Greater than or equal (>=) |

---

## Data Persistence

- **Database**: SQLite (single file, no server required)
- **Server**: Flask REST API
- **Storage**: All data stored in `backend/data.db`
- **Images**: Stored as base64, compressed to <50KB

---

## Security

### Authentication
- Password-protected access
- Password stored in database (persists across sessions/devices)
- Default password: **1234** (change after first login)

### HTTPS (Optional)
- Self-signed certificate support (10-year validity)
- CA certificate support (Let's Encrypt via nginx)
- Toggle in `backend/app.py`: `USE_HTTPS = True`

---

## Usage

### Add a Record
1. Login with password
2. Fill in form fields (Category and Item Description required)
3. Click camera icon to upload an image (optional)
4. Click **加 (Add)** to save
5. Confirm the action in the popup

### Search Records
1. Click **搜 (Search)**
2. Enter search criteria in any fields
3. Click Search button
4. View matching records in Item List
5. Click a row to select, then click **選 (Select)** to load it
6. Click **回 (Return)** to go back without changes

### Edit a Record
1. Search and select the record to edit
2. Record loads into the form
3. Make changes
4. Click **改 (Edit)** to save changes
5. Confirm the action in the popup

### Delete a Record
1. Search and select the record to delete
2. Click **删 (Delete)**
3. Confirm the action in the popup

### Image Operations
- **Upload**: Click camera icon, select image file
- **Zoom**: Click 🔍 Zoom button for fullscreen view (click anywhere to close)
- **Download**: Click ⬇️ Download button to save image

### Change Password
1. Click the key icon (🔑) in the top-right corner
2. Enter current password
3. Enter new password twice
4. Click "Change Password" to save

### Logout
- Click the door icon (🚪) in the top-right corner

---

## System Requirements

### Server Requirements
- **Python**: 3.8+
- **OS**: Raspberry Pi OS, Ubuntu, Windows 10/11
- **RAM**: 512 MB minimum (2 GB recommended)
- **Storage**: 100 MB + database space

### Client Requirements
- **Browser**: Any modern web browser (Chrome, Firefox, Safari, Edge)
- **Network**: Access to server on port 5000

---

## Architecture

```
┌─────────────┐     HTTP/HTTPS     ┌─────────────┐
│   Browser   │ ◄────────────────► │ Flask Server│
│  (Frontend) │                    │  (Backend)  │
│  index.html │                    │   app.py    │
└─────────────┘                    └──────┬──────┘
                                          │
                                          ▼
                                   ┌─────────────┐
                                   │   SQLite    │
                                   │  database   │
                                   │  (data.db)  │
                                   └─────────────┘
```

### API Endpoints
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/health` | GET | Health check |
| `/api/items` | GET | Get all items |
| `/api/items/search` | GET | Search with filters |
| `/api/items` | POST | Add item |
| `/api/items/<id>` | PUT | Update item |
| `/api/items/<id>` | DELETE | Delete item |
| `/api/items/batch-delete` | POST | Batch delete |
| `/api/stats` | GET | Database statistics |
| `/api/password` | GET | Get password |
| `/api/password` | PUT | Update password |
| `/api/compress-image` | POST | Compress image |

---

## Deployment

See `DEPLOY.md` for complete deployment instructions for:
- Raspberry Pi
- Ubuntu/Linux
- Windows

### Quick Deploy (Raspberry Pi)
```bash
# Copy files
scp -r simple-image-app admin@<ip>:/home/admin/

# Run installer
cd /home/admin/simple-image-app
bash deploy/install.sh

# Access
# Open: http://<ip>:5000
```

---

## Maintenance

### Backup Database
```bash
# Linux
/home/admin/simple-image-app/deploy/backup.sh

# Windows
deploy\backup.bat
```

### View Logs
```bash
# Linux
journalctl -u homestorage -f

# Windows
# Event Viewer → Applications
```

### Service Management
```bash
# Linux
sudo systemctl start homestorage
sudo systemctl stop homestorage
sudo systemctl restart homestorage
sudo systemctl status homestorage
```

---

## Files

| File | Description |
|------|-------------|
| `backend/app.py` | Flask REST API server |
| `backend/requirements.txt` | Python dependencies |
| `backend/data.db` | SQLite database |
| `frontend/index.html` | Web interface |
| `deploy/` | Deployment scripts |
| `DEPLOY.md` | Deployment guide |

---

## Version

**1.6** - March 2026

### Recent Changes
- ✅ Image compression (<50KB guaranteed)
- ✅ Image zoom (fullscreen view)
- ✅ Image download
- ✅ Password stored in database
- ✅ HTTPS support (self-signed or CA)
- ✅ Client-server architecture
- ✅ Confirmation dialogs for CRUD operations

---

## License

Private project for home use.
