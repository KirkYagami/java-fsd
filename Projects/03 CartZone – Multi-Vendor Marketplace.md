### **Project 03**

**Project Title**: CartZone – Multi-Vendor Marketplace

**Project Description**: Build a multi-vendor e-commerce platform where sellers register stores and manage product listings, buyers browse and place orders, and admins oversee the entire marketplace. The system handles inventory control, a full order lifecycle, and role-separated dashboards — emphasizing transactional integrity, relational commerce modeling, and real-world order management workflows.

**Objective**: Implement a three-role system (Admin / Seller / Buyer) with JWT authentication and Spring Security Model core entities: `Store`, `Product`, `Order`, `OrderItem`, `Category` with MySQL relationships Build a product listing system with inventory tracking — auto-reduce stock on order placement Implement full order lifecycle management: Placed → Confirmed → Shipped → Delivered → Cancelled Add product search and filtering by category, price range, and seller Enforce inventory constraints — prevent orders exceeding available stock with proper error responses Provide seller dashboards (sales, orders, inventory) and admin views (platform-wide stats, user management)

**Tools Used**:

- **Backend Framework**: Spring Boot, Spring Security, Spring Data JPA
- **Authentication**: JWT, BCrypt
- **Database**: MySQL with Hibernate ORM
- **Frontend**: Thymeleaf or basic HTML/CSS with Bootstrap
- **Build & Deploy**: Maven, GitHub Actions CI/CD, AWS EC2

**Weeks (during training)**: all weeks
**Project Type**: Intermediate transactional e-commerce project emphasizing multi-role access, inventory management, and order lifecycle modeling

**Outcome**: Delivered a functioning multi-vendor marketplace with secure role separation, constraint-aware inventory handling, and a complete order management pipeline. The project demonstrates practical commerce patterns — transactional data integrity, relational modeling, and layered Spring architecture — in a fully EC2-deployable monolith.