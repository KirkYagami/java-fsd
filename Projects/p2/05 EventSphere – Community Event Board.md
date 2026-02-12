### **Project 05**

**Project Title**: EventSphere – Community Event Board

**Project Description**: Build a community-driven event platform where users post and discover local events, RSVP to attend, and engage through comments. Organizers manage their events and track attendee lists, while admins moderate platform content. The system supports image uploads for event banners and emphasizes social interaction modeling, content moderation workflows, and user engagement tracking.

**Objective**: Design a three-role system (Admin / Organizer / Attendee) with JWT authentication and Spring Security Model entities: `Event`, `RSVP`, `Comment`, `Category`, `User` with MySQL relationships Implement event RSVP system with capacity enforcement — prevent over-booking when a seat limit is set Build a comment system per event with admin moderation (flag and remove inappropriate content) Support image upload for event banners stored on AWS S3 Enable event filtering and search by category, date, and location keyword Provide organizer dashboards showing RSVP counts, attendee lists, and event engagement

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT, BCrypt
- **Database**: MySQL with Hibernate ORM
- **File Storage**: AWS S3 (via AWS SDK for Java)
- **Frontend**: Thymeleaf or basic HTML/CSS with Bootstrap
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**: all weeks

**Project Type**: Intermediate social platform project emphasizing user engagement modeling, content moderation, and capacity-aware transactional logic

**Outcome**: Delivered a fully functional community event board with secure multi-role access, capacity-aware RSVPs, moderated comments, and S3-backed media uploads. The project demonstrates social feature modeling and engagement tracking within a clean, EC2-deployable Spring Boot architecture.
