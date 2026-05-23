# Week 7 — Backend Development
## Adelaide Care Connect | CPRO306 Capstone Project
**Kent Institute Australia | Group 8 | 2026**

---

## Overview

| Property | Detail |
|---|---|
| **Week** | Week 7 |
| **Theme** | Backend Development |
| **Deliverable** | Backend repository progress |
| **Lead** | Muman Ghale (Backend Developer) |
| **Support** | Samyog Bajgain (Database), Rabeel Riasat (Integration) |

---

## Activities Completed This Week

### 1. Server Setup & Environment Configuration

The development environment has been fully configured on all team members' machines using **XAMPP 8.1** running **Apache 2.4** and **MySQL 8.0**.

**Directory structure confirmed at:**
```
C:\xampp\htdocs\adelaide-care-connect\     (Windows)
/Applications/XAMPP/htdocs/adelaide-care-connect/   (Mac)
```

**PHP configuration verified (`php.ini`):**
```ini
extension=pdo_mysql      ; PDO MySQL driver — enabled
extension=openssl        ; AES-256 encryption — enabled
extension=curl           ; OpenRouter API calls — enabled
extension=fileinfo       ; File upload MIME detection — enabled
```

**Apache virtual host configured** to serve the project from `localhost/adelaide-care-connect`.

---

### 2. Database Connection Layer

**File:** `config/db.php`

Implemented a **PDO singleton pattern** ensuring a single database connection is reused throughout the request lifecycle. All queries use **prepared statements** to prevent SQL injection.

```php
function getDB(): PDO {
    static $pdo = null;
    if ($pdo === null) {
        $dsn = 'mysql:host=' . DB_HOST . ';dbname=' . DB_NAME . ';charset=utf8mb4';
        $pdo = new PDO($dsn, DB_USER, DB_PASS, [
            PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES   => false,
        ]);
    }
    return $pdo;
}
```

**Connection verified:** Successfully connects to `adelaide_care_connect` database with all 10 tables accessible.

---

### 3. Application Configuration

**File:** `config/config.php`

Central configuration file containing:

- **AES-256-CBC encryption helpers** for sensitive participant data
- **CSRF token generation and validation**
- **Session management** with secure settings
- **Flash message helpers** for user feedback
- **Redirect utility functions**

```php
// AES-256-CBC encryption for sensitive fields
function encryptData(string $data): string {
    return base64_encode(
        openssl_encrypt($data, 'AES-256-CBC', ENCRYPT_KEY, 0, ENCRYPT_IV)
    );
}

function decryptData(string $data): string {
    return openssl_decrypt(
        base64_decode($data), 'AES-256-CBC', ENCRYPT_KEY, 0, ENCRYPT_IV
    );
}

// CSRF token generation
function generateCsrfToken(): string {
    if (empty($_SESSION['csrf_token'])) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }
    return $_SESSION['csrf_token'];
}
```

**Encrypted fields in database:**
| Table | Field | Encryption |
|---|---|---|
| participants | ndis_number | AES-256-CBC |
| participants | dob | AES-256-CBC |
| participants | address | AES-256-CBC |
| documents | file_path | AES-256-CBC |
| users | password_hash | bcrypt cost=12 |

---

### 4. Authentication System

**File:** `controllers/AuthController.php`

Complete authentication system implemented and tested:

#### 4.1 User Registration
```
POST /register.php
```

**Validation rules implemented:**
| Field | Validation |
|---|---|
| first_name, last_name | Required, max 80 chars, alpha only |
| email | Required, valid email format, unique in DB |
| ndis_number | Required, exactly 9 digits (`/^\d{9}$/`) |
| dob | Required, valid date, participant must be 18+ |
| address | Required, max 255 chars |
| password | Required, min 8 chars, must contain uppercase, lowercase, number |
| password_confirm | Must match password |

