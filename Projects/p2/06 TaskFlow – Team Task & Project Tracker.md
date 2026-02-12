### **Project 06**

**Project Title**: TaskFlow – Team Task & Project Tracker **Project Description**: Build a collaborative project management tool where teams create projects, break them down into tasks, assign members, and track progress through a Kanban-style status workflow. The system maintains a per-project activity log for transparency and includes deadline tracking with overdue detection — emphasizing workflow state management, team collaboration modeling, and audit logging.

**Objective**: Design a multi-role system (Admin / Project Manager / Team Member) with JWT auth and Spring Security Model entities: `Project`, `Task`, `TeamMember`, `ActivityLog`, `Comment` with MySQL relationships Implement Kanban task status workflow: To Do → In Progress → In Review → Done with transition validation Build team management — managers invite members, assign tasks, and set deadlines Track overdue tasks automatically and surface them on the dashboard Maintain an append-only `ActivityLog` table capturing all task and project changes with timestamps Allow task-level comments for team communication and context

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT, BCrypt
- **Database**: MySQL with Hibernate ORM
- **Frontend**: Thymeleaf or basic HTML/CSS with Bootstrap (Kanban board rendered via JS drag-and-drop or status buttons)
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**: all weeks

**Project Type**: Intermediate workflow and collaboration project emphasizing state machine design, audit logging, and team-scoped data access

**Outcome**: Delivered a working project tracker with enforced Kanban state transitions, deadline awareness, team-scoped access, and a transparent activity log. The system reflects real-world PM tool patterns and demonstrates workflow modeling, multi-user data scoping, and clean Spring architecture in an EC2-deployable monolith.
