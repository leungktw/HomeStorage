# Home Storage Inventory App - Complete Specification Document

**Version:** 1.6
**Date:** March 2026
**Purpose:** Complete specification for recreating the Home Storage Inventory Management System

---

## 1. Project Overview

Build a web-based home storage inventory management system with:
- Client-server architecture (Flask backend + web frontend)
- SQLite database for permanent storage
- Image upload with automatic compression
- Responsive design for desktop, tablet, and mobile
- Password authentication
- Optional HTTPS support

**Target Deployment:** Raspberry Pi (production), Ubuntu, Windows (development)

---

## 2. Technical Stack

### Backend
- **Framework:** Flask 3.0.0
- **Database:** SQLite 3.x (single file, no server)
- **Image Processing:** Pillow 10.x
- **CORS:** Flask-CORS 4.0.0
- **Python:** 3.8+

### Frontend
- **HTML5:** Single page application
- **CSS3:** Flexbox, Grid, CSS variables, animations
- **JavaScript:** Vanilla ES6+ (no framework required)
- **Responsive:** 3 breakpoints (desktop, 600px, 360px)

### Server Configuration
- **Port:** 5000
- **HTTPS:** Optional (self-signed or CA certificates)
- **Auto-start:** Systemd service (Linux) or Windows Service

---

## 3. Core Features

### 3.1 Authentication
- Password-protected login screen
- Default password: `1234`
- Password stored in database (table: `settings`)
- Password change functionality via modal dialog
- Logout button in header
- Form fields empty after login (no auto-restore)

### 3.2 Main Screen - Form Fields

| Field | Type | Required | Max Length | Notes |
|-------|------|----------|------------|-------|
| Category | Text input | Yes | 100 | Wildcard search |
| Item Description | Text input | Yes | 200 | Wildcard search |
| Zone | Text input | No | 50 | Wildcard search |
| Location | Text input | No | 100 | Wildcard search |
| Date In | Date picker | No | - | ISO format (YYYY-MM-DD) |
| Image | File upload | No | - | Auto-compress to <50KB |

### 3.3 Main Screen - Action Buttons

| Button | Icon | Function | Confirmation |
|--------|------|----------|--------------|
| 加 (Add) | ➕ | Create new record | Yes - modal dialog |
| 搜 (Search) | 🔍 | Open search results | No |
| 改 (Edit) | ✏️ | Update selected record | Yes - modal dialog |
| 删 (Delete) | 🗑️ | Delete selected record | Yes - modal dialog |
| 清 (Clear) | 🔄 | Clear all form fields | No |

### 3.4 Item List Screen (Search Results)

**Table Columns:**
1. Category
2. Item
3. Zone
4. Location
5. Date

**Result Count Badge:** Shows number of matching records

**Action Buttons:**
| Button | Icon | Function |
|--------|------|----------|
| 選 (Select) | ✓ | Copy selected record to Main Screen |
| 回 (Return) | ↩ | Return to Main Screen without changes |

### 3.5 Image Features

**Upload:**
- Camera icon button to trigger file selection
- Accept all image formats (JPEG, PNG, etc.)
- Preview thumbnail displayed after selection

**Compression (Backend):**
- Automatically compress to <50KB before saving
- Use quality reduction (start at 85, reduce by 5)
- Resize if needed (max 800x600, then reduce further)
- Convert RGBA/P to RGB for JPEG compatibility
- Return compressed base64 string

**Display:**
- Show image preview in form when record loaded
- Zoom button (🔍) - fullscreen modal view
- Download button (⬇️) - save image to local device

### 3.6 Search Functionality

**Wildcard Matching:**
- All text fields use LIKE '%term%' matching
- Space-separated terms use AND logic
- Example: "a b" matches records containing both "a" AND "b"

**Date Filter:**
- Date field uses >= comparison (items on or after selected date)
- Only applied if date is provided

**Search Endpoint:**
```
GET /api/items/search?category=&item=&zone=&location=&date=
```

### 3.7 Confirmation Dialogs

