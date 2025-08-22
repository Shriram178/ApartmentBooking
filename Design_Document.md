# Design
 
## Problem statement
 
**"We need a comprehensive apartment booking system that supports individual and team accommodations with role-based allocation and administrative management"**
 
The customer requires an application to facilitate individual and team-based accommodation bookings while allowing administrators to manage and allocate accommodations efficiently. The system must support flexible date management, gender-based restrictions, role-based accommodation assignment (Manager -> Flat, Lead -> Room, Engineer -> Bed), and complex team booking logic with individual member cancellation capabilities. The application must integrate with the existing SIM tool built using React, Express, Node.js, and PostgreSQL.
 
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
 

 <img width="6232" height="5426" alt="image" src="https://github.com/user-attachments/assets/9c03f1bc-923d-4cd0-99a6-85e361d6ee32" />


 
### API Design Structure (With Request/Response Examples)

#### Authentication
**POST** `/api/auth/login`
```json
Request:
{
  "username": "john.doe",
  "password": "Secret123!"
}

Response:
{
  "token": "jwt-token",
  "user": { "id": 1, "name": "John Doe", "role": "Manager" }
}
```

#### User Management
**GET** `/api/users`
```json
Response:
[
  { "id": 1, "name": "Alice Smith", "role": "Lead", "gender": "female" },
  { "id": 2, "name": "Bob Lee", "role": "Engineer", "gender": "male" }
]
```

#### Booking
**POST** `/api/bookings`
```json
Request:
{
  "userId": 1,
  "cityId": 1,
  "bookingType": "individual",
  "BookingMembers": [
    {
      "userId": 1,
      "checkInTime": "2025-07-30T09:00:00",
      "checkOutTime": "2025-07-30T18:00:00"
    }
  ]
}

Response:
{
  "requestId": 45,
  "status": "pending",
  "createdAt": "2025-08-22T10:00:00Z"
}
```

**POST** `/api/bookings` (TEAM)
```json
Request:
{
  "userId": 1,
  "cityId": 1,
  "bookingType": "team",
  "BookingMembers": [
    {
      "userId": 1,
      "checkInTime": "2025-07-30T09:00:00",
      "checkOutTime": "2025-07-30T18:00:00"
    },
     {
      "userId": 1,
      "checkInTime": "2025-07-30T09:00:00",
      "checkOutTime": "2025-07-30T15:00:00"
    }
  ]
}

Response:
{
  "requestId": 45,
  "status": "pending",
  "createdAt": "2025-08-22T10:00:00Z"
}
```

**POST** `/api/availability/check/{cityId}`
```json
Request:
{
  "checkInTime": "2025-07-30T09:00:00",
  "checkOutTime": "2025-07-30T18:00:00"
}

Response:
{
    "success": true,
    "availability": {
        "flat": 1,
        "room": 2,
        "bed": 8
    }
}
```

## Get User Request
**POST** `/api/requests/user/{userId}`

This user is either the requester or a member of the booking request. It will return all the requests made by the user or the requests in which the user is a member.

```json

Response:
{
    "success": true,
    "data": [
        {
            "requestId": 2,
            "cityName": "Madurai",
            "bookingType": "individual",
            "requestedAt": "2025-07-28T07:31:19.944Z",
            "requestedUser": {
                "id": 1,
                "name": "Veerandra Prasath",
                "mail": "veerandra.prasath@solitontech.com",
                "role": "project engineer",
                "gender": "male"
            },
            "bookingMembers": [
                {
                    "userId": 2,
                    "username": "Shri Ram",
                    "role": "project engineer",
                    "gender": "male",
                    "mail": "shriram.saravanan@solitontech.com",
                    "checkIn": "2025-07-30T03:30:00.000Z",
                    "checkOut": "2025-07-30T12:30:00.000Z"
                }
            ]
        }
    ]
}
```
## Cancel Request
**Delete** `/api/requests/{requestId}/cancel/{userId}` 
This will remove the user from the team/individual request
```json
Response:
{
    "success": true,
    "message": "Individual request cancelled successfully"
}
```

