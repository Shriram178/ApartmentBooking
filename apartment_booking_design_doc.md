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
|     - Analytics Dashboard                      | 4 hours        |
| Testing                                        | 6 hours          |
| Deployment & Integration  | 4 hours          |



| **Total Estimated Time** | **6 days** |
| ------------------------ | ---------- |

# Design

## Problem Statement

**"Our company needs a unified portal for apartment booking based on employee roles, with dynamic configuration, analytics, and fallback mechanisms."**

Currently, employees lack a structured way to book company apartments. We aim to create a centralized single-page application to manage apartment bookings by role and availability, with fallback options through admin intervention. The application should support drag-and-drop apartment configuration, usage analytics, and integrate with existing portals.

## Implementation/Design

### Purpose and Usage

The application allows employees to book apartments or beds according to their designation. Role-based rules restrict booking capabilities, and when a booking cannot be fulfilled, a request system forwards queries to admins for further handling. Admins can dynamically build apartment structures, assign cities, and view usage analytics.

### Booking Rules Based on Role

| Category | Designation                | Booking Permission               | Sharing |
| -------- | -------------------------- | -------------------------------- | ------- |
| 1        | Project Engineer and below | Cottage (single bed only)        | Yes     |
| 2        | Senior till Manager        | One room only (shared apartment) | Yes     |
| 3        | Manager and above          | Entire apartment only            | No      |

- If any **cottage** is filled, show **room unavailable** for Category 2 and apartment unavailable for Category 3.
- If any **room or cottage** is filled in an apartment, show **apartment unavailable** to Category 3.
- When unavailable, allow users to send a **[Send Request]** to Admin for manual handling.

### Admin Features

- Add Cities.
- Add Apartments under cities.
- Drag-and-drop UI to:
  - Add Rooms to Apartments.
  - Add Cottages (beds) to Rooms.
- Configure apartment layout similar to BookMyShow theater booking style.
- View usage analytics for apartments, rooms, and cottages.

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
- `Bookings (Id, BookedByUserId, BookingForUserId, CottageId?, RoomId?, ApartmentId, DateFrom, DateTo)`
- `Requests (Id, RequestedByUserId, BookingForUserId, Reason, Status, CreatedAt)`

### Booking Blocking Logic

To avoid creating redundant booking entries (e.g., filling all cottages for a Manager booking), the system stores a single booking entry at the relevant level (Apartment, Room, or Cottage) and uses dynamic logic to determine availability:

- **Apartment-level booking**: Blocks all rooms and cottages inside it.
- **Room-level booking**: Blocks that room and all its cottages.
- **Cottage-level booking**: Blocks that bed only.

This is determined dynamically when the frontend queries availability for a specific date.

### Example API: `/api/bookings/availability?date=2025-07-11`

#### Updated Response (Nested Tree Structure):

```json
{
  "APT_001": {
    "status": "booked",
    "blockedBy": "Manager1",
    "rooms": null
  },
  "APT_002": {
    "status": null,
    "blockedBy": null,
    "rooms": {
      "Room_01": {
        "status": "booked",
        "blockedBy": "Manager2",
        "cottages": null
      },
      "Room_02": {
        "status": null,
        "blockedBy": null,
        "cottages": {
          "Cottage_03": {
            "status": "booked",
            "blockedBy": "PE1"
          }
        }
      }
    }
  }
}
```

#### Benefits of Nested Tree Structure:

- Avoids redundancy of repeating apartmentId and roomId on every row.
- Easier to map directly into nested apartment > room > cottage layout in the frontend.
- More natural fit for UI tree rendering (e.g., expandable apartment → rooms → cottages).
- Consumes less bandwidth due to hierarchical grouping.

#### Trade-offs vs Flat Structure:

| Flat Structure Advantage       | Tree Structure Advantage                  |
| ------------------------------ | ----------------------------------------- |
| Easier filtering and searching | Matches layout of apartment structure     |
| Uniform object shape           | No redundant ID repetition                |
| Quick flat table rendering     | Simpler to fill apartment layout in React |

Depending on how the frontend UI is designed (layout-first or list-first), this tree structure may be preferable.

#### Role Evaluation Logic (same as before):

- **Manager** → Show apartment unavailable if any `status` is booked at any level within that apartment.
- **Senior** → Show room unavailable if any cottage inside the room is booked.
- **PE** → Show cottage unavailable if that cottage is booked.

1. **Login:**

   - User logs in with Microsoft Outlook SSO.
   - Role is retrieved.

2. **Select Booking Dates:**

   - User is shown a calendar to choose `From` and `To` dates.

3. **Fetch Available Apartments:**

   - After selecting the dates, a list of all apartments is shown.
   - Both **available** and **unavailable** apartments are clearly marked.
   - Backend filters availability based on selected date range and target user role.

4. **Select Apartment & View Layout:**

   - User clicks on an apartment to view room/cottage layout.
   - UI disables rooms/cottages based on booking logic.

5. **Book or Request:**

   - If a valid booking space is found → Proceed with booking.
   - If all options are blocked → **[Send Request]** button enabled.

6. **Admin Handles Requests:**

   - Admin receives notification.
   - Admin assigns alternate accommodation or reassigns cancelled slots.

7. **Admin Analytics View:**

   - Admin can view usage charts (e.g., line/bar chart showing occupancy over time).
   - Filters available:
     - By City, Apartment, Room, Cottage
     - By Date Range
   - Graph shows:
     - Number of bookings
     - Number of users
     - Peak usage times

### UI/UX

- SPA built using Next.js.
- Admin drag-drop UI: `react-beautiful-dnd` or similar.
- City & Apartment filter.
- Visual layout for selecting beds/rooms.
- Calendar for date selection.
- Request button on unavailable layout.
- Admin dashboard with graphs (using Chart.js / Recharts).

### Non-Functional Requirements (NFRs)

- **Scalability:** Should support 1000+ users concurrently.
- **Security:** All data must be role-protected and secure using Microsoft login.
- **Performance:** Fast availability filtering based on role.
- **Modular Admin UI:** Should support future extensions (e.g., maintenance tracking).
- **Resilience:** Request system fallback when bookings are blocked.
- **Analytics:** Admin can analyze usage trends via charts with filters.

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

A role-based apartment booking portal integrated into existing office systems, with Outlook-based login, analytics, request handling, and admin-configurable layouts. Efficient, intuitive, and adaptable to future use cases.