**Process flow:**
```
1. Validate all input fields (server-side)
2. Check email is not already registered
3. Validate NDIS number format (9 digits)
4. Hash password using bcrypt (cost = 12)
5. AES-256 encrypt: ndis_number, dob, address
6. INSERT into users table (role = 'participant')
7. INSERT into participants table with encrypted fields
8. Set success flash message
9. Redirect to login page
```

#### 4.2 User Login
```
POST /login.php
```

**Security measures:**
- Password verified using `password_verify()` (bcrypt-aware)
- Failed login does NOT reveal which field was wrong (prevents enumeration)
- CSRF token validated on every POST request
- Session regenerated after successful login (`session_regenerate_id(true)`)
- Role stored in session for RBAC checks

**Role-based redirects after login:**
```php
match ($role) {
    'admin'       => redirect('admin/dashboard.php'),
    'staff'       => redirect('staff/dashboard.php'),
    'participant' => redirect('participant/dashboard.php'),
};
```

#### 4.3 Logout
```
POST /logout.php
```
- Destroys session data
- Clears session cookie
- Redirects to login page

---

### 5. Authentication Guard

**File:** `includes/auth_check.php`

Every protected page includes this guard at the top to verify the user is logged in and has the correct role.

```php
// Usage in participant pages:
requireRole('participant');

// Usage in staff pages:
requireRole('staff');

// Usage in admin pages:
requireRole('admin');
```

If the check fails, the user is redirected to the login page with a flash error message. This is applied to all 17 protected pages across the three portals.

---

### 6. Model Layer — All 5 Models Complete

All five model classes have been built and tested against the live database.

#### 6.1 User Model (`models/User.php`)
| Method | Description |
|---|---|
| `findByEmail($email)` | Fetch user record by email for login |
| `findById($id)` | Fetch user by primary key |
| `create($data)` | Insert new user + participant/staff record |
| `updateProfile($id, $data)` | Update first name, last name, phone |
| `updatePassword($id, $hash)` | Update bcrypt password hash |
| `deactivate($id)` | Soft-delete: set is_active = 0 |

#### 6.2 Booking Model (`models/Booking.php`)
| Method | Description |
|---|---|
| `create($data)` | Insert new booking, status = pending |
| `getByParticipant($pid)` | All bookings for a participant |
| `getById($id)` | Single booking with service and staff details |
| `cancel($id, $pid)` | Cancel if >24hrs before booking_date |
| `approve($id, $staffId, $notes)` | Approve + auto-create shift |
| `reject($id, $notes)` | Set status to rejected with notes |
| `getPending()` | All pending bookings for admin dashboard |
| `getAll($filters)` | All bookings with optional status/date filters |

**24-hour cancellation rule implementation:**
```php
public function cancel(int $bookingId, int $participantId): bool {
    $booking = $this->getById($bookingId);
    $bookingDateTime = new DateTime($booking['booking_date'] . ' ' . $booking['start_time']);
    $now = new DateTime();
    $diff = $now->diff($bookingDateTime);
    $hoursUntil = ($diff->days * 24) + $diff->h;

    if ($hoursUntil < 24) {
        throw new Exception('Bookings cannot be cancelled within 24 hours of the scheduled time.');
    }
    // Proceed with cancellation...
}
```

#### 6.3 Document Model (`models/Document.php`)
| Method | Description |
|---|---|
| `upload($userId, $file, $desc)` | Validate, store, encrypt file path |
| `getByUser($userId)` | All documents for a user |
| `getById($id)` | Single document record |
| `serve($id, $userId)` | Decrypt path, stream file to browser |
| `delete($id, $userId)` | Soft-delete: set is_active = 0 |

**File upload validation:**
```php
$allowedMimes = ['application/pdf', 'image/jpeg', 'image/png', 'application/msword',
                 'application/vnd.openxmlformats-officedocument.wordprocessingml.document'];
$maxSize = 5 * 1024 * 1024; // 5MB

if (!in_array($mimeType, $allowedMimes)) {
    throw new Exception('File type not allowed. Accepted: PDF, JPG, PNG, DOC, DOCX');
}
if ($fileSize > $maxSize) {
    throw new Exception('File size exceeds 5MB limit.');
}
```