**Delete** `/api/requests/{requestId}/cancelTeamRequest/{userId}` 
This will cancel the team request; this can only be done by the requester (requester.Id == userId).
```json
Response:
{
    "success": true,
    "message": "Team request cancelled successfully"
}
```

## Get Upcoming Stays

**POST** `/api/bookings/user/{userId}`

 - It will not include the currently staying accommodation.
 - The upcoming details are not displayed to the requester, Only members for who the stay is for will be shown in this response.

```json
Response:
{
  "success": true,
  "data": [
    {
      "requestId": 123,
      "cityName": "Madurai",
      "bookingType": "individual",
      "requestedAt": "2023-09-01T00:00:00Z",
      "requestedUser": {
        "id": 1,
        "name": "John Doe",
        "mail": "john@example.com",
        "role": "user",
        "gender": "male"
      },
      "bookingMembers": [
        {
          "userId": 1,
          "username": "John Doe",
          "checkIn": "2023-10-15T00:00:00Z",  // Future date
          "checkOut": "2023-10-20T00:00:00Z",
          "accommodation": {
        "apartment": {
          "id": 1,
          "name": "Sunrise Apartments"
        },
        "flat": {
          "id": 5,
          "name": "Flat 101"
        },
        "room": {
          "id": 10,
          "name": "Room A"
        },
        "bed": {
          "id": 15,
          "name": "Bed 1"
        }
      }
        }
      ]
    }
  ]
}
```
## User Booking History:

**GET** `/api/bookings/history/user/{userId}`
```json


Response:
{
  "success": true,
  "data": [
    {
      "requestId": 1,
      "status": "approved",
      "processedAt": "2023-10-16T10:15:00.000Z",
      "city": {
        "id": 1,
        "name": "Madurai"
      },
      "requestedAt": "2023-10-15T09:30:00.000Z",
      "bookingType": "corporate",
      "requestedUser": {
        "id": 2,
        "name": "John Doe",
        "email": "john@example.com",
        "role": "project engineer",
        "gender": "male"
      },
      "bookingMembers": [
        {
          "userId": 1,
          "name": "Jane Smith",
          "email": "jane@example.com",
          "role": "manager",
          "gender": "female",
          "checkIn": "2023-10-20T14:00:00.000Z",
          "checkOut": "2023-10-25T11:00:00.000Z",
          "accommodation": {
            "apartment": { "id": 1, "name": "Sunrise Apartments" },
            "flat": { "id": 5, "name": "Flat 5A" },
          }
        }
      ]
    }
  ]
}
```

## Admin

## Get Booking History
**POST** `/api/bookings/history?city=1&status=pending&checkIn=2025-07-30T02:35:00.000Z&checkOut=2025-07-30T12:40:00.000Z&role=project `
```json
Response:
{
    "success": true,
    "data": [
        {
            "requestId": 2,
            "city": "Madurai",
            "requestedBy": {
                "id": 1,
                "name": "Veerandra Prasath",
                "email": "veerandra.prasath@solitontech.com",
                "role": "project engineer",
                "gender": "male"
            },
            "status": "pending",
            "bookingType": "individual",
            "processedAt": null,
            "requestedAt": "2025-07-28T07:31:19.944Z",
            "bookingMembers": [
                {
                    "id": 2,
                    "name": "Shri Ram",
                    "email": "shriram.saravanan@solitontech.com",
                    "checkIn": "2025-07-30T03:30:00.000Z",
                    "checkOut": "2025-07-30T12:30:00.000Z",
                    "assignedAccommodation": {
                        "apartment": null,
                        "flat": null,
                        "room": null,
                        "bed": null
                    }
                }
            ]
        }
    ]
}
```

