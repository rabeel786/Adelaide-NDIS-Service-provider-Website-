# Week 8 — Frontend Development
## Adelaide Care Connect | CPRO306 Capstone Project
**Kent Institute Australia | Group 8 | 2026**

---

## Overview

| Property | Detail |
|---|---|
| **Week** | Week 8 |
| **Theme** | Frontend Development |
| **Deliverable** | Frontend repository progress |
| **Lead** | Ashish Neupane (Frontend Developer) |
| **Support** | Sanup Shrestha (UI/UX), Rabeel Riasat (Admin Portal) |

---

## Activities Completed This Week

### 1. Public Website Pages

All five public-facing pages are complete, responsive, and connected to the live database.

| Page | File | Status |
|---|---|---|
| Home Page | `index.php` | ✅ Complete |
| About Page | `about.php` | ✅ Complete |
| Services Page | `services.php` | ✅ Complete |
| News & Resources | `news.php` | ✅ Complete |
| Contact Page | `contact.php` | ✅ Complete |

#### 1.1 Home Page (`index.php`)

The home page is the primary landing page for public visitors and NDIS participants. Key sections implemented:

- **Hero section** with 5-slide auto-rotating background slideshow (5.5 second intervals), left/right arrow controls, and dot indicators. Crossfade transition of 1.6 seconds between slides. Pauses on mouse hover.
- **Quick links bar** — navy accent bar with 4 action links (Register, Services, Contact, My Portal)
- **Services grid** — 6 service cards pulled dynamically from the `services` database table. Each card has a photo, category badge, description, hourly rate, and a Book Now / Register to Book button
- **Mission section** — two-column layout with photo and text, floating stat card ("500+ Lives Supported"), NDIS badge
- **How It Works** — 3-step visual cards (Register → Book → Receive Support) with photos
- **Impact stats** — full-width photo background section with 4 animated counters (500+ participants, 2400+ services, 50+ workers, 98% satisfaction)
- **Testimonials** — 3 quote cards with profile photos; centre card featured in navy
- **News section** — 3 latest articles from the database with photo backgrounds and category badges
- **CTA banner** — photo background with Register and Contact buttons

**Hero slideshow JavaScript:**
```javascript
(function(){
  var slides  = document.querySelectorAll('.hero-slide');
  var dots    = document.querySelectorAll('.hero-dot');
  var current = 0, timer = null;

  function goTo(n){
    slides[current].classList.remove('active');
    dots[current].classList.remove('active');
    current = (n + slides.length) % slides.length;
    slides[current].classList.add('active');
    dots[current].classList.add('active');
  }

  function reset(){ clearInterval(timer); timer = setInterval(function(){ goTo(current+1); }, 5500); }
  reset();

  document.getElementById('heroNext').addEventListener('click', function(){ goTo(current+1); reset(); });
  document.getElementById('heroPrev').addEventListener('click', function(){ goTo(current-1); reset(); });
})();
```

#### 1.2 Services Page (`services.php`)

- Hero section with photo background and breadcrumb navigation
- Category filter buttons (All, Daily Living, Community, Therapeutic, Coordination, Plan Management)
- 6 service cards with photos pulled from the database, each showing hourly rate
- JavaScript filter: clicking a category hides cards not matching that category with smooth CSS transition
- Participants see a **Book Now** button; guests see **Register to Book**

#### 1.3 News Page (`news.php`)

- Hero section with photo background
- Category filter tabs (All, News, NDIS Update, Health Tip, Resource)
- 8 article cards with photo thumbnails, category pills, titles, excerpts, and publication dates
- Single article view — clicking **Read More** loads full article content from the database
- Full HTML content rendered using the article's stored rich text

#### 1.4 Contact Page (`contact.php`)

- Hero section with photo background
- Two-column layout: contact details + enquiry form
- Phone, email, address, and business hours displayed
- Enquiry form with fields: name, email, phone, subject, message
- Server-side validation + CSRF token protection
- On submission, saves to `enquiries` database table and displays success message

---

### 2. Login & Registration Pages

#### 2.1 Login Page (`login.php`)