#### 6.4 Shift Model (`models/Shift.php`)
| Method | Description |
|---|---|
| `create($staffId, $bookingId, $data)` | Auto-create shift when booking approved |
| `getByStaff($staffId)` | All shifts for a staff member |
| `accept($shiftId, $staffId)` | Set status = accepted |
| `decline($shiftId, $staffId, $reason)` | Set status = declined with reason |
| `getUpcoming($staffId)` | Future shifts sorted by date |
| `getDetail($shiftId)` | Shift + participant info (decrypted address) |

#### 6.5 Admin Model (`models/Admin.php`)
| Method | Description |
|---|---|
| `getDashboardStats()` | KPI counts for all 6 dashboard cards |
| `getAllParticipants()` | All participants with decrypted NDIS numbers |
| `getAllStaff()` | All staff with user details |
| `addStaff($data)` | Create user + staff record |
| `getAllBookings($filters)` | Bookings with participant + service details |
| `getAllServices()` | Full service catalogue |
| `updateService($id, $data)` | Edit service name, rate, description |
| `toggleService($id)` | Activate / deactivate a service |
| `getEnquiries()` | All public enquiries by submission date |
| `markEnquiryRead($id)` | Set is_read = 1 |
| `getNewsArticles()` | All articles for news management page |
| `publishArticle($id)` | Toggle is_published = 1 |

---

### 7. AJAX Endpoints

Two AJAX endpoints built to support front-end interactions without page reload.

#### 7.1 Chatbot Endpoint
```
POST /participant/ajax/chatbot.php
```

**Authentication:** Participant session required — returns HTTP 403 if not logged in.

**Request:**
```json
{ "message": "What services does Adelaide Care Connect offer?" }
```

**Process:**
```
1. Validate: not empty, under 1000 characters
2. Build messages array: system prompt + last 10 session turns + new message
3. POST to https://openrouter.ai/api/v1/chat/completions
4. Model: openrouter/auto
5. Save anonymised log (SHA-256 session token) to chatbot_logs table
6. Return response JSON
```

**Response:**
```json
{ "response": "Adelaide Care Connect offers six NDIS services...", "tokens": 142 }
```

**Error responses:**
| HTTP | Condition | Message |
|---|---|---|
| 400 | Empty message | `Message cannot be empty.` |
| 400 | Over 1000 chars | `Message too long.` |
| 403 | Not logged in | `Unauthorised.` |
| 429 | Rate limited | `Rate limit reached. Please wait.` |

#### 7.2 Get Available Staff Endpoint
```
GET /participant/ajax/get_staff.php?service_id=2&date=2026-05-20
```

**Authentication:** Participant session required.

**Logic:** Returns all active staff NOT already assigned to an accepted or pending shift on the requested date.

```sql
SELECT s.staff_id, u.first_name, u.last_name
FROM staff s
JOIN users u ON s.user_id = u.user_id
WHERE u.is_active = 1
AND s.staff_id NOT IN (
    SELECT sh.staff_id FROM shifts sh
    WHERE sh.shift_date = ? AND sh.status != 'declined'
)
ORDER BY u.first_name ASC
```

**Response:**
```json
[
  { "staff_id": 3, "first_name": "Sarah", "last_name": "Johnson" },
  { "staff_id": 5, "first_name": "James", "last_name": "Wilson" }
]
```

---

## Questions: What Endpoints and Validations Are Complete?

### ✅ Completed Endpoints