### Get All Pending Requests
**GET** `/api/requests`
```json
Response:
{
    "success": true,
    "data": [
        {
            "requestId": 2,
            "city": "Madurai",
            "requestedBy": {
                "userId": 1,
                "name": "Veerandra Prasath",
                "email": "veerandra.prasath@solitontech.com",
                "role": "project engineer",
                "gender": "male"
            },
            "status": "pending",
            "bookingType": "individual",
            "requestedAt": "2025-07-28T07:31:19.944Z",
            "bookingMembers": [
                {
                    "bookingMemberId": 3,
                    "name": "Shri Ram",
                    "email": "shriram.saravanan@solitontech.com",
                    "gender": "male",
                    "checkIn": "2025-07-30T03:30:00.000Z",
                    "checkOut": "2025-07-30T12:30:00.000Z"
                }
            ]
        }
    ]
}
```

## Get availability by city
**POST** `/api/availability/city/{cityId}`
```json
Request:
{
  "DATES": [
    {
      "checkIn": "2023-08-01T14:00:00Z",
      "checkOut": "2023-08-05T10:00:00Z"
    },
    {
      "checkIn": "2023-08-10T14:00:00Z",
      "checkOut": "2023-08-15T10:00:00Z"
    }
  ]
}

Response:
{
    "cityId": 1,
    "cityName": "Madurai",
    "data": [
        {
            "checkIn": "2025-08-30T09:00:00",
            "checkOut": "2025-08-30T18:00:00",
            "apartmentsStatus": [
                {
                    "id": 1,
                    "name": "Apt1",
                    "flats": [
                        {
                            "id": 1,
                            "name": "F1",
                            "isAvailable": true,
                            "gender": null,
                            "rooms": [
                                {
                                    "id": 3,
                                    "name": "R1",
                                    "isAvailable": true,
                                    "beds": [
                                        {
                                            "id": 3,
                                            "name": "Bed 1",
                                            "isAvailable": true
                                        },
                                        {
                                            "id": 4,
                                            "name": "Bed 2",
                                            "isAvailable": true
                                        },
                                        {
                                            "id": 5,
                                            "name": "Bed 3",
                                            "isAvailable": true
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": 2,
                            "name": "F2",
                            "isAvailable": true,
                            "gender": null,
                            "rooms": [
                                {
                                    "id": 4,
                                    "name": "R1",
                                    "isAvailable": true,
                                    "beds": [
                                        {
                                            "id": 6,
                                            "name": "Bed 1",
                                            "isAvailable": true
                                        },
                                        {
                                            "id": 7,
                                            "name": "Bed 2",
                                            "isAvailable": true
                                        },
                                        {
                                            "id": 8,
                                            "name": "Bed 3",
                                            "isAvailable": true
                                        }
                                    ]
                                },
                                {
                                    "id": 5,
                                    "name": "R2",
                                    "isAvailable": true,
                                    "beds": [
                                        {
                                            "id": 9,
                                            "name": "Bed 1",
                                            "isAvailable": true
                                        },
                                        {
                                            "id": 10,
                                            "name": "Bed 3",
                                            "isAvailable": true
                                        },
                                        {
                                            "id": 11,
                                            "name": "Bed 2",
                                            "isAvailable": true
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
```

## Approve Request
**POST** `/api/requests/{requestId}/approve`
```json
Request:
{
  "remarks": "Approved for engineering offsite",  // optional
  "allocatedAccommodation": [
    {
      "bookingMemberId": 789,
      "assignedAccommodation": {
        "apartmentId": 1,
        "flatId": 101,
        "roomId": 5,
        "bedId": 12
      }
    },
    {
      "bookingMemberId": 790,
      "assignedAccommodation": {
        "apartmentId": 1,
        "flatId": 101,
        "roomId": 5
      }
    }
  ]
}

Response:
{
  "success": true,
  "message": "Request approved successfully",
  "data": {
    "requestId": 456,
    "processedAt": "2023-08-15T14:30:22Z",
    "assignedCount": 2
  }
}
```