Show modal confirmation for:
- Add: "Add '[item name]' to the database?"
- Edit: "Update '[item name]' with the new information?"
- Delete: "Delete '[item name]' from the database?"

Each dialog has:
- Title with icon
- Message
- OK button (proceeds with action)
- Cancel button (closes dialog)

---

## 4. Database Schema

### Table: items

```sql
CREATE TABLE items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    category TEXT NOT NULL,
    item TEXT NOT NULL,
    zone TEXT,
    location TEXT,
    date TEXT,
    image TEXT,
    imageBase64 TEXT
);

CREATE INDEX idx_category ON items(category);
CREATE INDEX idx_item ON items(item);
```

### Table: settings

```sql
CREATE TABLE settings (
    key TEXT PRIMARY KEY,
    value TEXT
);

-- Insert default password
INSERT OR IGNORE INTO settings (key, value) VALUES ('password', '1234');
```

### Field Specifications

| Field | Type | Description |
|-------|------|-------------|
| id | INTEGER | Auto-increment primary key |
| category | TEXT | Item category (required) |
| item | TEXT | Item description (required) |
| zone | TEXT | Storage zone |
| location | TEXT | Specific location |
| date | TEXT | Date in (YYYY-MM-DD format) |
| image | TEXT | Image filename |
| imageBase64 | TEXT | Base64-encoded image data |

---

## 5. API Endpoints

### Health Check
```
GET /api/health
Response: {"status": "healthy", "database": "connected"}
```

### Get All Items
```
GET /api/items
Response: Array of item objects
```

### Search Items
```
GET /api/items/search?category=&item=&zone=&location=&date=
Response: Array of matching item objects
Query params: All optional, date uses >= comparison
```

### Add Item
```
POST /api/items
Body: {"category": "...", "item": "...", "zone": "...", "location": "...", "date": "...", "image": "...", "imageBase64": "..."}
Response: {"success": true, "id": <new_id>}
Validation: category and item required
```

### Update Item
```
PUT /api/items/<id>
Body: Same as Add Item
Response: {"success": true}
404 if item not found
```

### Delete Item
```
DELETE /api/items/<id>
Response: {"success": true}
404 if item not found
```

### Batch Delete
```
POST /api/items/batch-delete
Body: {"ids": [1, 2, 3]}
Response: {"success": true, "deleted": <count>}
```

### Get Statistics
```
GET /api/stats
Response: {"total_items": <count>, "total_categories": <count>}
```

### Get Password
```
GET /api/password
Response: {"password": "..."}
```

### Update Password
```
PUT /api/password
Body: {"password": "..."}
Response: {"success": true}
```

### Compress Image
```
POST /api/compress-image
Body: {"imageBase64": "data:image/jpeg;base64,..."}
Response: {"imageBase64": "data:image/jpeg;base64,..."}
Returns compressed image under 50KB
```

---

## 6. UI/UX Requirements

### Design System

**Color Scheme:**
- Primary: #4f46e5 (indigo)
- Primary Dark: #4338ca
- Primary Light: #818cf8
- Success: #10b981
- Warning: #f59e0b
- Danger: #ef4444
- Gray scale: 50-900

**Visual Style:**
- Gradient backgrounds (indigo to purple)
- Rounded corners (12-16px)
- Shadow effects (subtle to prominent)
- Smooth transitions (0.2s)
- Icons on all buttons

**Typography:**
- Font: System fonts (-apple-system, BlinkMacSystemFont, etc.)
- Base size: 14-16px (desktop), scaled down for mobile
- Headings: Bold, larger

### Layout

**Login Screen:**
- Full-screen overlay
- Centered login box
- Logo icon
- Password input with eye icon (optional)
- Login button
- Error message area

**Main Screen:**
- Header with title, server status, password change, logout
- Form with labeled fields (2-column layout on desktop)
- Image upload section with preview
- Action button row
- Smooth transitions between screens

**Item List Screen:**
- Header with title and result count
- Scrollable table container
- Fixed header row
- Striped rows for readability
- Selected row highlighting
- Action button row

