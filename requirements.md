# Home Storage App - Requirements Document

## Initial Request
**Date:** 2026-03-30

Create a simple web application for home storage inventory management.

---

## Core Features Requested

### 1. Main Screen (CRUD1.pptx - Slide 1)
- Form fields:
  - Category
  - Item Description
  - Zone
  - Location
  - Date In
  - Image (with upload capability)
- Five action buttons:
  - 加 (Add) - Add new record
  - 搜 (Search) - Search records
  - 改 (Edit) - Edit existing record
  - 删 (Delete) - Delete record
  - 清 (Clear) - Clear form

### 2. Item List Screen (CRUD1.pptx - Slide 2)
- Data table with columns: Category, Item, Zone, Location, Date
- Two buttons:
  - 選 (Select) - Copy selected record back to Main Screen
  - 回 (Return) - Return to Main Screen without changes

### 3. Search Functionality
- Wildcard search using values from form fields
- Space-separated terms treated as AND logic
  - Example: "a b" finds records containing both "a" AND "b"
- All matching records displayed in Item List screen

### 4. Data Persistence
- Store data in browser's localStorage
- Form auto-saves as user types
- Data persists across browser sessions

### 5. Image Handling
- Upload images via camera icon
- Images stored with each record (base64 encoding)
- Images displayed when record is loaded

---

## Additional Requirements (Added Later)

### 6. Login System
- Password-protected access
- Default password: "1234"
- Password change functionality
- Login screen with modern design
- Logout button in app header

### 7. UI/UX Improvements
- Professional modern design with:
  - Gradient color scheme (indigo/purple)
  - Smooth animations and transitions
  - Shadow effects and rounded corners
  - Icons on buttons
  - Result count badge on Item List
- Responsive design for mobile devices:
  - Media queries for 600px and 360px breakpoints
  - Touch-friendly buttons
  - Scrollable table on small screens
  - No horizontal overflow

### 8. Post-Login Behavior
- Form fields should be empty after login (not restored from storage)

### 9. Mobile Optimization
- Input fields should not overflow/wrap to extra lines
- App should scale down to fit screen rather than overflow
- Proper text truncation with ellipsis for long content
- Adjusted column widths for mobile displays

---

## Technical Constraints

- Client-server architecture with Flask backend and SQLite database
- Server runs on Raspberry Pi (production) or local machine (development)
- REST API with JSON for client-server communication
- Images automatically compressed to <50KB before storage
- HTTPS support for secure data transmission
- Password stored in database (persistent across sessions/devices)

---

## Files in Project

| File | Description |
|------|-------------|
| `backend/app.py` | Flask REST API server with SQLite database |
| `backend/requirements.txt` | Python dependencies (Flask, Flask-CORS, Pillow) |
| `backend/data.db` | SQLite database for permanent storage |
| `frontend/index.html` | Web frontend with responsive design |
| `deploy/systemd/homestorage.service` | Systemd service for auto-start on Raspberry Pi |
| `deploy/backup.sh` | Database backup script |
| `deploy/generate-cert.bat` | SSL certificate generation script |
| `INSTALL.md` | Installation and deployment instructions |
| `README.md` | User documentation |
| `requirements.md` | This file - requirements documentation |

---

## Design References

- `CRUD1.pptx` - Original UI design slides
  - Slide 1: Main Screen layout
  - Slide 2: Listing screen layout

---

## Revision History

| Date | Change |
|------|--------|
| 2026-03-30 | Initial app creation with basic CRUD + search |
| 2026-03-30 | Fixed image storage per record |
| 2026-03-30 | Added login system with password change |
| 2026-03-30 | Professional UI redesign |
| 2026-03-31 | Fixed post-login form state |
| 2026-03-31 | Mobile responsive improvements |
| 2026-03-31 | Client-server architecture with Flask + SQLite |
| 2026-03-31 | Image compression to <50KB before storage |
| 2026-03-31 | Password stored in database instead of localStorage |
| 2026-03-31 | HTTPS support added |
