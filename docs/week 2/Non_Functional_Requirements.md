# Non-Functional Requirements

The following non-functional requirements define system quality attributes.

| ID | Category | Requirement | Metric |
|----|----------|------------|--------|
| NFR-01 | Performance | Pages must load quickly | < 3 seconds |
| NFR-02 | Security | Passwords must be securely hashed | Bcrypt |
| NFR-03 | Security | Sensitive data must be encrypted | AES-256 |
| NFR-04 | Security | Prevent SQL injection and XSS attacks | OWASP compliance |
| NFR-05 | Security | All data must be transmitted securely | HTTPS/SSL |
| NFR-06 | Availability | System must be available during business hours | 99% uptime |
| NFR-07 | Usability | System must follow accessibility standards | WCAG 2.1 AA |
| NFR-08 | Usability | UI must be responsive across devices | Mobile/Desktop |
| NFR-09 | Scalability | System should handle large number of users | 1000+ users |
| NFR-10 | Maintainability | Code must follow standard practices | PSR-12 |
| NFR-11 | Compatibility | System must work on major browsers | Chrome, Firefox, Edge |
| NFR-12 | Privacy | System must comply with privacy laws | Australian Privacy Act |

## Summary
These requirements ensure the system is secure, reliable, scalable, and user-friendly.
