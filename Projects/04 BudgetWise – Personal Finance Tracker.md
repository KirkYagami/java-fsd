### **Project 04**

**Project Title**: BudgetWise – Personal Finance Tracker

**Project Description**: Build a secure personal finance web application that enables users to log income and expenses, categorize transactions, set monthly budgets, and view spending summaries through visual charts. The system leverages MySQL aggregation queries for insightful reporting and supports CSV export of transaction history — emphasizing data integrity, query efficiency, and clean financial data modeling.

**Objective**: Design a transaction logging system supporting Income and Expense types with custom user-defined categories Implement budget management — users set monthly limits per category and receive over-budget alerts Model entities: `User`, `Transaction`, `Category`, `Budget` with proper MySQL relationships Build financial aggregation endpoints (monthly totals, category-wise spending, income vs. expense comparison) Enable filtering of transactions by date range, category, and type via query parameters Implement CSV export of transaction history for a given date range Render spending summary charts on the frontend using Chart.js or a lightweight equivalent

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT, BCrypt
- **Database**: MySQL with Hibernate ORM, native/JPQL aggregation queries
- **Frontend**: Thymeleaf + Chart.js for summary visualizations
- **Export**: Apache Commons CSV or OpenCSV for CSV generation
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**: all weeks
**Project Type**: Intermediate data-centric web application emphasizing SQL aggregation, financial modeling, and reporting

**Outcome**: Delivered a reliable personal finance tracker with accurate aggregation-backed summaries, budget alerting, and exportable transaction history. The system demonstrates real-world query design, clean entity relationships, and a user-friendly dashboard — packaged as a production-ready Spring Boot monolith on EC2.