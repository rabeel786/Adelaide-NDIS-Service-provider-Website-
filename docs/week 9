# Week 9 — Feature Integration
## Adelaide Care Connect | CPRO306 Capstone Project
**Kent Institute Australia | Group 8 | 2026**

---

## Overview

| Property | Detail |
|---|---|
| **Week** | Week 9 |
| **Theme** | Feature Integration |
| **Deliverable** | Working MVP |
| **Lead** | Rabeel Riasat (Project Manager) |
| **Support** | All team members |

---

## Activities Completed This Week

### 1. Frontend and Backend Integration

All frontend pages have been connected to their corresponding backend models and database tables. Every form, table, and dynamic component in the system now reads from and writes to the live MySQL database. Static placeholder data has been fully replaced with live database queries.

#### 1.1 Integration Points Completed

| Frontend Component | Backend Connection | Status |
|---|---|---|
| Home page service cards | `SELECT * FROM services WHERE is_active = 1` | ✅ Connected |
| Home page news cards | `SELECT * FROM news_resources WHERE is_published = 1 LIMIT 3` | ✅ Connected |
| Participant dashboard KPIs | Booking model `getByParticipant()` + Document model `getByUser()` | ✅ Connected |
| Bookings table | Booking model `getByParticipant()` with status filter | ✅ Connected |
| Book a service modal — service dropdown | `SELECT * FROM services WHERE is_active = 1` | ✅ Connected |
| Book a service modal — staff dropdown | AJAX `get_staff.php` — live availability check | ✅ Connected |
| Cost estimator | JavaScript calculates from service `hourly_rate` in DB | ✅ Connected |
| Document grid | Document model `getByUser()` with decrypted file paths | ✅ Connected |
| AI chatbot | OpenRouter API via `ajax/chatbot.php` — `openrouter/auto` model | ✅ Connected |
| Staff shifts calendar | Shift model `getByStaff()` grouped by week | ✅ Connected |
| Shift detail page | Shift model `getDetail()` — participant address decrypted | ✅ Connected |
| Admin dashboard KPIs | Admin model `getDashboardStats()` — 6 live counts | ✅ Connected |
| Admin bookings table | Admin model `getAllBookings()` with status filters | ✅ Connected |
| Admin approve modal — staff dropdown | `SELECT * FROM staff JOIN users` — active staff only | ✅ Connected |
| Admin participants table | Admin model `getAllParticipants()` — paginated | ✅ Connected |
| Admin enquiries inbox | Admin model `getEnquiries()` ordered by date | ✅ Connected |

---

### 2. Authentication Integration

The complete authentication flow has been tested end-to-end across all three portals. Sessions are correctly maintained across page navigation and role checks are enforced on every protected page.

#### 2.1 Session Flow Verified

```
Guest visits /participant/dashboard.php
    → auth_check.php fires requireRole('participant')
    → No valid session found
    → Flash message: "Please log in to access your dashboard"
    → Redirect to /login.php
    → User logs in with valid credentials
    → session_regenerate_id(true) called
    → $_SESSION['user_id'], ['user_role'], ['user_name'] set
    → Redirect to /participant/dashboard.php
    → auth_check.php passes
    → Dashboard loads with live data
```

#### 2.2 Cross-Portal Access Prevention

Verified that each role cannot access another role's portal:

| Attempt | Result |
|---|---|
| Participant visits `admin/dashboard.php` | ✅ Redirected to login with error |
| Staff visits `participant/bookings.php` | ✅ Redirected to login with error |
| Admin visits `staff/shifts.php` | ✅ Redirected to login with error |
| Unauthenticated user visits any portal | ✅ Redirected to login with error |

---

### 3. Booking Workflow — Full End-to-End

The complete booking lifecycle has been integrated and tested from participant submission through to staff shift assignment.

```
PARTICIPANT                    DATABASE                    ADMIN / STAFF
    │                              │                             │
    │── Submit booking ───────────►│                             │
    │   (service, date, staff,     │ INSERT bookings             │
    │    start/end time, notes)    │ status = 'pending'          │
    │                              │                             │
    │◄─ Booking confirmed ─────────│                             │
    │   status: Pending            │                             │
    │                              │◄──── Admin views pending ───│
    │                              │      bookings dashboard     │
    │                              │                             │
    │                              │      Admin approves ───────►│
    │                              │ UPDATE bookings             │
    │                              │ status = 'approved'         │
    │                              │ INSERT shifts               │
    │                              │ (staff_id, booking_id,      │
    │                              │  date, time, status=        │
    │                              │  'assigned')                │
    │                              │                             │
    │◄─ Status updated ────────────│                             │
    │   Booking shows Approved     │                             │
    │                              │◄──── Staff views new shift ─│
    │                              │      in their calendar      │
    │                              │                             │
    │                              │      Staff accepts ────────►│
    │                              │ UPDATE shifts               │
    │                              │ status = 'accepted'         │
```

