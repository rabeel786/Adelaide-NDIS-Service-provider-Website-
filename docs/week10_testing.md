# Week 10 — Testing
## Adelaide Care Connect | CPRO306 Capstone Project
**Kent Institute Australia | Group 8 | 2026**

---

## Overview

| Property | Detail |
|---|---|
| **Week** | Week 10 |
| **Theme** | Testing |
| **Deliverable** | Test Plan and Test Reports |
| **Lead** | Bushan KC (QA & Documentation) |
| **Support** | All team members |

---

## Activities Completed This Week

### 1. Testing Strategy

The team adopted a four-layer testing approach covering unit testing, integration testing, system testing, and user acceptance testing (UAT). All testing was carried out by Bushan KC using a structured test log with each test case documented by ID, description, preconditions, steps, expected result, actual result, and pass/fail status.

| Testing Layer | Scope | Method |
|---|---|---|
| Unit Testing | Individual functions and model methods | Manual execution with known inputs |
| Integration Testing | Frontend-to-backend connections, AJAX endpoints | Browser-based with network inspection |
| System Testing | End-to-end user journeys across all three portals | Full walkthrough per role |
| Security Testing | SQL injection, XSS, CSRF, direct URL access | Manual attack simulation |
| Cross-Browser Testing | Chrome, Firefox, Microsoft Edge | Visual and functional comparison |

---

### 2. Test Cases — Full Results (TC-01 to TC-15)

All 15 test cases defined in the SRS report have now been executed and documented.

#### 2.1 Authentication & Registration (TC-01 to TC-05)

| TC-ID | Description | Precondition | Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|
| TC-01 | Participant registration with valid 9-digit NDIS number | User not registered | Complete 3-step form with NDIS number 123456789 | Account created, redirected to login | Account created, redirect confirmed | ✅ Pass |
| TC-02 | Registration rejected when NDIS number is not 9 digits | User not registered | Enter NDIS number 12345 (5 digits) | Inline error: "NDIS number must be exactly 9 digits" | Error displayed, form not submitted | ✅ Pass |
| TC-03 | Successful login redirects to correct role dashboard | Valid accounts exist for all 3 roles | Log in with each of the 3 demo accounts | Admin → admin dashboard, Staff → staff dashboard, Participant → participant dashboard | All 3 redirects correct | ✅ Pass |
| TC-04 | Failed login does not reveal which field is incorrect | Valid email, wrong password | Enter correct email, wrong password | Generic error: "Invalid email or password" shown | Generic message shown, no field hint | ✅ Pass |
| TC-05 | Unauthenticated access redirected to login | User not logged in | Navigate directly to `/participant/dashboard.php` | Redirect to login with flash message | Redirected correctly with flash message | ✅ Pass |

#### 2.2 Booking System (TC-06 to TC-08)

| TC-ID | Description | Precondition | Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|
| TC-06 | Participant submits a booking successfully | Participant logged in, services exist | Select service, date, staff, time → submit | Booking created with status Pending, visible in My Bookings | Booking created and displayed with Pending badge | ✅ Pass |
| TC-07 | Booking cancellation within 24 hours is blocked | Booking exists within 24 hours | Click Cancel on a booking scheduled for today | Error: "Bookings cannot be cancelled within 24 hours" | Error message displayed, booking not cancelled | ✅ Pass |
| TC-08 | Booking cancellation more than 24 hours away succeeds | Booking exists for 3 days from now | Click Cancel on future booking → confirm | Booking status updated to Cancelled | Status updated to Cancelled in table | ✅ Pass |

#### 2.3 Document Management (TC-09)

| TC-ID | Description | Precondition | Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|
| TC-09 | Document upload — PDF accepted, executable rejected | Participant logged in | Upload a valid PDF (2MB) then attempt to upload an .exe file | PDF accepted and stored. .exe rejected with error message | PDF stored successfully. .exe blocked with "File type not allowed" error | ✅ Pass |