- Split two-column layout — dark branded panel (left) + white form panel (right)
- Left panel lists portal benefits with tick icons
- Form fields: Email Address, Password with show/hide toggle
- Remember me checkbox
- CSRF token hidden field on form
- Inline error messages on failed login (generic — does not reveal which field failed)
- Demo credentials info box for presentation purposes
- Responsive: left panel hidden on mobile, form takes full width

#### 2.2 Registration Page (`register.php`) — 3-Step Form

**Step 1 — Personal Details:**
- First Name, Last Name, Date of Birth, Phone Number
- Street Address, Suburb, State (defaulting to SA), Postcode
- Emergency Contact Name, Phone, Relationship
- AES-256 encryption notice displayed to user

**Step 2 — NDIS Details:**
- NDIS Number (9-digit validation with real-time JavaScript feedback)
- NDIS Plan Start Date, NDIS Plan End Date
- Support Needs (text area)

**Step 3 — Account Setup:**
- Email Address (uniqueness checked on blur via AJAX)
- Password with strength indicator bar (weak / fair / strong / very strong)
- Confirm Password with match validation

**Progress indicator:** 3 circles connected by lines. Active step is navy filled, completed steps show green tick, pending steps are gray.

**Client-side JavaScript validation on each step:**
```javascript
// NDIS number — must be exactly 9 digits
document.getElementById('ndis_number').addEventListener('input', function(){
    const val = this.value.replace(/\D/g,'');
    const valid = /^\d{9}$/.test(val);
    this.style.borderColor = valid ? '#22C55E' : '#EF4444';
    document.getElementById('ndis_hint').textContent =
        valid ? '✓ Valid NDIS number' : `${9 - val.length} digits remaining`;
});
```

---

### 3. Participant Portal

All 5 participant portal pages implemented and connected to the backend models.

#### 3.1 Participant Dashboard (`participant/dashboard.php`)

- Page header: "Welcome back, [First Name]! 👋"
- **4 KPI cards:** Total Bookings, Upcoming Bookings, Documents Stored, AI Chatbot (quick access)
- **Recent Bookings table:** service name, date, time range, status badge (Pending / Approved / Cancelled)
- **Quick Actions bar:** Book a Service, Upload Document, View Schedule, Edit Profile buttons
- All data pulled live from the database using Booking and Document models

#### 3.2 My Bookings Page (`participant/bookings.php`)

- Status filter tabs: All, Pending, Approved, Completed, Cancelled
- Bookings table with service, staff name, date, time, status, and action buttons
- **Book a Service modal** triggered by button click:
  - Service dropdown (from database)
  - Date picker
  - Staff preference dropdown — populated via AJAX call to `get_staff.php` when date is selected
  - Start time / end time selectors
  - **Cost estimator:** calculates `(end_time - start_time) × hourly_rate` in real time
  - Additional notes text area
  - CSRF token on form
- **Cancel button** — shows confirmation modal before submitting; server enforces 24-hour rule

**Cost estimator JavaScript:**
```javascript
function updateCost(){
    const start = document.getElementById('start_time').value;
    const end   = document.getElementById('end_time').value;
    const rate  = parseFloat(document.getElementById('hourly_rate').value);
    if(start && end && rate){
        const diff = (new Date('1970-01-01T'+end) - new Date('1970-01-01T'+start)) / 3600000;
        document.getElementById('cost_estimate').textContent =
            diff > 0 ? '$' + (diff * rate).toFixed(2) : 'Invalid time range';
    }
}
```

#### 3.3 My Documents Page (`participant/documents.php`)

- KPI cards: Total Files, Storage Used, Encryption Status (AES-256 lock icon)
- Document grid: file name, file type icon, size, upload date, description
- Upload area with drag-and-drop support
- Accepted formats shown: PDF, JPG, PNG, DOC, DOCX — max 5MB
- Download button streams the file through PHP (decrypts path, serves securely)
- Delete button soft-deletes record (is_active = 0)
- Encryption notice: "🔒 All files encrypted with AES-256"

