# Design
 
## Problem statement
 
**"We need a comprehensive apartment booking system that supports individual and team accommodations with role-based allocation and administrative management"**
 
The customer requires an application to facilitate individual and team-based accommodation bookings while allowing administrators to manage and allocate accommodations efficiently. The system must support flexible date management, gender-based restrictions, role-based accommodation assignment (Manager â†’ Flat, Lead â†’ Room, Engineer â†’ Bed), and complex team booking logic with individual member cancellation capabilities. The application must integrate with the existing SIM tool built using React, Express, Node.js, and PostgreSQL.
 
## Implementation/Design
 
### Purpose and Usage
The application allows users to book accommodations for themselves or teams by selecting cities, dates, and team members. Administrators can review requests, allocate specific accommodations based on role and availability, and approve or reject bookings. The system enforces business rules including maximum 2-week stays, advance booking windows, and gender-based accommodation restrictions.
 
### Database Design Approach
- **Hierarchical Accommodation Structure**: Cities â†’ Apartments â†’ Flats â†’ Rooms â†’ Beds
- **Flexible Booking System**: Support for both individual and team bookings with polymorphic resource allocation
- **Role-Based Access Control**: Three-tier user system (Manager, Lead, Project Engineer) with admin privileges
- **Comprehensive Audit Trail**: All actions logged with user attribution and timestamps
- **Gender-Based Restrictions**: Enum constraints ensuring accommodation compatibility
 
### Database Schema
 
#### Core Entities
```sql
-- User Management
companies (company_id, company_name, description, created_at, updated_at, is_active)
users (user_id, company_id, first_name, last_name, email, phone, role, created_at, updated_at, is_active)
 
-- Location Hierarchy  
cities (city_id, city_name, state, country, created_at, updated_at, is_active)
apartments (apartment_id, city_id, apartment_name, address, total_flats, description, created_at, updated_at, is_active)
flats (flat_id, apartment_id, flat_number, floor, flat_type, total_rooms, max_occupancy, gender_restriction, created_at, updated_at, is_active)
rooms (room_id, flat_id, room_number, room_type, total_beds, max_occupancy, gender_restriction, created_at, updated_at, is_active)
beds (bed_id, room_id, bed_number, bed_type, gender_restriction, created_at, updated_at, is_active)
 
-- Booking Management
booking_requests (request_id, requester_id, city_id, is_team_booking, check_in_date, check_out_date, request_status, admin_remarks, requested_at, processed_at, processed_by)
booking_members (member_id, request_id, user_id, check_in_date, check_out_date, member_status, allocated_resource_type, allocated_resource_id, created_at, updated_at)
allocation_history (allocation_id, member_id, resource_type, resource_id, allocated_by, allocated_at, deallocated_at, allocation_status)
 
-- System Management
notifications (notification_id, user_id, request_id, notification_type, subject, message, is_read, sent_at, read_at)
audit_logs (log_id, user_id, action, table_name, record_id, old_values, new_values, ip_address, user_agent, created_at)
```
 
### Entity Relationship Diagram Code
```
// Database ERD for Apartment Booking System
// Use this code in eraser.io
 
User Management Group [color: blue] {
  Companies [icon: building] {
    + company_id (PK)
    + company_name
    + description
    + created_at, updated_at
    + is_active
  }
  
  Users [icon: users] {
    + user_id (PK)
    + company_id (FK)
    + first_name, last_name
    + email, phone
    + role (Manager/Lead/Engineer)
    + created_at, updated_at
    + is_active
  }
}
 
Location Management Group [color: green] {
  Cities [icon: map] {
    + city_id (PK)
    + city_name
    + state, country
    + created_at, updated_at
    + is_active
  }
  
  Apartments [icon: building-2] {
    + apartment_id (PK)
    + city_id (FK)
    + apartment_name
    + address
    + total_flats
    + created_at, updated_at
    + is_active
  }
  
  Flats [icon: home] {
    + flat_id (PK)
    + apartment_id (FK)
    + flat_number
    + floor, flat_type
    + total_rooms
    + gender_restriction
    + created_at, updated_at
    + is_active
  }
  
  Rooms [icon: door] {
    + room_id (PK)
    + flat_id (FK)
    + room_number
    + room_type
    + total_beds
    + gender_restriction
    + created_at, updated_at
    + is_active
  }
  
  Beds [icon: bed] {
    + bed_id (PK)
    + room_id (FK)
    + bed_number
    + bed_type
    + gender_restriction
    + created_at, updated_at
    + is_active
  }
}
 
Booking Management Group [color: orange] {
  BookingRequests [icon: calendar] {
    + request_id (PK)
    + requester_id (FK)
    + city_id (FK)
    + is_team_booking
    + check_in_date, check_out_date
    + request_status
    + admin_remarks
    + requested_at, processed_at
    + processed_by (FK)
  }
  
  BookingMembers [icon: user-group] {
    + member_id (PK)
    + request_id (FK)
    + user_id (FK)
    + check_in_date, check_out_date
    + member_status
    + allocated_resource_type
    + allocated_resource_id
    + created_at, updated_at
  }
  
  AllocationHistory [icon: history] {
    + allocation_id (PK)
    + member_id (FK)
    + resource_type
    + resource_id
    + allocated_by (FK)
    + allocated_at, deallocated_at
    + allocation_status
  }
}
 
System Management Group [color: purple] {
  Notifications [icon: bell] {
    + notification_id (PK)
    + user_id (FK)
    + request_id (FK)
    + notification_type
    + subject, message
    + is_read
    + sent_at, read_at
  }
  
  AuditLogs [icon: shield] {
    + log_id (PK)
    + user_id (FK)
    + action
    + table_name
    + record_id
    + old_values, new_values
    + ip_address, user_agent
    + created_at
  }
}
 
// Relationships
Companies ||--o{ Users : "employs"
Cities ||--o{ Apartments : "contains"
Apartments ||--o{ Flats : "contains"
Flats ||--o{ Rooms : "contains"
Rooms ||--o{ Beds : "contains"
Users ||--o{ BookingRequests : "creates"
Cities ||--o{ BookingRequests : "for location"
BookingRequests ||--o{ BookingMembers : "contains"
Users ||--o{ BookingMembers : "member of"
BookingMembers ||--o{ AllocationHistory : "allocated to"
Users ||--o{ Notifications : "receives"
BookingRequests ||--o{ Notifications : "about"
Users ||--o{ AuditLogs : "performs actions"
```
 
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

 
### Team Booking Sequence Diagram

