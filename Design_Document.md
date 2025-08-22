# Design
 
## Problem statement
 
**"We need a comprehensive apartment booking system that supports individual and team accommodations with role-based allocation and administrative management"**
 
The customer requires an application to facilitate individual and team-based accommodation bookings while allowing administrators to manage and allocate accommodations efficiently. The system must support flexible date management, gender-based restrictions, role-based accommodation assignment (Manager â†’ Flat, Lead â†’ Room, Engineer â†’ Bed), and complex team booking logic with individual member cancellation capabilities. The application must integrate with the existing SIM tool built using React, Express, Node.js, and PostgreSQL.
 
## Implementation/Design
 
### Purpose and Usage
The application allows users to book accommodations for themselves or teams by selecting cities, dates, and team members. Administrators can review requests, allocate specific accommodations based on role and availability, and approve or reject bookings. The system enforces business rules including maximum 2-week stays, advance booking windows, and gender-based accommodation restrictions.
 
### Database Design Approach
- **Hierarchical Accommodation Structure**: Cities --> Apartments --> Flats --> Rooms --> Beds
- **Flexible Booking System**: Support for both individual and team bookings with smart resource allocation
- **Role-Based Access Control**: Three-tier user system (Manager, Lead, Project Engineer) with admin privileges
- **Comprehensive Audit Trail**: All actions logged with user attribution and timestamps
- **Gender-Based Restrictions**: Enum constraints ensuring accommodation compatibility
 
### Database Schema
 

 

 
### API Design Structure
```typescript
// REST API Endpoints (35 total)
 
// Authentication (3 endpoints)
POST   /api/auth/login
POST   /api/auth/logout  
GET    /api/auth/me
 
// User Management (2 endpoints)
GET    /api/users/company-members
GET    /api/users/{user_id}
 
// Location (2 endpoints)
GET    /api/cities
GET    /api/cities/{city_id}/apartments
 
// Booking (9 endpoints)
GET    /api/availability
GET    /api/availability/team
POST   /api/bookings
POST   /api/bookings/team
GET    /api/bookings/my-requests
GET    /api/bookings/{request_id}
PUT    /api/bookings/{request_id}/cancel
PUT    /api/bookings/cancel-member/{member_id}
PUT    /api/bookings/{request_id}/cancel-team
 
// Admin (15 endpoints)
GET    /api/admin/requests
GET    /api/admin/team-requests/{request_id}
POST   /api/admin/allocate
POST   /api/admin/allocate/member
PUT    /api/admin/approve/{request_id}
PUT    /api/admin/approve-team/{request_id}
PUT    /api/admin/reject/{request_id}
GET    /api/admin/occupancy
GET    /api/admin/booking-history
GET    /api/admin/export/bookings
GET    /api/admin/export/occupancy
POST   /api/admin/cities
POST   /api/admin/apartments
PUT    /api/admin/apartments/{apartment_id}
DELETE /api/admin/apartments/{apartment_id}
 
// Notifications (2 endpoints)
GET    /api/notifications
PUT    /api/notifications/{notification_id}/read
 
// System (2 endpoints)
GET    /api/health
GET    /api/version
```
 
### Individual Booking Sequence Diagram
<img width="3342" height="3418" alt="image" src="https://github.com/user-attachments/assets/9ea412ae-58c9-4fd2-b4ee-d556a52a12a9" />

 
### Team Booking Sequence Diagram
<img width="3468" height="8027" alt="image" src="https://github.com/user-attachments/assets/b1902835-5e4e-4665-b1af-a16a59c252c0" />