#### 3.4 AI Chatbot Page (`participant/chatbot.php`)

- Chat window with scroll
- **Bot avatar** (navy circle with robot icon) for AI messages
- **User avatar** (orange circle with first name initial) for participant messages
- **Welcome message** on load with 4 quick-question suggestion chips:
  - "What is the NDIS?"
  - "How do I book a service?"
  - "What services do you offer?"
  - "How is my data protected?"
- Message input bar with character counter (0/1000)
- Send button + Enter key support
- Typing indicator (3 animated bouncing dots) shown while waiting for response
- **New Chat** button clears session history
- Timestamps on every message
- Error messages displayed inline if API returns an error
- All messages sent via `fetch()` to `ajax/chatbot.php`

**Fetch call to chatbot AJAX:**
```javascript
const res = await fetch('ajax/chatbot.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message: input.value.trim() })
});
const data = await res.json();
if(data.error){
    appendMessage('bot', '⚠ ' + data.error);
} else {
    appendMessage('bot', data.response);
}
```

#### 3.5 My Profile Page (`participant/profile.php`)

- Displays current user details: name, email, phone, suburb, state
- Edit form for name and phone (email cannot be changed)
- Change password section: current password, new password, confirm — all validated
- NDIS plan dates displayed (read-only)
- Emergency contact details displayed

---

### 4. Staff Portal

All 4 staff portal pages implemented.

#### 4.1 Staff Dashboard (`staff/dashboard.php`)

- Welcome banner with staff name
- KPI cards: Total Shifts, Upcoming Shifts, Pending Response, Completed
- Alert banner if there are shifts awaiting acceptance (amber warning)
- Recent shifts table with date, service, participant name, status badge
- Quick navigation to My Shifts

#### 4.2 Staff Shifts Page (`staff/shifts.php`)

**Two view modes toggled by buttons:**

**Calendar View:**
- Weekly calendar grid (Mon–Sun)
- Previous/Next week navigation
- Each shift displayed as a coloured block inside the day cell:
  - 🟡 Amber — Awaiting response
  - 🟢 Green — Accepted
  - 🔴 Red — Declined
- Click a shift block → navigates to shift detail page

**List View:**
- Tabbed: Upcoming / Pending / Completed
- Table showing date, time, service, participant, hours, status
- Accept / Decline buttons on pending shifts

#### 4.3 Shift Detail Page (`staff/shift_detail.php`)

- Full shift information: date, time, duration, estimated pay
- **Participant information:** first name, suburb (address decrypted for display)
- **Emergency contact:** name, relationship, phone
- **Accept button** — sets shift status to accepted
- **Decline button** — opens reason modal, submits with decline reason text
- Note: participant's full address and NDIS number are only shown on this page, not in lists

#### 4.4 Staff Profile Page (`staff/profile.php`)

- Current details: name, email, phone, position, qualifications
- Edit form for phone and availability notes
- Change password section

---

### 5. Admin Portal

All 8 admin portal pages implemented.

#### 5.1 Admin Dashboard (`admin/dashboard.php`)

- **6 KPI cards** (2 rows of 3):
  - Active Participants, Active Staff, Pending Bookings, Open Enquiries, Total Bookings, Documents Stored
- **Recent Bookings table** — last 10 bookings with Approve / Reject action buttons
- **Recent Activity feed** — latest system events (new bookings, new enquiries, approved bookings)
- **Quick Actions bar** — Add Staff, Manage Bookings, View Enquiries, Add Service buttons

#### 5.2 Participants Page (`admin/participants.php`)

- Search bar filtering by name or email
- Table: name, email, phone, suburb, NDIS plan dates, status badge, View button
- Participant count shown

#### 5.3 Staff Page (`admin/staff.php`)

- Staff listing table with name, email, position, status
- **Add Staff modal:** first name, last name, email, phone, position, qualifications — creates user account with temporary password

#### 5.4 Bookings Page (`admin/bookings.php`)

- Status filter cards: Pending, Approved, Rejected, Completed counts
- Search and filter by status and date
- **Approve modal:**
  - Shows participant name, service, date, time
  - Staff member dropdown (all active staff)
  - Admin notes text area
  - Confirm Approval button — sets status to approved, auto-creates shift record