#### 2.4 Security Testing (TC-10 to TC-12)

| TC-ID | Description | Precondition | Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|
| TC-10 | SQL injection attempt on login form blocked | Login page accessible | Enter `' OR '1'='1` in email field, any password | Login fails, no data exposed, PDO prepared statement blocks injection | Login rejected, no SQL error, no data leak | ✅ Pass |
| TC-11 | XSS attempt in booking notes field blocked | Participant logged in | Enter `<script>alert('xss')</script>` in booking notes | Script not executed, text stored and displayed safely | Script rendered as plain text via `htmlspecialchars()` | ✅ Pass |
| TC-12 | CSRF token validation prevents cross-site form submission | Login page accessible | Submit login POST request without CSRF token | Request rejected with 403 error | 403 error returned, session not created | ✅ Pass |

#### 2.5 Staff Shift Management (TC-13 to TC-14)

| TC-ID | Description | Precondition | Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|
| TC-13 | Staff accepts a shift successfully | Shift assigned to staff | Log in as staff → My Shifts → click Accept on pending shift | Shift status updated to Accepted, no longer shows in pending list | Status updated to Accepted, removed from pending tab | ✅ Pass |
| TC-14 | Staff declines a shift with a reason | Shift assigned to staff | Log in as staff → My Shifts → click Decline → enter reason → submit | Shift status updated to Declined, reason stored in database | Status updated to Declined, reason visible in shift detail | ✅ Pass |

#### 2.6 Cross-Browser Compatibility (TC-15)

| TC-ID | Description | Precondition | Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|
| TC-15 | All pages render correctly across Chrome, Firefox, and Edge | Website running on localhost | Open each portal in Chrome, Firefox, Edge — test login, dashboard, booking | Consistent layout and functionality across all three browsers | Consistent rendering confirmed. Minor Firefox flexbox gap resolved | ✅ Pass |

---

### 3. Test Summary

| Category | Total | Passed | Failed | Pass Rate |
|---|---|---|---|---|
| Authentication & Registration | 5 | 5 | 0 | 100% |
| Booking System | 3 | 3 | 0 | 100% |
| Document Management | 1 | 1 | 0 | 100% |
| Security Testing | 3 | 3 | 0 | 100% |
| Staff Shift Management | 2 | 2 | 0 | 100% |
| Cross-Browser Compatibility | 1 | 1 | 0 | 100% |
| **TOTAL** | **15** | **15** | **0** | **100%** |

---

### 4. Bugs Found and Resolved During Testing

| # | Bug | Discovered In | Severity | Resolution | Status |
|---|---|---|---|---|---|
| 1 | Firefox flexbox gap causing service card misalignment on services page | TC-15 | Low | Added `-moz-` prefixed flexbox properties in `style.css` | ✅ Fixed |
| 2 | Edge browser not rendering CSS `backdrop-filter` on navbar | TC-15 | Low | Added `-webkit-backdrop-filter` fallback in `enhancements.css` | ✅ Fixed |
| 3 | Booking cancel button still visible after status changes to Cancelled | TC-08 | Medium | Added PHP condition: `if ($booking['status'] === 'pending')` before rendering Cancel button | ✅ Fixed |
| 4 | XSS in booking notes — `htmlspecialchars()` not applied on admin view | TC-11 | High | Added `htmlspecialchars()` to all admin booking note outputs in `admin/bookings.php` | ✅ Fixed |
| 5 | CSRF validation missing on staff accept/decline forms | TC-12 | High | Added `validateCsrfToken()` call to `staff/shifts.php` POST handler | ✅ Fixed |
| 6 | Shift detail page showing raw encrypted address string | TC-13 | Medium | Added `decryptData()` call on address field before passing to view | ✅ Fixed |
| 7 | Firefox date input returning different format (MM/DD/YYYY vs YYYY-MM-DD) | TC-15 | Medium | Added JavaScript date normalisation before AJAX call in booking form | ✅ Fixed |

