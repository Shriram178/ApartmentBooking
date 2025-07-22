# Meeting Summary – Manager Review (22-Jul-2025)

## Action Items and Implementation Plan

### 1. Individual Check-in/Check-out for Team Requests
- **Description**: Support different check-in/check-out times for each individual in a team booking.
- **Impact**: Adds flexibility for team bookings and accommodates real-world use cases.
- **Implementation**: UI form changes and backend schema support per-user timing.

### 2. Cancelling a Request
- **Description**: Enable users to cancel their submitted booking requests before allocation.
- **Impact**: Reduces manual intervention and avoids stale requests.
- **Implementation**: Add “Cancel” button in user dashboard; update request status logic.

### 3. Show Apartment Availability on Booking Calendar
- **Description**: Display real-time availability in the calendar used for booking.
- **Impact**: Improves user decision-making and reduces unnecessary requests.
- **Implementation**: Sync calendar view with backend data based on apartment occupancy.

### 4. User Homepage: Upcoming and Past Bookings
- **Description**: Display user’s upcoming bookings and booking history on the homepage.
- **Impact**: Improves user experience and transparency.
- **Implementation**: API changes to fetch booking data; UI card/list component.

### 5. Mandatory Rejection Remarks by Admin
- **Description**: Admin must enter a reason when rejecting a request.
- **Impact**: Ensures communication clarity; reason sent via email to user.
- **Implementation**: Make "remarks" field required on rejection.

### 6. UI Optimization – Reduce White Space
- **Description**: Eliminate excessive padding/margin that leads to unnecessary scrolling.
- **Impact**: Improves efficiency and usability on all devices.
- **Implementation**: Revise spacing across major UI components and layout.

### 7. System Management Page – One-click Add Unit
- **Description**: Admin should be able to add apartments/flats/rooms/beds using a single form.
- **Impact**: Greatly reduces number of clicks; simplifies unit creation.
- **Implementation**: Unified creation form using dynamic fieldsets.

### 8. Occupancy Page – Add Date Range Filter
- **Description**: Allow admin to view occupancy for specific past/future date ranges.
- **Impact**: Supports planning and conflict resolution.
- **Implementation**: Add date-range picker and filtered query logic.

### 9. Enhanced Export with Summary Row
- **Description**: Exported occupancy CSV should include meaningful metadata (e.g., total beds, occupied, available).
- **Impact**: Provides quick insight at a glance; improves report clarity.
- **Implementation**: Modify CSV generation to append a summary row at top.

### 10. Integration with SIM Tool (Exploration)
- **Description**: Explore integration of accommodation system with the internal SIM tool.
- **Impact**: Centralizes employee-related operations.
- **Action**: Explore SIM tool to evaluate feasibility.

### 11. Theme Alignment with SIM Tool
- **Description**: Match the UI design with the SIM tool for consistency.
- **Impact**: Provides unified visual experience for internal tools.
- **Implementation**: Apply SIM theme variables to UI components (color, font, layout).