- **Reject modal:**
  - Rejection reason text area
  - Confirm Rejection button

#### 5.5 Services Page (`admin/services.php`)

- Active / Inactive toggle for each service
- Edit modal: service name, category, description, hourly rate, icon

#### 5.6 Documents Page (`admin/documents.php`)

- All uploaded participant documents with owner, file type, size, upload date
- Download button for admin access

#### 5.7 News & Resources Page (`admin/news.php`)

- Article listing with title, category, published status, date
- Publish / Unpublish toggle
- Full article content stored as HTML in database

#### 5.8 Enquiries Page (`admin/enquiries.php`)

- Enquiry inbox with unread count badge
- Table: name, email, subject, submission date, read status
- Mark as Read button
- Reply via Email link (opens default mail client with pre-filled recipient)

---

## Questions: Which Core User Journeys Are Working?

### ✅ Fully Working User Journeys

| Journey | Steps | Status |
|---|---|---|
| **Public visitor views services** | Home → Services → view card with pricing | ✅ Working |
| **Participant registers** | Register (3 steps) → validates NDIS → redirects to login | ✅ Working |
| **Participant logs in** | Login → bcrypt verify → session → participant dashboard | ✅ Working |
| **Participant books a service** | Dashboard → New Booking → select service → AJAX loads staff → cost estimate → submit → Pending | ✅ Working |
| **Admin approves a booking** | Admin dashboard → Bookings → Approve → assign staff → confirm → Approved + shift created | ✅ Working |
| **Staff accepts a shift** | Staff dashboard → My Shifts → view detail → Accept → Accepted | ✅ Working |
| **Staff declines a shift** | Staff dashboard → Shift detail → Decline → enter reason → Declined | ✅ Working |
| **Participant uploads a document** | Documents page → upload PDF → validates type and size → encrypted and stored | ✅ Working |
| **Participant uses chatbot** | Chatbot page → type question → AJAX to OpenRouter → response displayed | ✅ Working |
| **Public submits enquiry** | Contact page → fill form → submit → saved to DB → success message | ✅ Working |
| **Admin reads enquiry** | Admin → Enquiries → view → Mark as Read | ✅ Working |

### 🔄 Journeys In Testing

| Journey | Status | Notes |
|---|---|---|
| Participant cancels booking | 🔄 In Testing | 24hr rule enforced, edge cases being verified |
| Admin publishes news article | 🔄 In Testing | Toggle working, rich text display being confirmed |

---

## CSS Design System

**File:** `assets/css/style.css` (1,710 lines) + `assets/css/enhancements.css` (968 lines)

The design system is built entirely with **custom CSS variables** — no Bootstrap or external CSS framework.

### Colour Tokens
```css
:root {
  --primary:       #1F4E79;   /* Navy blue — headings, navbar, primary buttons */
  --primary-dark:  #0F2D4A;   /* Dark navy — sidebar, footer */
  --primary-light: #2E75B6;   /* Mid blue — links, sub-headings */
  --primary-xlight:#D6E4F0;   /* Light blue — card backgrounds */
  --accent:        #F4A261;   /* Orange — CTA buttons, active nav, badges */
  --success:       #22C55E;   /* Green — approved status, pass results */
  --warning:       #F59E0B;   /* Amber — pending status, warnings */
  --error:         #EF4444;   /* Red — rejected status, error messages */
  --bg:            #F0F4F8;   /* Off-white — page background */
}
```