## Reject Request
**POST** `/api/requests/{requestId}/reject`
```json
Request:
{
  "remarks": "Rejected due to insufficient availability during requested dates"
}

Response:
{
  "success": true,
  "message": "Request rejected successfully",
  "data": {
    "requestId": 456,
    "processedAt": "2023-08-15T15:22:10Z",
    "remarks": "Rejected due to insufficient availability"
  }
}
```

## Get Occupancy
**POST** `/api/occupancy?city=1&apartment=1`
```json
Request:

{
      "checkIn": "2025-07-30T09:00:00Z",
      "checkOut": "2025-07-30T18:00:00Z"
}


Response:
{
    "success": true,
    "data": {
        "period": {
            "checkInTime": "2025-07-30T09:00:00Z",
            "checkOutTime": "2025-07-30T18:00:00Z"
        },
        "filtersApplied": {
            "city": "1",
            "apartment": "1",
            "status": "all"
        },
        "stats": {
            "totalBeds": 3,
            "occupied": 2,
            "vacant": 1,
            "filteredBeds": 3
        },
        "beds": [
            {
                "bed": {
                    "id": 3,
                    "name": "Bed 1",
                    "status": "occupied",
                    "occupiedBy": {
                        "user": {
                            "id": 2,
                            "name": "Shri Ram",
                            "email": "shriram.saravanan@solitontech.com",
                            "role": "project engineer",
                            "gender": "male"
                        },
                        "period": {
                            "checkInTime": "2025-07-30T09:00:00",
                            "checkOutTime": "2025-07-30T18:00:00"
                        }
                    }
                },
                "location": {
                    "room": {
                        "id": 3,
                        "name": "R1"
                    },
                    "flat": {
                        "id": 1,
                        "name": "F1"
                    },
                    "apartment": {
                        "id": 1,
                        "name": "Apt1"
                    },
                    "city": {
                        "id": 1,
                        "name": "Madurai"
                    }
                }
            },
            {
                "bed": {
                    "id": 4,
                    "name": "Bed 2",
                    "status": "vacant",
                    "occupiedBy": null
                },
                "location": {
                    "room": {
                        "id": 3,
                        "name": "R1"
                    },
                    "flat": {
                        "id": 1,
                        "name": "F1"
                    },
                    "apartment": {
                        "id": 1,
                        "name": "Apt1"
                    },
                    "city": {
                        "id": 1,
                        "name": "Madurai"
                    }
                }
            },
            {
                "bed": {
                    "id": 6,
                    "name": "Bed 1",
                    "status": "occupied",
                    "occupiedBy": {
                        "user": {
                            "id": 3,
                            "name": "shruthi",
                            "email": "shruthi.akka@solitontech.com",
                            "role": "senior project engineer",
                            "gender": "female"
                        },
                        "period": {
                            "checkInTime": "2025-07-30T09:00:00",
                            "checkOutTime": "2025-07-30T18:00:00"
                        }
                    }
                },
                "location": {
                    "room": {
                        "id": 4,
                        "name": "R1"
                    },
                    "flat": {
                        "id": 2,
                        "name": "F2"
                    },
                    "apartment": {
                        "id": 1,
                        "name": "Apt1"
                    },
                    "city": {
                        "id": 1,
                        "name": "Madurai"
                    }
                }
            }
        ]
    }
}
```
---
 
### Individual Booking Sequence Diagram
<img width="3342" height="3418" alt="image" src="https://github.com/user-attachments/assets/9ea412ae-58c9-4fd2-b4ee-d556a52a12a9" />

 
### Team Booking Sequence Diagram
<img width="3468" height="8027" alt="image" src="https://github.com/user-attachments/assets/b1902835-5e4e-4665-b1af-a16a59c252c0" />