**Mobile Optimizations:**
- Single column layout below 600px
- Reduced font sizes (5% reduction)
- Touch-friendly button sizes
- No horizontal scrolling
- Text truncation with ellipsis

### Interactive Elements

**Loading Indicator:**
- Full-screen overlay with spinner
- Shown during API calls
- Prevents interaction during loading

**Toast Notifications:**
- Success messages (green)
- Error messages (red)
- Auto-dismiss after 3 seconds
- Positioned at bottom-right

**Confirmation Modal:**
- Centered dialog
- Title with icon
- Message
- OK and Cancel buttons
- Different icon colors for different actions

---

## 7. Responsive Breakpoints

### Desktop (default)
- Two-column form layout
- Full-size fonts (14-16px)
- Table shows all columns

### Tablet (max-width: 600px)
- Single-column form layout
- Fonts reduced by 10%
- Table scrollable horizontally

### Mobile (max-width: 360px)
- Compact single-column layout
- Fonts reduced by additional 5%
- Button text shortened if needed
- Image preview max-height reduced
- Table columns: Category, Item (others hidden or abbreviated)

---

## 8. File Structure

```
simple-image-app/
├── backend/
│   ├── app.py                          # Flask application
│   └── requirements.txt                # Dependencies
├── frontend/
│   └── index.html                      # Single-file frontend
├── deploy/
│   ├── systemd/
│   │   └── homestorage.service         # Linux auto-start
│   ├── install.sh                      # Linux installer
│   ├── install.bat                     # Windows installer
│   ├── backup.sh                       # Linux backup
│   ├── backup.bat                      # Windows backup
│   ├── generate-cert.sh                # Linux SSL
│   └── generate-cert.bat               # Windows SSL
├── DEPLOY.md                           # Deployment guide
├── README.md                           # User docs
└── requirements.md                     # This file
```

---

## 9. Deployment Requirements

### Raspberry Pi (Production)

**System Requirements:**
- Raspberry Pi OS (32/64-bit)
- Python 3.8+
- 512MB RAM minimum
- 4GB storage minimum

**Installation Steps:**
1. Copy files to /home/<user>/simple-image-app
2. Install system packages: python3-pillow, libjpeg-dev, zlib1g-dev
3. Install Python packages: flask, flask-cors
4. Initialize database (runs on first start)
5. Configure systemd service
6. Enable and start service
7. Configure firewall (optional)
8. Generate SSL certificate (optional)

**Service Configuration:**
- Run as non-root user
- Auto-start on boot
- Restart on failure
- Log to journalctl

### Ubuntu (Development/Production)

Same as Raspberry Pi, but may use different package names.

### Windows (Development)

**Installation:**
1. Install Python 3.8+ from python.org
2. Copy files to C:\Users\<user>\simple-image-app
3. Install dependencies: pip install -r requirements.txt
4. Run: python backend/app.py
5. Optional: Install as Windows Service using NSSM

---

## 10. Security Requirements

### Authentication
- Password stored in database (table: settings)
- Default password: 1234
- Password change via API endpoint
- No password recovery (local deployment only)

### HTTPS Support
- Toggle via configuration: `USE_HTTPS = True/False`
- Self-signed certificate generation (10-year validity)
- Support for CA-signed certificates
- Certificate files stored in `ssl/` directory

### Network Security
- Local network only (no internet exposure by default)
- Firewall configuration optional
- CORS enabled for local network access

---

## 11. Image Compression Algorithm

**Goal:** Compress all images to under 50KB

**Algorithm:**
1. Decode base64 to bytes
2. Open with PIL/Pillow
3. Convert RGBA/P to RGB
4. Set initial quality: 85
5. Set max dimensions: 800x600
6. Resize if larger (preserve aspect ratio)
7. Loop:
   - Compress with current quality
   - If size <= 50KB: exit loop
   - If quality > 20: reduce quality by 5
   - Else: reduce dimensions by 20%, reset quality to 50
8. Re-encode to base64
9. Return compressed image

**Error Handling:**
- If compression fails, return original image
- Log compression ratio for debugging

