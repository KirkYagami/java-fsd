### **Project 02**

**Project Title**: LearnHub – Online Course Catalog & Enrollment System

**Project Description**: Build a structured e-learning platform where instructors create and organize courses into modules and lessons, students browse and enroll, and admins oversee and approve course listings. The system tracks per-student progress through course content and supports file uploads for learning materials — emphasizing content hierarchy modeling, enrollment workflows, and file handling.

**Objective**: Design a three-role system (Admin / Instructor / Student) with Spring Security and JWT Implement course hierarchy: `Course` → `Module` → `Lesson` with ordered content and MySQL relationships Build an enrollment system with status tracking (Pending → Active → Completed) Track student lesson-level progress and expose a percentage-complete metric per course Support file uploads for course materials (PDFs, slides) stored on AWS S3 Enable admin approval workflow for newly submitted courses before they go live Provide instructor dashboards showing enrollment counts and student progress per course

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT, BCrypt
- **Database**: MySQL with Hibernate ORM
- **File Storage**: AWS S3 (via AWS SDK for Java)
- **Frontend**: Thymeleaf or basic HTML/CSS with Bootstrap
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**: all weeks

**Project Type**: Intermediate content management and enrollment project emphasizing hierarchical data modeling, file handling, and progress tracking
**Outcome**: Delivered a working course platform supporting the full lifecycle from course creation and admin approval to student enrollment and progress tracking. The system demonstrates hierarchical JPA relationships, S3 integration, and role-gated content access — all within a clean, deployable Spring Boot monolith.