---

### 5. Security Testing Detail

#### 5.1 SQL Injection — TC-10

**Attack vector tested:**
```sql
' OR '1'='1
' OR 1=1--
'; DROP TABLE users;--
admin'--
```

**Result:** All four injection attempts rejected. PDO prepared statements correctly separate SQL code from user input. Database tables unaffected.

**Code confirming protection:**
```php
// In AuthController.php — login method
$stmt = $db->prepare('SELECT * FROM users WHERE email = ? AND is_active = 1');
$stmt->execute([$email]);  // User input bound as parameter, never interpolated into SQL
```

#### 5.2 XSS — TC-11

**Attack vector tested:**
```html
<script>alert('xss')</script>
<img src="x" onerror="alert('xss')">
javascript:alert('xss')
```

**Result:** All three vectors rendered as plain text. `htmlspecialchars()` converts `<` to `&lt;` and `>` to `&gt;` before output, preventing script execution.

**High severity fix applied:** Admin booking notes display was missing `htmlspecialchars()`. Fixed in `admin/bookings.php`.

#### 5.3 Direct URL Access — Verified

All protected routes tested by navigating directly without a session:

```
http://localhost/adelaide-care-connect/admin/dashboard.php
http://localhost/adelaide-care-connect/participant/dashboard.php
http://localhost/adelaide-care-connect/staff/dashboard.php
http://localhost/adelaide-care-connect/participant/ajax/chatbot.php
```

**Result:** All four redirected to login page. AJAX endpoint returned HTTP 403 JSON error.

---

### 6. Final Backend Security Testing

**Muman Ghale** conducted a final security review of all backend code this week.

#### 6.1 Session Security Verified

```php
// Session configuration in config.php — confirmed secure
session_set_cookie_params([
    'lifetime' => 0,           // Session cookie expires when browser closes
    'path'     => '/',
    'secure'   => false,       // Set to true in production with HTTPS
    'httponly' => true,        // Cookie not accessible via JavaScript
    'samesite' => 'Strict',    // CSRF protection at cookie level
]);
session_start();
session_regenerate_id(true);   // Called on every login
```

#### 6.2 File Upload Security Confirmed

```php
// Document model — upload() method security checks
$finfo    = finfo_open(FILEINFO_MIME_TYPE);
$mimeType = finfo_file($finfo, $tmpPath);  // Read actual MIME, not extension

$allowed  = ['application/pdf','image/jpeg','image/png',
             'application/msword',
             'application/vnd.openxmlformats-officedocument.wordprocessingml.document'];

if (!in_array($mimeType, $allowed)) {
    throw new Exception('File type not allowed.');
}
// File stored in /uploads/documents/ which is ABOVE web root
// Direct browser access to uploaded files is blocked by .htaccess
```

#### 6.3 .htaccess Upload Protection Confirmed

```apache
# uploads/documents/.htaccess
Options -Indexes
Deny from all
```

This prevents any uploaded file from being accessed directly via a browser URL. All file serving goes through the PHP download handler which validates the session and decrypts the file path.

---

### 7. Cross-Browser Compatibility Report

Full cross-browser testing conducted across all 22 pages.

| Browser | Version Tested | Result | Issues Found |
|---|---|---|---|
| Google Chrome | 124 | ✅ Pass | None |
| Mozilla Firefox | 125 | ✅ Pass | Flexbox gap (fixed), date format (fixed) |
| Microsoft Edge | 124 | ✅ Pass | `backdrop-filter` prefix (fixed) |

**Pages verified in all 3 browsers:**