**Booking approval auto-creates shift — code verified:**
```php
// In Booking model — approve() method
public function approve(int $bookingId, int $staffId, string $notes = ''): bool {
    $db = getDB();

    // Update booking status
    $stmt = $db->prepare(
        'UPDATE bookings SET status = "approved", staff_id = ?, admin_notes = ?
         WHERE booking_id = ?'
    );
    $stmt->execute([$staffId, $notes, $bookingId]);

    // Fetch booking details for shift creation
    $booking = $this->getById($bookingId);

    // Auto-create shift record
    $shiftStmt = $db->prepare(
        'INSERT INTO shifts (staff_id, booking_id, shift_date, start_time, end_time, status)
         VALUES (?, ?, ?, ?, ?, "assigned")'
    );
    $shiftStmt->execute([
        $staffId,
        $bookingId,
        $booking['booking_date'],
        $booking['start_time'],
        $booking['end_time'],
    ]);

    return true;
}
```

---

### 4. AI Chatbot Integration

The OpenRouter AI chatbot has been fully integrated and is working end-to-end in the participant portal.

#### 4.1 Integration Issues Resolved

**Issue:** Initial implementation used model `mistralai/mistral-7b-instruct` which returned HTTP 404 — endpoint no longer available on OpenRouter.

**Resolution:** Model updated to `openrouter/auto` which dynamically selects the best available free model. Chatbot now responds correctly.

```php
// Fixed in participant/ajax/chatbot.php
define('OPENROUTER_MODEL', 'openrouter/auto');  // Changed from mistral-7b-instruct
```

**Issue:** CSRF token mismatch on session timeout caused chatbot form to fail silently.

**Resolution:** CSRF token regenerated on page load rather than session start. Form now handles session expiry gracefully.

#### 4.2 Verified Chatbot Behaviour

| Test | Expected | Result |
|---|---|---|
| Send "What is the NDIS?" | NDIS explanation response | ✅ Pass |
| Send empty message | Error: "Message cannot be empty" | ✅ Pass |
| Send 1001 character message | Error: "Message too long" | ✅ Pass |
| Access chatbot as staff user | HTTP 403 Unauthorised | ✅ Pass |
| Session history retained across turns | Context maintained in conversation | ✅ Pass |
| Log saved to `chatbot_logs` table | SHA-256 session token, no PII | ✅ Pass |
| New Chat clears session history | Fresh conversation starts | ✅ Pass |

#### 4.3 System Prompt

The chatbot uses a carefully designed system prompt restricting responses to NDIS-related topics:

```
You are a helpful NDIS support assistant for Adelaide Care Connect,
a registered NDIS service provider in Adelaide, South Australia.

Your role is to:
- Answer questions about the NDIS
- Help participants understand their NDIS plans and funding
- Explain Adelaide Care Connect services
- Guide participants on using the online portal

Important: Do NOT discuss topics unrelated to disability support or NDIS.
Keep responses under 200 words. Always be compassionate and respectful.
```

---

### 5. AES-256 Encryption Integration

Verified that AES-256 encryption is working correctly at all data entry and retrieval points.

#### 5.1 Encryption Points Verified

| Operation | Field | Encrypted Before Save | Decrypted On Read |
|---|---|---|---|
| Participant registration | ndis_number | ✅ Yes | ✅ Shift detail page only |
| Participant registration | dob | ✅ Yes | ✅ Profile page only |
| Participant registration | address | ✅ Yes | ✅ Shift detail + profile |
| Document upload | file_path | ✅ Yes | ✅ Download/serve only |

**Verified in phpMyAdmin:** The `ndis_number` column stores binary-encoded ciphertext, not the plaintext NDIS number. Decryption only occurs at the application layer when explicitly needed.

#### 5.2 phpMyAdmin Verification

Running this query in phpMyAdmin confirms encryption is active:
```sql
SELECT ndis_number, AES_DECRYPT(FROM_BASE64(ndis_number), 'AdelaideCC2026Key!') AS decrypted
FROM participants LIMIT 5;
```
The `ndis_number` column shows encrypted data. The `decrypted` column shows the plain NDIS number only when the correct key is provided.