| Endpoint | Method | Auth | Status |
|---|---|---|---|
| `/register.php` | POST | Public | ✅ Complete |
| `/login.php` | POST | Public | ✅ Complete |
| `/logout.php` | POST | Any role | ✅ Complete |
| `/participant/bookings.php` (create) | POST | Participant | ✅ Complete |
| `/participant/bookings.php` (cancel) | POST | Participant | ✅ Complete |
| `/participant/documents.php` (upload) | POST | Participant | ✅ Complete |
| `/participant/ajax/chatbot.php` | POST | Participant | ✅ Complete |
| `/participant/ajax/get_staff.php` | GET | Participant | ✅ Complete |
| `/staff/shifts.php` (accept) | POST | Staff | ✅ Complete |
| `/staff/shifts.php` (decline) | POST | Staff | ✅ Complete |
| `/admin/bookings.php` (approve) | POST | Admin | ✅ Complete |
| `/admin/bookings.php` (reject) | POST | Admin | ✅ Complete |
| `/admin/participants.php` (view all) | GET | Admin | ✅ Complete |
| `/admin/staff.php` (add staff) | POST | Admin | ✅ Complete |
| `/admin/services.php` (update) | POST | Admin | ✅ Complete |
| `/admin/enquiries.php` (mark read) | POST | Admin | ✅ Complete |
| `/contact.php` (enquiry submit) | POST | Public | ✅ Complete |

### ✅ Completed Validations

| Validation | Location | Type |
|---|---|---|
| NDIS number must be exactly 9 digits | `AuthController.php` | Server-side regex |
| Email must be unique | `AuthController.php` | DB uniqueness check |
| Password minimum 8 characters | `AuthController.php` | Server-side |
| CSRF token on all POST requests | `config.php` | Server-side |
| File MIME type whitelist | `Document.php` | Server-side |
| File size maximum 5MB | `Document.php` | Server-side |
| 24-hour booking cancellation window | `Booking.php` | Server-side datetime |
| SQL injection prevention | All models | PDO prepared statements |
| XSS prevention | All views | `htmlspecialchars()` |
| Chatbot message max 1000 chars | `chatbot.php` | Server-side |
| Session role check on every page | `auth_check.php` | Server-side |

---

## Repository Progress

### Files Committed This Week

```
config/
├── config.php          ← AES-256 helpers, CSRF, session, flash messages
├── db.php              ← PDO singleton connection
└── openai.php          ← OpenRouter API key configuration

controllers/
└── AuthController.php  ← Registration, login, logout with bcrypt + RBAC

models/
├── User.php            ← User CRUD + profile management
├── Booking.php         ← Booking lifecycle with 24hr cancellation rule
├── Document.php        ← Secure file upload with encryption
├── Shift.php           ← Shift management for staff portal
└── Admin.php           ← All admin operations and dashboard stats

includes/
├── auth_check.php      ← requireRole() session guard
├── header.php          ← Shared HTML head, navbar, Font Awesome
└── footer.php          ← Shared footer with links

participant/ajax/
├── chatbot.php         ← OpenRouter AI chatbot handler
└── get_staff.php       ← Available staff AJAX endpoint
```

### Commit Message Used
```
Week 7 Deliverable: Backend development — auth system, models, API handlers
```

---

## Team Contributions This Week

| Member | Role | Contribution |
|---|---|---|
| Muman Ghale | Backend Developer | AuthController, all 5 models, AJAX endpoints, config files |
| Samyog Bajgain | Database Designer | Verified schema with models, tested all queries, confirmed encryption |
| Rabeel Riasat | Project Manager | Sprint review, GitHub PR reviews, integration coordination |
| Ashish Neupane | Frontend Developer | Connected registration and login forms to AuthController |

---

## Next Week (Week 8) — Frontend Development

| Task | Assigned To |
|---|---|
| Build participant portal pages (dashboard, bookings, documents, chatbot) | Ashish Neupane |
| Build staff portal pages (shifts calendar, shift detail) | Ashish Neupane |
| Build admin portal pages (dashboard, participants, bookings, services) | Rabeel Riasat |
| Connect booking form AJAX to get_staff.php endpoint | Ashish Neupane |
| Connect document upload form to Document model | Ashish Neupane |
| CSS enhancements — gradient buttons, improved KPI cards, sidebar glow | Sanup Shrestha |

---

*Adelaide Care Connect | CPRO306 Capstone Project | Kent Institute Australia | Week 7 Deliverable | 2026*