| Page | Chrome | Firefox | Edge |
|---|---|---|---|
| Home (slideshow) | ✅ | ✅ | ✅ |
| Services (category filter) | ✅ | ✅ | ✅ |
| About | ✅ | ✅ | ✅ |
| News (article list + single) | ✅ | ✅ | ✅ |
| Contact (form submit) | ✅ | ✅ | ✅ |
| Login | ✅ | ✅ | ✅ |
| Register (3 steps) | ✅ | ✅ | ✅ |
| Participant dashboard | ✅ | ✅ | ✅ |
| Participant bookings | ✅ | ✅ | ✅ |
| Participant documents | ✅ | ✅ | ✅ |
| Participant chatbot | ✅ | ✅ | ✅ |
| Participant profile | ✅ | ✅ | ✅ |
| Staff dashboard | ✅ | ✅ | ✅ |
| Staff shifts (calendar + list) | ✅ | ✅ | ✅ |
| Staff shift detail | ✅ | ✅ | ✅ |
| Staff profile | ✅ | ✅ | ✅ |
| Admin dashboard | ✅ | ✅ | ✅ |
| Admin participants | ✅ | ✅ | ✅ |
| Admin staff | ✅ | ✅ | ✅ |
| Admin bookings | ✅ | ✅ | ✅ |
| Admin services | ✅ | ✅ | ✅ |
| Admin news | ✅ | ✅ | ✅ |

---

### 8. Database Performance Checks

**Samyog Bajgain** ran performance checks on the key queries used throughout the system.

#### 8.1 Queries Analysed with EXPLAIN

```sql
-- Admin dashboard — participant count
EXPLAIN SELECT COUNT(*) FROM users WHERE role = 'participant' AND is_active = 1;
-- Result: Uses index on 'role' column — 0.001s

-- Participant bookings — most frequently run query
EXPLAIN SELECT b.*, s.service_name, u.first_name, u.last_name
FROM bookings b
JOIN services s ON b.service_id = s.service_id
LEFT JOIN users u ON b.staff_id = u.user_id
WHERE b.participant_id = ?
ORDER BY b.booking_date DESC;
-- Result: participant_id indexed via FK — 0.003s

-- Staff availability check for booking form
EXPLAIN SELECT s.staff_id, u.first_name, u.last_name
FROM staff s JOIN users u ON s.user_id = u.user_id
WHERE u.is_active = 1
AND s.staff_id NOT IN (
    SELECT sh.staff_id FROM shifts sh
    WHERE sh.shift_date = ? AND sh.status != 'declined'
);
-- Result: Subquery uses shift_date index — 0.004s
```

All three key queries execute under 5ms on the development dataset. Performance is acceptable for the scale of this project.

#### 8.2 chatbot_logs Privacy Verification

Confirmed that no PII is stored in the `chatbot_logs` table:

```sql
SELECT session_token, user_message, bot_response FROM chatbot_logs LIMIT 3;
```

The `session_token` column stores only SHA-256 hashed values — not user IDs, names, or NDIS numbers. Complies with the Australian Privacy Act 1988.

---

### 9. Final System Status

At the end of Week 10, the Adelaide Care Connect system is in a stable, tested state ready for the Assessment 6 final demonstration.

#### Feature Status Summary

| Portal | Pages | Features | Tested | Ready for Demo |
|---|---|---|---|---|
| Public Website | 5 | Slideshow, services, news, contact form | ✅ | ✅ |
| Participant Portal | 5 | Bookings, documents, AI chatbot, profile | ✅ | ✅ |
| Staff Portal | 4 | Shifts calendar, accept/decline, detail | ✅ | ✅ |
| Admin Portal | 8 | CRUD, approvals, enquiries, news | ✅ | ✅ |
| **Total** | **22** | **All features** | **15/15 tests pass** | **✅** |

#### Security Summary