---

### 6. Integration Bug Fixes

The following bugs were identified and resolved during integration testing this week:

| # | Bug | Root Cause | Fix Applied |
|---|---|---|---|
| 1 | OpenRouter chatbot returning HTTP 404 | Deprecated model ID `mistral-7b-instruct` | Changed model to `openrouter/auto` |
| 2 | Registration CSRF token mismatch on timeout | Token generated at session start, expires before submission | Moved token generation to page load |
| 3 | Hero slideshow step 3 image not loading | Local file path referenced for hosted image | Replaced with valid Unsplash CDN URL |
| 4 | Booking cost estimator showing NaN | Time input returning empty string before selection | Added null guard before calculation |
| 5 | Staff dropdown empty on booking form | `get_staff.php` date parameter format mismatch | Normalised date format to `Y-m-d` |
| 6 | Document download returning 404 | Decrypted path had trailing whitespace | Added `trim()` after `decryptData()` call |
| 7 | Admin dashboard KPI count off by one | COUNT query including soft-deleted records | Added `WHERE is_active = 1` filter |
| 8 | News category filter not resetting | JavaScript filter state not cleared on tab switch | Added `classList.remove('active')` reset |

---

### 7. Working MVP — Feature Completeness

The following table shows the MVP feature status at end of Week 9.

#### Core Features

| Feature | Implemented | Tested | Notes |
|---|---|---|---|
| Public website — 5 pages | ✅ | ✅ | All pages live with photo backgrounds |
| User registration — 3 steps | ✅ | ✅ | NDIS validation, AES-256 encryption |
| User login + RBAC | ✅ | ✅ | bcrypt verify, session, role redirect |
| Participant dashboard | ✅ | ✅ | Live KPIs from database |
| Service booking | ✅ | ✅ | Modal, AJAX staff loader, cost estimator |
| Booking cancellation (24hr rule) | ✅ | 🔄 | Core working, edge cases in testing |
| Document upload + encryption | ✅ | ✅ | MIME check, 5MB limit, AES-256 path |
| Secure document download | ✅ | ✅ | PHP streams file, no direct URL access |
| AI chatbot (OpenRouter) | ✅ | ✅ | Responding correctly with openrouter/auto |
| Staff shifts calendar | ✅ | ✅ | Weekly view with colour-coded shift blocks |
| Staff shift accept / decline | ✅ | ✅ | Status updates, decline reason recorded |
| Admin booking approval | ✅ | ✅ | Auto-creates shift on approval |
| Admin booking rejection | ✅ | ✅ | Reason stored in admin_notes |
| Admin participant management | ✅ | ✅ | Full listing with search |
| Admin staff management | ✅ | ✅ | Add staff creates user account |
| Admin service CRUD | ✅ | 🔄 | Edit and toggle working, in testing |
| Admin news management | ✅ | 🔄 | Publish/unpublish working, in testing |
| Admin enquiries inbox | ✅ | ✅ | Mark read, reply via email link |
| Contact enquiry form | ✅ | ✅ | Saves to DB, success message |
| News article listing + single view | ✅ | ✅ | 8 articles with images, full content |

#### Security Features

| Feature | Implemented | Verified |
|---|---|---|
| bcrypt password hashing (cost=12) | ✅ | ✅ |
| AES-256-CBC field encryption | ✅ | ✅ |
| CSRF token on all POST forms | ✅ | ✅ |
| PDO prepared statements (SQL injection) | ✅ | ✅ |
| XSS prevention (`htmlspecialchars`) | ✅ | ✅ |
| Session regeneration on login | ✅ | ✅ |
| Role-based access control | ✅ | ✅ |
| File upload MIME validation | ✅ | ✅ |
| Uploads stored outside web root | ✅ | ✅ |
| Chatbot logs anonymised (SHA-256) | ✅ | ✅ |

---

### 8. Assessment 4 Submission

Assessment 4 — Mid Project Deliverables (15%) submitted this week.

**Submission contents:**
```
Assessment4_Submission.zip
├── Assessment4_Report.docx     ← 2,776 words, 6 criteria covered
└── adelaide-care-connect/      ← Full project folder
    └── database/schema.sql     ← All 10 tables + 8 news articles + demo accounts
```