---

## 12. Testing Checklist

### Functional Tests
- [ ] Login with correct password
- [ ] Login with wrong password (error shown)
- [ ] Add new record with all fields
- [ ] Add new record with required fields only
- [ ] Add record with image (verify compression <50KB)
- [ ] Search with single term
- [ ] Search with multiple terms (AND logic)
- [ ] Search with date filter
- [ ] Select record from search results
- [ ] Edit record and save changes
- [ ] Delete record (with confirmation)
- [ ] Clear form
- [ ] Change password
- [ ] Logout
- [ ] Image zoom (fullscreen)
- [ ] Image download

### Responsive Tests
- [ ] Desktop view (1920x1080)
- [ ] Tablet view (768x1024)
- [ ] Mobile view (360x640)
- [ ] Mobile view (320x568)
- [ ] No horizontal scroll at any breakpoint
- [ ] All buttons touch-friendly on mobile

### API Tests
- [ ] GET /api/health
- [ ] GET /api/items
- [ ] GET /api/items/search with various params
- [ ] POST /api/items
- [ ] PUT /api/items/<id>
- [ ] DELETE /api/items/<id>
- [ ] POST /api/compress-image
- [ ] GET /api/password
- [ ] PUT /api/password

### Database Tests
- [ ] Database created on first run
- [ ] Records persist after restart
- [ ] Images stored correctly
- [ ] Password change persists
- [ ] VACUUM reduces file size after deletes

---

## 13. Known Issues & Solutions

### Python 3.13 + Pillow Build Failure
**Issue:** Pillow fails to build from source on Python 3.13

**Solution:** Install Pillow from system packages:
```bash
sudo apt install python3-pillow
```

### PEP 668 Externally Managed Environment
**Issue:** pip refuses to install packages system-wide

**Solution:** Use `--break-system-packages` flag:
```bash
pip3 install --break-system-packages <package>
```

### Service Paths Hardcoded
**Issue:** Service file has /home/pi but user is different

**Solution:** Update paths with sed:
```bash
sed -i 's|/home/pi|/home/<actual-user>|g' /etc/systemd/system/homestorage.service
sed -i 's/User=pi/User=<actual-user>/g' /etc/systemd/system/homestorage.service
```

### SQLite File Size After Delete
**Issue:** Database file stays large after deleting records

**Solution:** Run VACUUM:
```python
import sqlite3
conn = sqlite3.connect('data.db')
conn.execute('VACUUM')
conn.close()
```

---

## 14. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-30 | Initial release with basic CRUD |
| 1.1 | 2026-03-30 | Added image storage per record |
| 1.2 | 2026-03-30 | Added login system |
| 1.3 | 2026-03-30 | Professional UI redesign |
| 1.4 | 2026-03-31 | Responsive design improvements |
| 1.5 | 2026-03-31 | Client-server architecture (Flask + SQLite) |
| 1.6 | 2026-03-31 | Image compression, HTTPS, Zoom/Download |

---

## 15. Quick Recreation Instructions

To recreate this application with another AI assistant:

1. **Provide this document** as the complete specification
2. **Start with backend:** Create Flask app with all API endpoints
3. **Then frontend:** Create single HTML file with all UI and API integration
4. **Then deployment:** Create install scripts for target platform
5. **Test incrementally:** Verify each feature as it's built

**Expected output:**
- backend/app.py (~400 lines)
- backend/requirements.txt (3 packages)
- frontend/index.html (~2000 lines)
- deploy/* (7 scripts)
- Documentation files

**Estimated build time:** 30-60 minutes with AI assistance

---

## 16. Success Criteria

The application is complete when:
- [ ] All API endpoints functional
- [ ] All UI features working
- [ ] Image compression achieves <50KB
- [ ] Responsive design works at all breakpoints
- [ ] Authentication works (login, logout, password change)
- [ ] HTTPS can be enabled
- [ ] Service auto-starts on boot
- [ ] Backups work automatically
- [ ] No console errors in browser
- [ ] No errors in server logs

---

**End of Specification Document**