### Key UI Components Built
| Component | Description |
|---|---|
| `.navbar` | Sticky dark glass navbar with backdrop blur, accent logo, active underline |
| `.hero--slideshow` | Full-height hero with absolute-positioned slides and crossfade transition |
| `.kpi-card` | Stat card with gradient icon circle, hover lift effect |
| `.dash-card` | Dashboard section card with tinted header and subtle shadow |
| `.service-card` | Card with photo, overlay icon, hover zoom and border animation |
| `.sidebar` | Dark gradient sidebar with glowing orange active state |
| `.data-table` | Dark gradient header, hover row highlight |
| `.badge` | Coloured pill badges for all status values |
| `.btn--primary` | Gradient navy button with lift on hover |
| `.btn--accent` | Gradient orange button with glow shadow |
| `.modal-overlay` | Blurred dark backdrop with spring animation on modal open |
| `.chat-msg` | Chat bubble — gradient navy for user, white card for bot |
| `.step-indicator` | Multi-step form progress circles with connecting lines |
| `.how-step` | How It Works card with photo, step number, and description |
| `.testimonial-card` | Quote card with avatar, stars, featured variant in navy |
| `.impact-section` | Full-width fixed background photo with stat grid overlay |

### Responsive Breakpoints
```css
@media (max-width: 1024px) { /* Tablet — 2 column, collapse sidebar */ }
@media (max-width: 768px)  { /* Mobile — single column, hamburger menu */ }
@media (max-width: 480px)  { /* Small mobile — full width buttons, 1 col KPI */ }
```

---

## Repository Progress

### Files Committed This Week

```
Public pages:
  index.php               ← Home page with slideshow and all sections
  about.php               ← About page with mission, values, certifications
  services.php            ← Services page with category filter
  news.php                ← News page with article listing and single view
  contact.php             ← Contact page with enquiry form
  login.php               ← Split layout login with role-based redirect
  register.php            ← 3-step registration with NDIS validation

Participant portal:
  participant/dashboard.php    ← KPI cards, bookings table, quick actions
  participant/bookings.php     ← Booking management with modal and cost estimator
  participant/documents.php    ← Secure upload with drag-and-drop
  participant/chatbot.php      ← AI chatbot interface with typing indicator
  participant/profile.php      ← Profile management and password change

Staff portal:
  staff/dashboard.php          ← Welcome, KPIs, shift summary
  staff/shifts.php             ← Calendar and list view with accept/decline
  staff/shift_detail.php       ← Full shift info with participant details
  staff/profile.php            ← Staff profile and availability notes

Admin portal:
  admin/dashboard.php          ← 6 KPI cards, bookings table, activity feed
  admin/participants.php       ← Participant listing with search
  admin/staff.php              ← Staff management with Add Staff modal
  admin/bookings.php           ← Booking approval/rejection with modals
  admin/services.php           ← Service catalogue CRUD
  admin/documents.php          ← Document viewer
  admin/news.php               ← News article management
  admin/enquiries.php          ← Enquiry inbox

Assets:
  assets/css/style.css         ← Full design system (1,710 lines)
  assets/css/enhancements.css  ← Visual enhancements (968 lines)
  assets/js/main.js            ← Slideshow, filters, chatbot, forms
```

### Commit Message Used
```
Week 8 Deliverable: Frontend development — all portals, UI components, AJAX integration
```

---

## Team Contributions This Week

| Member | Role | Contribution |
|---|---|---|
| Ashish Neupane | Frontend Developer | All participant portal pages, login, register, contact, news, AJAX integration |
| Sanup Shrestha | UI/UX Designer | CSS enhancements, services page, about page, design system tokens |
| Rabeel Riasat | Project Manager | Admin portal all 8 pages, sprint review, GitHub PR coordination |
| Muman Ghale | Backend Developer | Bug fixes on models, AJAX endpoint adjustments, session handling fixes |

---

## Next Week (Week 9) — Feature Integration

| Task | Assigned To |
|---|---|
| End-to-end testing of all user journeys | Bushan KC |
| Complete remaining test cases TC-06 to TC-10 | Bushan KC |
| Fix integration issues found during testing | Muman Ghale |
| Assessment 4 report — system architecture section | Rabeel Riasat |
| Assessment 4 report — database and UI sections | Samyog Bajgain + Sanup Shrestha |
| Take all system screenshots for Assessment 4 report | All members |
| Submit Assessment 4 to Moodle | Rabeel Riasat |

---

*Adelaide Care Connect | CPRO306 Capstone Project | Kent Institute Australia | Week 8 Deliverable | 2026*