| Security Measure | Implemented | Tested | Status |
|---|---|---|---|
| AES-256-CBC field encryption | ✅ | ✅ | ✅ Active |
| bcrypt password hashing | ✅ | ✅ | ✅ Active |
| CSRF token validation | ✅ | ✅ | ✅ Active |
| SQL injection prevention (PDO) | ✅ | ✅ | ✅ Active |
| XSS prevention (htmlspecialchars) | ✅ | ✅ | ✅ Active |
| Role-based access control | ✅ | ✅ | ✅ Active |
| File upload validation + .htaccess | ✅ | ✅ | ✅ Active |
| Chatbot logs anonymised | ✅ | ✅ | ✅ Active |

---

## Questions: What Are the Most Critical Test Cases? What Bugs Exist?

### Most Critical Test Cases

| Rank | Test Case | Why Critical |
|---|---|---|
| 1 | TC-10 — SQL Injection | Protects the entire database from attack |
| 2 | TC-12 — CSRF validation | Prevents unauthorised form submissions |
| 3 | TC-05 — Unauthenticated access | Protects all private portal data |
| 4 | TC-11 — XSS prevention | Protects all users from malicious scripts |
| 5 | TC-01 — NDIS number validation | Core business requirement from NDIA |
| 6 | TC-06 — Booking submission | Primary function of the entire system |
| 7 | TC-09 — Document upload security | Protects sensitive participant files |

### Remaining Known Bugs

No open bugs remain. All 7 bugs identified during testing have been resolved and verified.

| Bug | Status |
|---|---|
| Firefox flexbox gap on service cards | ✅ Fixed |
| Edge `backdrop-filter` fallback missing | ✅ Fixed |
| Cancel button visible after cancellation | ✅ Fixed |
| XSS in admin booking notes display | ✅ Fixed |
| CSRF missing on staff shift forms | ✅ Fixed |
| Encrypted address showing raw in shift detail | ✅ Fixed |
| Firefox date format mismatch in booking AJAX | ✅ Fixed |

---

## Repository Progress

### Files Updated This Week

```
Bug fixes from testing:
  assets/css/style.css              ← Firefox flexbox prefix added
  assets/css/enhancements.css       ← Edge backdrop-filter fallback
  participant/bookings.php          ← Cancel button condition fix
  admin/bookings.php                ← htmlspecialchars() on notes output
  staff/shifts.php                  ← CSRF token validation added
  staff/shift_detail.php            ← decryptData() on address field
  assets/js/main.js                 ← Firefox date normalisation in booking AJAX

New documentation files:
  docs/test_plan.md                 ← Full test plan with all 15 test cases
  docs/week10_testing.md            ← This file
```

### Commit Message Used
```
Week 10 Deliverable: Testing — 15 test cases executed, 7 bugs fixed, 100% pass rate
```

---

## Team Contributions This Week

| Member | Role | Contribution | Hours |
|---|---|---|---|
| Bushan KC | QA & Documentation | All 15 test cases executed and documented, test report written | 20 |
| Muman Ghale | Backend Developer | Final security review, CSRF fix on shifts, session security verified | 29 |
| Ashish Neupane | Frontend Developer | Cross-browser fixes, cancel button fix, date format normalisation | 24 |
| Sanup Shrestha | UI/UX Designer | Firefox and Edge CSS fixes, final visual consistency check | 18 |
| Samyog Bajgain | Database Designer | EXPLAIN query analysis, chatbot logs privacy verification | 18 |
| Rabeel Riasat | Project Manager | Sprint 5 review, Assessment 5 preparation, GitHub commit verification | 22 |

---

## Next Week (Week 11) — Assessment 5 Final Deliverables

| Task | Assigned To |
|---|---|
| Write Assessment 5 final project report | All members |
| Compile complete GitHub contribution evidence | Bushan KC |
| Prepare live demonstration script for Assessment 6 | Rabeel Riasat |
| Final database backup and clean schema export | Samyog Bajgain |
| Prepare individual explanation notes per team member | All members |

---

*Adelaide Care Connect | CPRO306 Capstone Project | Kent Institute Australia | Week 10 Deliverable | 2026*