**Report sections:**
| Section | Content |
|---|---|
| 1. Introduction | Project overview, 22 pages, 3 portals |
| 2. System Requirements | Agile sprints, 10 BRs, testing progress TC-01 to TC-10 |
| 3. System Architecture | 3-tier, folder structure, tech stack table |
| 4. Database Design | ERD, 10 tables, AES-256 code snippet |
| 5. User Interface | All portals, all pages, AI chatbot section |
| 6. Teamwork | GitHub evidence, individual contributions, lecturer feedback |
| 7. Conclusion | Mid-project status, Sprint 5 remaining work |
| 8. References | 7 Harvard-format references |

---

## Questions: What MVP Features Are Now Fully Working?

### ✅ All MVP Features Working

Every feature defined in the original SRS report is either fully working or in final testing:

**Participant can:**
- ✅ Register with NDIS number validation
- ✅ Log in and access their personal dashboard
- ✅ Book any of the 6 NDIS services with staff preference
- ✅ See real-time cost estimate before submitting
- ✅ View booking history with status tracking
- ✅ Cancel bookings (24-hour rule enforced)
- ✅ Upload and download encrypted documents
- ✅ Chat with AI NDIS assistant
- ✅ Update profile and change password

**Staff can:**
- ✅ View assigned shifts in weekly calendar
- ✅ Accept shifts with one click
- ✅ Decline shifts with a reason
- ✅ View participant address and emergency contact on shift day
- ✅ Update profile and availability notes

**Admin can:**
- ✅ See live system overview on dashboard
- ✅ View, search, and manage all participants
- ✅ Add new staff members
- ✅ Approve and reject bookings with staff assignment
- ✅ Manage the NDIS service catalogue
- ✅ View all uploaded documents
- ✅ Publish news articles and resources
- ✅ Read and manage public enquiries

**System enforces:**
- ✅ AES-256 encryption on sensitive fields
- ✅ bcrypt password hashing
- ✅ CSRF protection on every form
- ✅ SQL injection prevention via PDO
- ✅ File type and size validation
- ✅ Role-based access control across all 22 pages

---

## Repository Progress

### Files Updated This Week

```
Integration fixes:
  participant/ajax/chatbot.php     ← Model changed to openrouter/auto
  models/Booking.php               ← approve() auto-creates shift, cancel() datetime fix
  models/Document.php              ← trim() added to decryptData() result
  models/Admin.php                 ← getDashboardStats() WHERE is_active = 1 fix
  participant/bookings.php         ← Cost estimator null guard, CSRF fix
  participant/ajax/get_staff.php   ← Date format normalised to Y-m-d
  assets/js/main.js                ← News filter reset fix, slideshow image URL fix

New files:
  database/demo_accounts.sql       ← 3 demo accounts (admin, staff, participant)
```

### Commit Message Used
```
Week 9 Deliverable: Feature integration — end-to-end testing, bug fixes, working MVP
```

---

## Demo Accounts

Three demo accounts are available for testing and demonstration:

| Role | Email | Password |
|---|---|---|
| Admin | admin@adelaidecareconnect.com.au | password |
| Staff | staff@adelaidecareconnect.com.au | password |
| Participant | participant@adelaidecareconnect.com.au | password |

Run `database/demo_accounts.sql` in phpMyAdmin after importing `schema.sql`.

---

## Team Contributions This Week

| Member | Role | Contribution |
|---|---|---|
| Rabeel Riasat | Project Manager | Integration coordination, admin portal bug fixes, Assessment 4 submission |
| Muman Ghale | Backend Developer | Bug fixes on 6 models, AJAX endpoint fixes, chatbot model update |
| Ashish Neupane | Frontend Developer | Cost estimator fix, document download fix, cross-browser testing |
| Sanup Shrestha | UI/UX Designer | CSS news filter fix, final responsive layout checks |
| Samyog Bajgain | Database Designer | Encryption verification, demo accounts SQL, phpMyAdmin ERD screenshots |
| Bushan KC | QA & Documentation | Test cases TC-01 to TC-10 executed and documented, Assessment 4 report |

---

## Next Week (Week 10) — Testing

| Task | Assigned To |
|---|---|
| Execute test cases TC-11 to TC-15 | Bushan KC |
| Final backend security testing | Muman Ghale |
| Cross-browser compatibility — Chrome, Firefox, Edge | Ashish Neupane |
| Final CSS polish and responsive fixes | Sanup Shrestha |
| Database performance checks on key queries | Samyog Bajgain |
| Sprint 5 review and Assessment 5 preparation | Rabeel Riasat |

---

*Adelaide Care Connect | CPRO306 Capstone Project | Kent Institute Australia | Week 9 Deliverable | 2026*
