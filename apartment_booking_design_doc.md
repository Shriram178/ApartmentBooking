# Estimation

Time Estimation\
The project will be developed in the following phases:

| Phase                                          | Estimated Time |
| ---------------------------------------------- | -------------- |
| Requirement Analysis                           | 1 day          |
| System Design                                  | 1 day          |
| Development                                    | 3 days         |
|     - Database Schema Design                   | 3 hours        |
|     - Authentication & Role Logic              | 4 hours        |
|     - Booking Flow Implementation              | 1 day          |
|     - Admin Dashboard & Drag-Drop UI           | 1 day          |
|     - Mail Request Handling                    | 4 hours        |
| Testing                                        | 1 day          |
| Deployment & Integration with Existing Portals | 1 day          |
|                                                |                |

| **Total Estimated Time** | **7 days** |
| ------------------------ | ---------- |

# Design

## Problem Statement

**"Our company needs a unified portal for apartment booking based on employee roles, with dynamic configuration and fallback mechanisms."**

Currently, employees lack a structured way to book company apartments. We aim to create a centralized single-page application to manage apartment bookings by role and availability, with fallback options through admin intervention. The application should support drag-and-drop apartment configuration and integrate with existing portals.

## Implementation/Design

### Purpose and Usage

The application allows employees to book apartments or beds according to their designation. Role-based rules restrict booking capabilities, and when a booking cannot be fulfilled, a request system forwards queries to admins for further handling. Admins can dynamically build apartment structures and assign cities.

### Booking Rules Based on Role

| Category | Designation                | Booking Permission               | Sharing |
| -------- | -------------------------- | -------------------------------- | ------- |
| 1        | Project Engineer and below | Cottage (single bed only)        | Yes     |
| 2        | Senior till Manager        | One room only (shared apartment) | Yes     |
| 3        | Manager and above          | Entire apartment only            | No      |

- If any **cottage** is filled, show **room unavailable** for Category 2 and 3.
- If any **room/cottage** is filled in an apartment, show **apartment unavailable** to Category 3.
- When unavailable, allow users to send a **[Send Request]** to Admin for manual handling.

### Admin Features

- Add Cities.
- Add Apartments under cities.
- Drag-and-drop UI to:
  - Add Rooms to Apartments.
  - Add Cottages (beds) to Rooms.
- Configure apartment layout similar to BookMyShow theater booking style.

### Integration Points

- Integrates with existing **seat booking** and **food booking** portals.
- Uses **Microsoft Outlook login** for authentication and user metadata (name, email, role).
- User booking is based on **the person selected**, not the logged-in user.

### Data Storage Approach

PostgreSQL Database Tables:

- `Users (Id, Name, Email, Role)`
- `Cities (Id, Name)`
- `Apartments (Id, CityId, Name)`
- `Rooms (Id, ApartmentId, Name)`
- `Cottages (Id, RoomId, Name)`
- `Bookings (Id, BookedByUserId, BookingForUserId, CottageId?, RoomId?, ApartmentId, DateRange)`
- `Requests (Id, RequestedByUserId, BookingForUserId, Reason, Status, CreatedAt)`

### Booking Blocking Logic

To avoid creating redundant booking entries (e.g., filling all cottages for a Manager booking), the system stores a single booking entry at the relevant level (Apartment, Room, or Cottage) and uses dynamic logic to determine availability:

- **Apartment-level booking**: Blocks all rooms and cottages inside it.
- **Room-level booking**: Blocks that room and all its cottages.
- **Cottage-level booking**: Blocks that bed only.

This is determined dynamically when the frontend queries availability for a specific date.

### Example API: `/api/bookings/availability?date=2025-07-11`

#### Sample Response:

```json
{
  "APT_001": {
    "status": "booked",
    "level": "Apartment",
    "blockedBy": "Manager1"
  },
  "APT_002": {
    "Room_01": {
      "status": "booked",
      "level": "Room",
      "blockedBy": "Manager2"
    },
    "Room_02": {
      "Cottage_03": {
        "status": "booked",
        "level": "Cottage",
        "blockedBy": "PE1"
      }
    }
  }
}
```

#### UI Interpretation:

- If apartment is booked → entire layout disabled.
- If a room is booked → disable the room and its beds.
- If a cottage is booked → disable just that bed.
- Based on selected user’s role, UI adapts to show **unavailable** with option to **[Send Request]** if blocked.

### Sequence Diagram (Visual Not Included)

1. User logs in using Outlook SSO.
2. User selects city → apartment → structure.
3. Based on selected person's role, sees filtered availability.
4. Books a space OR sends a request if unavailable.
5. Admin receives request email and handles it via dashboard.

### UI/UX

- SPA built using React.
- Admin drag-drop UI: `react-beautiful-dnd` or similar.
- City & Apartment filter.
- Visual layout for selecting beds/rooms.
- Request button on unavailable layout.

### Non-Functional Requirements (NFRs)

- **Scalability:** Should support 1000+ users concurrently.
- **Security:** All data must be role-protected and secure using Microsoft login.
- **Performance:** Fast availability filtering based on role.
- **Modular Admin UI:** Should support future extensions (e.g., maintenance tracking).
- **Resilience:** Request system fallback when bookings are blocked.

## Design Explanation

### Why Drag-Drop UI for Admin?

- Ensures flexible apartment configuration.
- Empowers non-technical admins to structure buildings.
- Similar to BookMyShow, so intuitive for users.

### Role Enforcement at Booking Time

- Booking rules enforced not during login, but when booking for a selected user.
- Supports assistants/admins booking on behalf of others.

### Integration Justification

- Reuse existing Microsoft login and user info (avoids duplication).
- Shared authentication across seat/food/apartment portals.

## Alternative Design Consideration

**Static Apartment Model:**

- Pre-defined apartment-room-cottage layouts.
- Limited flexibility and admin control.
- Rejected due to scaling and configuration complexity.

## Summary

A role-based apartment booking portal integrated into existing office systems, with Outlook-based login, request handling, and admin-configurable layouts. Efficient, intuitive, and adaptable to future use cases.

