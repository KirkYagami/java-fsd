### **Project 01**

**Project Title**: MediBook – Clinic Appointment Manager
**Project Description**: Build a secure, role-based clinic management system that enables patients to register and book appointments with doctors by specialty, while doctors manage their schedules and write post-visit notes. The system implements JWT-based authentication, role separation across three user types, and email notifications for appointment reminders — emphasizing real-world access control, relational data modeling, and transactional workflows.

**Objective**: Design a role-based user system supporting Admin, Doctor, and Patient with JWT authentication Implement appointment booking with conflict detection (no double-booking a doctor in the same slot) Model entities: `User`, `Doctor`, `Patient`, `Appointment`, `VisitNote` with proper MySQL relationships Build doctor schedule management — available slots, working hours, specialty filtering Send automated email notifications on booking confirmation and reminders via JavaMail Expose RESTful endpoints for appointment CRUD with status lifecycle (Scheduled → Completed → Cancelled) Provide an admin dashboard for user management and clinic-wide appointment overview

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT (jjwt library), BCrypt password encoding
- **Database**: MySQL with Hibernate ORM
- **Email**: JavaMailSender (SMTP)
- **Frontend**: Thymeleaf or basic HTML/CSS with Bootstrap
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**: all weeks

**Project Type**: Intermediate role-based web application emphasizing access control, scheduling logic, and relational data modeling

**Outcome**: Delivered a fully functional clinic management platform with secure multi-role access, real-time appointment handling, and automated email communication. The system demonstrates production-relevant patterns including JWT auth, constraint-aware scheduling, and a clean layered Spring architecture deployable on EC2.