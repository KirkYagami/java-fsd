### **Project 07**

**Project Title**: ApplyTrack – Job Application Tracker

**Project Description**: Build a personal job hunt management tool that allows users to log job applications, move them through a structured status pipeline, attach stage-specific notes, and upload resumes or supporting documents. A pipeline dashboard provides a visual overview of application health — emphasizing lifecycle state management, document handling, and meaningful personal productivity tooling.

**Objective**: Design a secure single-user-scoped system with JWT authentication ensuring users only access their own data Model entities: `Application`, `Company`, `StageNote`, `Document` with MySQL relationships Implement application status lifecycle: Wishlist → Applied → Interview → Offer → Rejected / Accepted Support multiple stage notes per application — timestamped entries capturing interview feedback and follow-ups Enable resume and document file uploads per application stored on AWS S3 Build a pipeline dashboard showing application counts per status stage and recent activity Add filtering and search across applications by company name, status, date range, and job title

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT, BCrypt
- **Database**: MySQL with Hibernate ORM
- **File Storage**: AWS S3 (via AWS SDK for Java)
- **Frontend**: Thymeleaf or basic HTML/CSS with Bootstrap
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**:all weeks

**Project Type**: Intermediate lifecycle management project emphasizing status workflows, document handling, and user-scoped data isolation

**Outcome**: Delivered a practical, fully personal job application tracker with a clean status pipeline, document management, and an at-a-glance dashboard. The project demonstrates user-scoped data design, file storage integration, and lifecycle state modeling — all packaged in a deployable Spring Boot monolith on EC2.