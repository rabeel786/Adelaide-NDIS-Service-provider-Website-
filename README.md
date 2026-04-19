
# **Adelaide Care Connect – NDIS Service Provider Website**

## **Project Overview**

Adelaide Care Connect is a full-stack web-based information system designed for an NDIS (National Disability Insurance Scheme) service provider in Adelaide, Australia.

The system aims to replace manual and fragmented processes with a centralized digital platform that improves:

* Client management
* Staff scheduling
* Service bookings
* Document handling
* Overall operational efficiency

---

## **Project Objectives**

* Provide a secure, role-based platform for **Participants, Staff, and Admins**
* Enable **online service booking and scheduling**
* Digitise **client registration and document storage**
* Integrate an **AI-powered chatbot** for NDIS-related queries
* Ensure compliance with:

  * Australian Privacy Act 1988
  * NDIS Practice Standards
* Deliver a **responsive and accessible UI (WCAG 2.1 AA)**

---

## **System Features**

### **Participant Portal**

* Register and manage profile
* Book and manage services
* Upload and access documents
* Interact with AI chatbot

### **Staff Portal**

* View and manage shifts
* Accept/decline shift requests
* Manage availability

### **Admin Portal**

* Manage participants and staff
* Approve/reject bookings
* Manage services and documents
* View reports and dashboards

### **Public Website**

* Service information
* News & resources
* Contact/enquiry form

---

## **Tech Stack**

| Layer           | Technology              |
| --------------- | ----------------------- |
| Front-End       | HTML5, CSS3, JavaScript |
| Back-End        | PHP 8.x                 |
| Database        | MySQL 8.x               |
| Server          | Apache (XAMPP/WAMP)     |
| AI Integration  | OpenAI GPT API          |
| Version Control | GitHub                  |

---

## **System Architecture**

The application follows a **Three-Tier Architecture**:

1. **Presentation Layer** – UI (HTML, CSS, JS)
2. **Application Layer** – PHP backend logic
3. **Data Layer** – MySQL database

---

## **Project Structure**

```
/adelaide-care-connect/
│
├── public/              # Public-facing files
│   ├── index.php
│   ├── css/
│   ├── js/
│   └── images/
│
├── src/                 # Core application logic
│   ├── controllers/
│   ├── models/
│   ├── views/
│   └── config/
│
├── storage/
│   └── uploads/         # Secure document storage
│
├── database/
│   ├── schema.sql
│   └── seed.sql
│
├── .env
└── README.md
```

---

## **Development Methodology**

This project follows an **Agile Scrum approach**:

* 2-week sprints
* Sprint planning & reviews
* Continuous integration via GitHub
* Iterative SDLC (Requirements → Design → Development → Testing)

---

## **Team Members**

| Name           | Role                |
| -------------- | ------------------- |
| Rabeel Riasat  | Project Manager     |
| Sanup Shrestha | Business Analyst    |
| Samyog Bajgain | Database Designer   |
| Ashish Neupane | Requirements Engineer |
| Muman Ghale    | Back-End Developer  |
| Bhushan     | QA & Documentation  |

---

## **Key Functionalities**

* Secure authentication with role-based access
* Service booking and scheduling system
* Document upload with access control
* AI chatbot integration
* Admin dashboard with reporting features

---

## **Security Features**

* Bcrypt password hashing
* AES-256 data encryption
* Prepared statements (SQL injection prevention)
* HTTPS/SSL enforcement
* Role-Based Access Control (RBAC)

---

## **Testing**

The system includes:

* Unit Testing
* Integration Testing
* System Testing
* User Acceptance Testing (UAT)

Sample test cases include:

* Registration validation
* Booking workflows
* Document uploads
* Security testing (SQL injection, XSS)

---

## **Setup Instructions**

### **1. Clone Repository**

```bash
git 
```

### **2. Setup Environment**

* Install XAMPP/WAMP
* Place project in `htdocs`

### **3. Database Setup**

* Import `database/schema.sql`
* Configure database in:

```
/src/config/db.php
```

### **4. Run Project**

* Start Apache & MySQL
* Open in browser:

```
http://localhost/adelaide-care-connect
```

---

## **Future Enhancements**

* Mobile application (iOS/Android)
* Integration with NDIS systems
* Advanced analytics dashboard
* Telehealth functionality

---

## **License**

This project is developed for academic purposes at Kent Institute Australia.

---

## **Acknowledgements**

* NDIS Framework (NDIA)
* Australian Privacy Act 1988
* OpenAI API Documentation
