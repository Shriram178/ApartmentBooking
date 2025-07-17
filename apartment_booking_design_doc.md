
# Estimation

**Time Estimation**  
The project will be developed in the following phases:

| Phase                                          | Target Date     |
| ---------------------------------------------- | ---------------- |
| Requirement Analysis                           | 10 Jul 2025      |
| System Design                                  | 11 Jul 2025      |
| Development                                    | 16 Jul 2025      |
|     - Database Schema Design                   | 14 Jul 2025      |
|     - Authentication & Role Logic              | 14 Jul 2025      |
|     - Booking Flow Implementation              | 14 Jul 2025      |
|     - Admin Dashboard & Drag-Drop UI           | 15 Jul 2025      |
|     - Mail Request Handling                    | 16 Jul 2025      |
|     - Analytics Dashboard                      | 16 Jul 2025      |
| Testing                                        | 17 Jul 2025      |
| Deployment & Integration                       | 18 Jul 2025      |

| **Total Estimated Time** | **7 days** |
| ------------------------ | ---------- |

# Design

## Problem Statement

**"Create a request-based apartment booking application where users can select cities and dates, submit requests, and admins allocate spaces based on roles and availability."**

Employees currently lack a streamlined method for booking temporary accommodations. This project aims to build a centralized portal where users request accommodations, and admins assign them dynamically through a role-aware allocation system.

## Features

### User Flow

- User logs in via Microsoft Outlook SSO.
- Selects city, check-in/out dates, check-in/out times.
- Chooses **booking type**: Individual or Team.
- Submits a request.
- Admin later approves and allocates appropriate bed/room/flat based on user role.

### Booking Rules Based on Role

| Role         | Default Allocation |
| ------------ | ------------------ |
| Engineer     | Bed                |
| Senior       | Room               |
| Manager      | Flat               |

- Admin can override and allocate any unit.
- Team bookings handled together with matching availability.

### Admin Panel

- Handles all incoming booking requests.
- Can assign beds/rooms/flats using dropdowns.
- Role-based logic assists with default suggestions.
- Red-dot indicator for pending requests.
- Toggle for individual/team request view.
- Drag-and-drop tree interface to manage:
  - Cities → Apartments → Flats → Rooms → Beds.
- Occupancy dashboard with filtering.
- Booking history with filters and type column.

### Data Import/Export

- Admin can **import/export** data in Excel (.xlsx) format for offline reporting.

### Apartment Configuration

- Context-aware creation of rooms/beds:
  - E.g., if admin is inside `Apartment A`, new room automatically belongs to it.
- Room/bed counts adjustable via UI.
- Bed names can be edited inline.
- Rooms/beds can be added/removed dynamically.

## Technology Stack

- **Frontend**: React (Next.js)
- **Backend**: Node.js + Express
- **Database**: PostgreSQL
- **Authentication**: Microsoft Outlook SSO (OAuth2)
- **UI Features**:
  - Calendar Picker
  - Tree View Component for Configuration
  - Red-dot for pending indicators
  - Toggle switch for request types
  - Drag & Drop UI using `react-beautiful-dnd`

## Database Schema (Simplified)

- `Users (Id, Name, Email, Role)`
- `Cities (Id, Name)`
- `Apartments (Id, CityId, Name)`
- `Flats (Id, ApartmentId, Name)`
- `Rooms (Id, FlatId, Name)`
- `Beds (Id, RoomId, Name)`
- `Requests (Id, RequestorId, Type, DateFrom, DateTo, TimeFrom, TimeTo, SubmittedAt, Status)`
- `Allocations (Id, RequestId, AllocatedBy, BedId?, RoomId?, FlatId?, AllocatedAt)`

## Booking Logic

- Admin sees a visual tree layout.
- Availability is inferred dynamically based on existing allocations.
- Default selection depends on requester role but can be overridden.
- Calendar view helps identify clashes or full bookings.

## Admin Dashboard

- Filter by city, apartment, role, request type, status.
- Export historical data for audit.
- View occupancy metrics and peak usage patterns.
- Interactive charts via Chart.js or Recharts.

## Non-Functional Requirements

- **Performance**: Optimized DB queries for availability and occupancy.
- **Scalability**: Designed to handle 1000+ concurrent requests.
- **Security**: SSO login; Role-based access.
- **Flexibility**: Modular design for reusability across other portals.
- **Resilience**: Manual override/request forwarding in failure cases.

## Summary

A modern, role-based request-driven apartment booking system that allows employees to submit booking intents, while giving admins the control to allocate based on availability. Integrated with Microsoft login, Excel import/export, and an intuitive admin UI for managing real estate inventory dynamically.
