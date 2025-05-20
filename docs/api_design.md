# Gaming Convention API Design

## Overview
This document outlines the proposed REST API endpoints for a gaming convention web application. These endpoints will interact with the database schema defined in the database_design.md document.

## Base URL
All API endpoints will be prefixed with: `/api/v1`

## Authentication
- Most endpoints require authentication via JWT token
- Token should be included in the Authorization header: `Authorization: Bearer {token}`
- Authentication endpoints are available for login, registration, and token refresh

## API Endpoints

### Authentication

#### Register User
- **POST** `/auth/register`
- **Description**: Register a new user account
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "securepassword",
    "firstName": "John",
    "lastName": "Doe"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "userId": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "Attendee",
    "createdAt": "2023-05-19T12:00:00Z"
  }
  ```

#### Login
- **POST** `/auth/login`
- **Description**: Authenticate a user and receive a JWT token
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "securepassword"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "token": "jwt-token",
    "refreshToken": "refresh-token",
    "expiresIn": 3600,
    "userId": "uuid",
    "role": "Attendee"
  }
  ```

#### Social Login
- **POST** `/auth/social/{provider}`
- **Description**: Authenticate using a social login provider (Google, Facebook, etc.)
- **Request Body**:
  ```json
  {
    "accessToken": "provider-access-token",
    "userData": {
      "id": "provider-user-id",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "token": "jwt-token",
    "refreshToken": "refresh-token",
    "expiresIn": 3600,
    "userId": "uuid",
    "role": "Attendee",
    "isNewUser": false
  }
  ```

#### Refresh Token
- **POST** `/auth/refresh`
- **Description**: Get a new JWT token using a refresh token
- **Request Body**:
  ```json
  {
    "refreshToken": "refresh-token"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "token": "new-jwt-token",
    "refreshToken": "new-refresh-token",
    "expiresIn": 3600
  }
  ```

### Users

#### Get Current User
- **GET** `/users/me`
- **Description**: Get the currently authenticated user's profile
- **Authentication**: Required
- **Response**: 200 OK
  ```json
  {
    "userId": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "Attendee",
    "createdAt": "2023-05-19T12:00:00Z",
    "updatedAt": "2023-05-19T12:00:00Z"
  }
  ```

#### Update User
- **PUT** `/users/me`
- **Description**: Update the current user's profile
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "firstName": "John",
    "lastName": "Smith",
    "email": "john.smith@example.com"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "userId": "uuid",
    "email": "john.smith@example.com",
    "firstName": "John",
    "lastName": "Smith",
    "role": "Attendee",
    "updatedAt": "2023-05-19T13:00:00Z"
  }
  ```

#### Get User (Admin)
- **GET** `/users/{userId}`
- **Description**: Get a specific user's profile
- **Authentication**: Required (Admin only)
- **Response**: 200 OK
  ```json
  {
    "userId": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "Attendee",
    "createdAt": "2023-05-19T12:00:00Z",
    "updatedAt": "2023-05-19T12:00:00Z"
  }
  ```

#### List Users (Admin)
- **GET** `/users`
- **Description**: Get a list of all users
- **Authentication**: Required (Admin only)
- **Query Parameters**:
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `role`: Filter by role
  - `search`: Search by name or email
- **Response**: 200 OK
  ```json
  {
    "users": [
      {
        "userId": "uuid",
        "email": "user@example.com",
        "firstName": "John",
        "lastName": "Doe",
        "role": "Attendee"
      }
    ],
    "total": 100,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
  ```

### Friends

#### Send Friend Request
- **POST** `/friends/request/{userId}`
- **Description**: Send a friend request to another user
- **Authentication**: Required
- **Response**: 201 Created
  ```json
  {
    "friendshipId": "uuid",
    "userId": "uuid",
    "friendUserId": "uuid",
    "status": "Pending",
    "requestDate": "2023-05-19T12:00:00Z"
  }
  ```

#### Accept Friend Request
- **PUT** `/friends/accept/{friendshipId}`
- **Description**: Accept a pending friend request
- **Authentication**: Required
- **Response**: 200 OK
  ```json
  {
    "friendshipId": "uuid",
    "userId": "uuid",
    "friendUserId": "uuid",
    "status": "Accepted",
    "requestDate": "2023-05-19T12:00:00Z",
    "acceptDate": "2023-05-20T10:00:00Z"
  }
  ```

#### Decline Friend Request
- **PUT** `/friends/decline/{friendshipId}`
- **Description**: Decline a pending friend request
- **Authentication**: Required
- **Response**: 200 OK
  ```json
  {
    "friendshipId": "uuid",
    "userId": "uuid",
    "friendUserId": "uuid",
    "status": "Declined",
    "requestDate": "2023-05-19T12:00:00Z"
  }
  ```

#### Block User
- **PUT** `/friends/block/{userId}`
- **Description**: Block a user (creates or updates a friendship record)
- **Authentication**: Required
- **Response**: 200 OK
  ```json
  {
    "friendshipId": "uuid",
    "userId": "uuid",
    "friendUserId": "uuid",
    "status": "Blocked",
    "requestDate": "2023-05-19T12:00:00Z"
  }
  ```

#### List Friends
- **GET** `/friends`
- **Description**: Get a list of the current user's friends
- **Authentication**: Required
- **Query Parameters**:
  - `status`: Filter by friendship status (Pending, Accepted, Declined, Blocked)
  - `direction`: Filter by request direction (Sent, Received)
- **Response**: 200 OK
  ```json
  {
    "friends": [
      {
        "friendshipId": "uuid",
        "userId": "uuid",
        "friendUserId": "uuid",
        "friendFirstName": "Jane",
        "friendLastName": "Smith",
        "status": "Accepted",
        "requestDate": "2023-05-19T12:00:00Z",
        "acceptDate": "2023-05-20T10:00:00Z",
        "direction": "Sent"
      }
    ]
  }
  ```

#### Remove Friend
- **DELETE** `/friends/{friendshipId}`
- **Description**: Remove a friendship or cancel a pending request
- **Authentication**: Required
- **Response**: 204 No Content

### Conventions

#### List Conventions
- **GET** `/conventions`
- **Description**: Get a list of all conventions
- **Authentication**: Optional
- **Query Parameters**:
  - `status`: Filter by status (Planning, Active, Completed, Cancelled)
  - `search`: Search by name or location
  - `upcoming`: Boolean to filter for upcoming conventions only
- **Response**: 200 OK
  ```json
  {
    "conventions": [
      {
        "conventionId": "uuid",
        "name": "GameCon 2023",
        "startDate": "2023-06-15",
        "endDate": "2023-06-18",
        "location": "Convention Center",
        "status": "Active",
        "currency": "USD"
      }
    ],
    "total": 5,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
  ```

#### Get Convention
- **GET** `/conventions/{conventionId}`
- **Description**: Get details for a specific convention
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "conventionId": "uuid",
    "name": "GameCon 2023",
    "startDate": "2023-06-15",
    "endDate": "2023-06-18",
    "location": "Convention Center",
    "description": "The premier gaming convention",
    "website": "https://gamecon2023.example.com",
    "logo": "/images/conventions/gamecon2023.png",
    "status": "Active",
    "currency": "USD",
    "createdAt": "2023-01-15T12:00:00Z",
    "updatedAt": "2023-05-01T10:30:00Z",
    "days": [
      {
        "dayId": "uuid",
        "date": "2023-06-15",
        "startTime": "09:00:00",
        "endTime": "22:00:00",
        "description": "Opening day"
      }
    ]
  }
  ```

#### Create Convention (Admin)
- **POST** `/conventions`
- **Description**: Create a new convention
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "GameCon 2024",
    "startDate": "2024-06-20",
    "endDate": "2024-06-23",
    "location": "Convention Center",
    "description": "The premier gaming convention",
    "website": "https://gamecon2024.example.com",
    "status": "Planning",
    "currency": "USD"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "conventionId": "uuid",
    "name": "GameCon 2024",
    "startDate": "2024-06-20",
    "endDate": "2024-06-23",
    "location": "Convention Center",
    "description": "The premier gaming convention",
    "website": "https://gamecon2024.example.com",
    "status": "Planning",
    "currency": "USD",
    "createdAt": "2023-07-01T12:00:00Z"
  }
  ```

#### Update Convention (Admin)
- **PUT** `/conventions/{conventionId}`
- **Description**: Update an existing convention
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "GameCon 2024 - Expanded",
    "startDate": "2024-06-20",
    "endDate": "2024-06-24",
    "location": "Convention Center",
    "description": "The premier gaming convention - Now with an extra day!",
    "website": "https://gamecon2024.example.com",
    "status": "Planning",
    "currency": "USD"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "conventionId": "uuid",
    "name": "GameCon 2024 - Expanded",
    "startDate": "2024-06-20",
    "endDate": "2024-06-24",
    "location": "Convention Center",
    "description": "The premier gaming convention - Now with an extra day!",
    "website": "https://gamecon2024.example.com",
    "status": "Planning",
    "currency": "USD",
    "updatedAt": "2023-07-15T14:30:00Z"
  }
  ```

### Convention Days

#### List Convention Days
- **GET** `/conventions/{conventionId}/days`
- **Description**: Get a list of all days for a specific convention
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "days": [
      {
        "dayId": "uuid",
        "date": "2023-06-15",
        "startTime": "09:00:00",
        "endTime": "22:00:00",
        "description": "Opening day",
        "isActive": true
      }
    ]
  }
  ```

#### Create Convention Day (Admin)
- **POST** `/conventions/{conventionId}/days`
- **Description**: Add a new day to a convention
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "date": "2023-06-16",
    "startTime": "09:00:00",
    "endTime": "22:00:00",
    "description": "Main event day",
    "isActive": true
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "dayId": "uuid",
    "conventionId": "uuid",
    "date": "2023-06-16",
    "startTime": "09:00:00",
    "endTime": "22:00:00",
    "description": "Main event day",
    "isActive": true
  }
  ```

#### Update Convention Day (Admin)
- **PUT** `/conventions/{conventionId}/days/{dayId}`
- **Description**: Update a convention day
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "startTime": "10:00:00",
    "endTime": "23:00:00",
    "description": "Main event day - Extended hours",
    "isActive": true
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "dayId": "uuid",
    "conventionId": "uuid",
    "date": "2023-06-16",
    "startTime": "10:00:00",
    "endTime": "23:00:00",
    "description": "Main event day - Extended hours",
    "isActive": true
  }
  ```

#### Delete Convention Day (Admin)
- **DELETE** `/conventions/{conventionId}/days/{dayId}`
- **Description**: Remove a day from a convention
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Badges

#### Purchase Badge
- **POST** `/conventions/{conventionId}/badges`
- **Description**: Purchase a badge for a convention
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "tierId": "uuid",
    "additionalFields": {
      "shirtSize": "XL",
      "dietaryRestrictions": "Vegetarian",
      "emergencyContact": "Jane Doe, 555-123-4567"
    }
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "badgeId": "uuid",
    "userId": "uuid",
    "conventionId": "uuid",
    "tierId": "uuid",
    "tierName": "VIP",
    "badgeNumber": "GAMECON2023-1234",
    "purchaseDate": "2023-05-19T12:00:00Z",
    "price": 75.00,
    "status": "Confirmed",
    "checkedIn": false,
    "additionalFields": {
      "shirtSize": "XL",
      "dietaryRestrictions": "Vegetarian",
      "emergencyContact": "Jane Doe, 555-123-4567"
    }
  }
  ```

#### Get My Badges
- **GET** `/badges/me`
- **Description**: Get all badges for the current user across all conventions
- **Authentication**: Required
- **Query Parameters**:
  - `conventionId`: Filter by convention
  - `status`: Filter by badge status
- **Response**: 200 OK
  ```json
  {
    "badges": [
      {
        "badgeId": "uuid",
        "conventionId": "uuid",
        "conventionName": "GameCon 2023",
        "tierId": "uuid",
        "tierName": "VIP",
        "badgeNumber": "GAMECON2023-1234",
        "purchaseDate": "2023-05-19T12:00:00Z",
        "price": 75.00,
        "status": "Confirmed",
        "checkedIn": false,
        "registeredEvents": [
          {
            "eventId": "uuid",
            "title": "Dungeons & Dragons Session",
            "startTime": "2023-06-15T14:00:00Z",
            "endTime": "2023-06-15T18:00:00Z",
            "status": "Registered"
          }
        ]
      }
    ]
  }
  ```

#### Get Badge
- **GET** `/badges/{badgeId}`
- **Description**: Get details for a specific badge
- **Authentication**: Required (Badge owner or Admin/Staff)
- **Response**: 200 OK
  ```json
  {
    "badgeId": "uuid",
    "userId": "uuid",
    "userName": "John Doe",
    "conventionId": "uuid",
    "conventionName": "GameCon 2023",
    "tierId": "uuid",
    "tierName": "VIP",
    "badgeNumber": "GAMECON2023-1234",
    "purchaseDate": "2023-05-19T12:00:00Z",
    "price": 75.00,
    "status": "Confirmed",
    "checkedIn": false,
    "checkInTime": null,
    "previousUserId": null,
    "transferDate": null,
    "additionalFields": {
      "shirtSize": "XL",
      "dietaryRestrictions": "Vegetarian",
      "emergencyContact": "Jane Doe, 555-123-4567"
    }
  }
  ```

#### Transfer Badge
- **POST** `/badges/{badgeId}/transfer`
- **Description**: Transfer a badge to another user
- **Authentication**: Required (Badge owner)
- **Request Body**:
  ```json
  {
    "recipientEmail": "jane.smith@example.com",
    "notes": "Transferring my badge as I can't attend"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "badgeId": "uuid",
    "badgeNumber": "GAMECON2023-1234",
    "previousUserId": "uuid",
    "previousUserName": "John Doe",
    "newUserId": "uuid",
    "newUserName": "Jane Smith",
    "transferDate": "2023-06-01T12:00:00Z",
    "status": "Confirmed"
  }
  ```

#### Check In Badge
- **POST** `/badges/{badgeId}/check-in`
- **Description**: Check in a badge at the convention
- **Authentication**: Required (Staff or Admin)
- **Response**: 200 OK
  ```json
  {
    "badgeId": "uuid",
    "badgeNumber": "GAMECON2023-1234",
    "userName": "John Doe",
    "checkedIn": true,
    "checkInTime": "2023-06-15T09:30:00Z"
  }
  ```

#### Check In Attendee
- **POST** `/attendees/{attendeeId}/check-in`
- **Description**: Check in an attendee at the convention
- **Authentication**: Required (Staff or Admin)
- **Response**: 200 OK
  ```json
  {
    "attendeeId": "uuid",
    "badgeNumber": "CON2023-1234",
    "checkedIn": true,
    "checkInTime": "2023-06-15T09:30:00Z"
  }
  ```

#### List Attendees (Admin/Staff)
- **GET** `/attendees`
- **Description**: Get a list of all attendees
- **Authentication**: Required (Admin or Staff)
- **Query Parameters**:
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `registrationType`: Filter by registration type
  - `checkedIn`: Filter by check-in status
  - `search`: Search by name or badge number
- **Response**: 200 OK
  ```json
  {
    "attendees": [
      {
        "attendeeId": "uuid",
        "badgeNumber": "CON2023-1234",
        "firstName": "John",
        "lastName": "Doe",
        "registrationType": "Regular",
        "checkedIn": true
      }
    ],
    "total": 500,
    "page": 1,
    "limit": 20,
    "pages": 25
  }
  ```

### Events

#### Create Event (Admin)
- **POST** `/events`
- **Description**: Create a new event
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "title": "Dungeons & Dragons Session",
    "description": "A beginner-friendly D&D session",
    "startTime": "2023-06-15T14:00:00Z",
    "endTime": "2023-06-15T18:00:00Z",
    "locationId": "uuid",
    "spaceId": "uuid",
    "categoryId": "uuid",
    "maxAttendees": 6,
    "status": "Scheduled",
    "isPublic": true,
    "additionalFields": {
      "difficultyLevel": "Beginner",
      "materialsProvided": true,
      "ageRestriction": "13+"
    }
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "eventId": "uuid",
    "title": "Dungeons & Dragons Session",
    "description": "A beginner-friendly D&D session",
    "startTime": "2023-06-15T14:00:00Z",
    "endTime": "2023-06-15T18:00:00Z",
    "locationId": "uuid",
    "locationName": "Room 101",
    "spaceId": "uuid",
    "spaceName": "Table 1",
    "categoryId": "uuid",
    "categoryName": "RPGs",
    "maxAttendees": 6,
    "currentAttendees": 0,
    "status": "Scheduled",
    "createdByUserId": "uuid",
    "createdByUserName": "John Doe",
    "isPublic": true,
    "additionalFields": {
      "difficultyLevel": "Beginner",
      "materialsProvided": true,
      "ageRestriction": "13+"
    }
  }
  ```

#### Get Event
- **GET** `/events/{eventId}`
- **Description**: Get details for a specific event
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "eventId": "uuid",
    "title": "Dungeons & Dragons Session",
    "description": "A beginner-friendly D&D session",
    "startTime": "2023-06-15T14:00:00Z",
    "endTime": "2023-06-15T18:00:00Z",
    "locationId": "uuid",
    "locationName": "Room 101",
    "categoryId": "uuid",
    "categoryName": "RPGs",
    "maxAttendees": 6,
    "currentAttendees": 3,
    "status": "Scheduled",
    "isRegistered": false,
    "canRegister": true
  }
  ```

#### List Events
- **GET** `/events`
- **Description**: Get a list of all events
- **Authentication**: Optional
- **Query Parameters**:
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `categoryId`: Filter by category
  - `startDate`: Filter by start date
  - `endDate`: Filter by end date
  - `status`: Filter by status
  - `search`: Search by title or description
- **Response**: 200 OK
  ```json
  {
    "events": [
      {
        "eventId": "uuid",
        "title": "Dungeons & Dragons Session",
        "startTime": "2023-06-15T14:00:00Z",
        "endTime": "2023-06-15T18:00:00Z",
        "locationName": "Room 101",
        "categoryName": "RPGs",
        "maxAttendees": 6,
        "currentAttendees": 3,
        "status": "Scheduled"
      }
    ],
    "total": 50,
    "page": 1,
    "limit": 20,
    "pages": 3
  }
  ```

#### Update Event (Admin)
- **PUT** `/events/{eventId}`
- **Description**: Update an existing event
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "title": "Advanced Dungeons & Dragons Session",
    "description": "An advanced D&D session for experienced players",
    "startTime": "2023-06-15T14:00:00Z",
    "endTime": "2023-06-15T18:00:00Z",
    "locationId": "uuid",
    "categoryId": "uuid",
    "maxAttendees": 6,
    "status": "Scheduled"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "eventId": "uuid",
    "title": "Advanced Dungeons & Dragons Session",
    "description": "An advanced D&D session for experienced players",
    "startTime": "2023-06-15T14:00:00Z",
    "endTime": "2023-06-15T18:00:00Z",
    "locationId": "uuid",
    "locationName": "Room 101",
    "categoryId": "uuid",
    "categoryName": "RPGs",
    "maxAttendees": 6,
    "currentAttendees": 3,
    "status": "Scheduled"
  }
  ```

#### Delete Event (Admin)
- **DELETE** `/events/{eventId}`
- **Description**: Delete an event
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Event Hosts

#### List Event Hosts
- **GET** `/events/{eventId}/hosts`
- **Description**: Get a list of all hosts for an event
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "hosts": [
      {
        "hostId": "uuid",
        "badgeId": "uuid",
        "badgeNumber": "GAMECON2023-1234",
        "userName": "John Doe",
        "role": "Main Host",
        "notes": "Experienced DM"
      },
      {
        "hostId": "uuid",
        "badgeId": "uuid",
        "badgeNumber": "GAMECON2023-5678",
        "userName": "Jane Smith",
        "role": "Co-Host",
        "notes": "Rules expert"
      }
    ]
  }
  ```

#### Add Event Host
- **POST** `/events/{eventId}/hosts`
- **Description**: Add a host to an event
- **Authentication**: Required (Event creator or Admin)
- **Request Body**:
  ```json
  {
    "badgeId": "uuid",
    "role": "Co-Host",
    "notes": "Rules expert"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "hostId": "uuid",
    "eventId": "uuid",
    "badgeId": "uuid",
    "badgeNumber": "GAMECON2023-5678",
    "userName": "Jane Smith",
    "role": "Co-Host",
    "notes": "Rules expert"
  }
  ```

#### Update Event Host
- **PUT** `/events/{eventId}/hosts/{hostId}`
- **Description**: Update a host's role or notes
- **Authentication**: Required (Event creator or Admin)
- **Request Body**:
  ```json
  {
    "role": "Assistant",
    "notes": "Helping with setup and rules"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "hostId": "uuid",
    "eventId": "uuid",
    "badgeId": "uuid",
    "badgeNumber": "GAMECON2023-5678",
    "userName": "Jane Smith",
    "role": "Assistant",
    "notes": "Helping with setup and rules"
  }
  ```

#### Remove Event Host
- **DELETE** `/events/{eventId}/hosts/{hostId}`
- **Description**: Remove a host from an event
- **Authentication**: Required (Event creator or Admin)
- **Response**: 204 No Content

### Event Tier Restrictions

#### List Event Tier Restrictions
- **GET** `/events/{eventId}/tier-restrictions`
- **Description**: Get a list of badge tier restrictions for an event
- **Authentication**: Required (Event creator or Admin)
- **Response**: 200 OK
  ```json
  {
    "restrictions": [
      {
        "restrictionId": "uuid",
        "tierId": "uuid",
        "tierName": "Standard",
        "isAllowed": false
      },
      {
        "restrictionId": "uuid",
        "tierId": "uuid",
        "tierName": "VIP",
        "isAllowed": true
      }
    ]
  }
  ```

#### Add Event Tier Restriction
- **POST** `/events/{eventId}/tier-restrictions`
- **Description**: Add a badge tier restriction to an event
- **Authentication**: Required (Event creator or Admin)
- **Request Body**:
  ```json
  {
    "tierId": "uuid",
    "isAllowed": false
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "restrictionId": "uuid",
    "eventId": "uuid",
    "tierId": "uuid",
    "tierName": "Standard",
    "isAllowed": false
  }
  ```

#### Update Event Tier Restriction
- **PUT** `/events/{eventId}/tier-restrictions/{restrictionId}`
- **Description**: Update a badge tier restriction
- **Authentication**: Required (Event creator or Admin)
- **Request Body**:
  ```json
  {
    "isAllowed": true
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "restrictionId": "uuid",
    "eventId": "uuid",
    "tierId": "uuid",
    "tierName": "Standard",
    "isAllowed": true
  }
  ```

#### Remove Event Tier Restriction
- **DELETE** `/events/{eventId}/tier-restrictions/{restrictionId}`
- **Description**: Remove a badge tier restriction from an event
- **Authentication**: Required (Event creator or Admin)
- **Response**: 204 No Content

### Event Registrations

#### Register for Event
- **POST** `/events/{eventId}/register`
- **Description**: Register the current user for an event
- **Authentication**: Required
- **Response**: 201 Created
  ```json
  {
    "registrationId": "uuid",
    "eventId": "uuid",
    "attendeeId": "uuid",
    "registrationTime": "2023-05-19T12:00:00Z",
    "status": "Registered"
  }
  ```

#### Cancel Registration
- **DELETE** `/events/{eventId}/register`
- **Description**: Cancel the current user's registration for an event
- **Authentication**: Required
- **Response**: 204 No Content

#### List My Registrations
- **GET** `/event-registrations/me`
- **Description**: Get a list of all events the current user is registered for
- **Authentication**: Required
- **Query Parameters**:
  - `status`: Filter by registration status
  - `upcoming`: Boolean to filter for upcoming events only
- **Response**: 200 OK
  ```json
  {
    "registrations": [
      {
        "registrationId": "uuid",
        "eventId": "uuid",
        "title": "Dungeons & Dragons Session",
        "startTime": "2023-06-15T14:00:00Z",
        "endTime": "2023-06-15T18:00:00Z",
        "locationName": "Room 101",
        "status": "Registered"
      }
    ]
  }
  ```

#### List Event Registrations (Admin/Staff)
- **GET** `/events/{eventId}/registrations`
- **Description**: Get a list of all attendees registered for an event
- **Authentication**: Required (Admin or Staff)
- **Response**: 200 OK
  ```json
  {
    "registrations": [
      {
        "registrationId": "uuid",
        "attendeeId": "uuid",
        "badgeNumber": "CON2023-1234",
        "firstName": "John",
        "lastName": "Doe",
        "registrationTime": "2023-05-19T12:00:00Z",
        "status": "Registered"
      }
    ],
    "total": 6
  }
  ```

### Event Categories

#### List Categories
- **GET** `/event-categories`
- **Description**: Get a list of all event categories
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "categories": [
      {
        "categoryId": "uuid",
        "name": "RPGs",
        "description": "Role-playing games"
      },
      {
        "categoryId": "uuid",
        "name": "Board Games",
        "description": "Modern board games"
      }
    ]
  }
  ```

#### Create Category (Admin)
- **POST** `/event-categories`
- **Description**: Create a new event category
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "Card Games",
    "description": "Trading card games and other card-based games"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "categoryId": "uuid",
    "name": "Card Games",
    "description": "Trading card games and other card-based games"
  }
  ```

### Locations

#### List Locations
- **GET** `/locations`
- **Description**: Get a list of all locations
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "locations": [
      {
        "locationId": "uuid",
        "name": "Room 101",
        "capacity": 30,
        "description": "Small conference room",
        "floor": 1
      },
      {
        "locationId": "uuid",
        "name": "Main Hall",
        "capacity": 200,
        "description": "Large event space",
        "floor": 1
      }
    ]
  }
  ```

#### Create Location (Admin)
- **POST** `/locations`
- **Description**: Create a new location
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "Room 102",
    "capacity": 25,
    "description": "Small conference room",
    "floor": 1
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "locationId": "uuid",
    "name": "Room 102",
    "capacity": 25,
    "description": "Small conference room",
    "floor": 1
  }
  ```

### Spaces

#### List Spaces
- **GET** `/locations/{locationId}/spaces`
- **Description**: Get a list of all spaces within a location
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "spaces": [
      {
        "spaceId": "uuid",
        "name": "Table 1",
        "capacity": 6,
        "description": "Gaming table near the window",
        "status": "Available"
      },
      {
        "spaceId": "uuid",
        "name": "Table 2",
        "capacity": 8,
        "description": "Large gaming table in the center",
        "status": "Reserved"
      }
    ]
  }
  ```

#### Get Space
- **GET** `/spaces/{spaceId}`
- **Description**: Get details for a specific space
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "spaceId": "uuid",
    "locationId": "uuid",
    "locationName": "Room 102",
    "name": "Table 1",
    "capacity": 6,
    "description": "Gaming table near the window",
    "status": "Available"
  }
  ```

#### Create Space (Admin)
- **POST** `/locations/{locationId}/spaces`
- **Description**: Create a new space within a location
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "Table 3",
    "capacity": 4,
    "description": "Small gaming table in the corner",
    "status": "Available"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "spaceId": "uuid",
    "locationId": "uuid",
    "name": "Table 3",
    "capacity": 4,
    "description": "Small gaming table in the corner",
    "status": "Available"
  }
  ```

#### Update Space (Admin)
- **PUT** `/spaces/{spaceId}`
- **Description**: Update an existing space
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "Table 3 - Premium",
    "capacity": 4,
    "description": "Premium gaming table in the corner",
    "status": "Reserved"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "spaceId": "uuid",
    "locationId": "uuid",
    "name": "Table 3 - Premium",
    "capacity": 4,
    "description": "Premium gaming table in the corner",
    "status": "Reserved"
  }
  ```

#### Delete Space (Admin)
- **DELETE** `/spaces/{spaceId}`
- **Description**: Delete a space
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Vendors

#### List Vendors
- **GET** `/vendors`
- **Description**: Get a list of all vendors
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "vendors": [
      {
        "vendorId": "uuid",
        "companyName": "Dice Emporium",
        "boothNumber": "A1",
        "description": "Specialty dice retailer",
        "website": "https://example.com"
      }
    ],
    "total": 20,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
  ```

#### Create Vendor (Admin)
- **POST** `/vendors`
- **Description**: Create a new vendor
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "companyName": "Board Game Central",
    "contactName": "Jane Smith",
    "contactEmail": "jane@example.com",
    "contactPhone": "555-123-4567",
    "boothNumber": "B2",
    "description": "Board game retailer",
    "website": "https://example.com"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "vendorId": "uuid",
    "companyName": "Board Game Central",
    "contactName": "Jane Smith",
    "contactEmail": "jane@example.com",
    "contactPhone": "555-123-4567",
    "boothNumber": "B2",
    "description": "Board game retailer",
    "website": "https://example.com"
  }
  ```

### Games

#### List Games
- **GET** `/games`
- **Description**: Get a list of all games
- **Authentication**: Optional
- **Query Parameters**:
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `categoryId`: Filter by category
  - `search`: Search by title or publisher
  - `minPlayers`: Filter by minimum players
  - `maxPlayers`: Filter by maximum players
- **Response**: 200 OK
  ```json
  {
    "games": [
      {
        "gameId": "uuid",
        "title": "Catan",
        "publisher": "Kosmos",
        "minPlayers": 3,
        "maxPlayers": 4,
        "duration": 90,
        "yearPublished": 1995,
        "categoryName": "Board Games"
      }
    ],
    "total": 100,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
  ```

#### Get Game
- **GET** `/games/{gameId}`
- **Description**: Get details for a specific game
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "gameId": "uuid",
    "title": "Catan",
    "publisher": "Kosmos",
    "description": "In Catan, players try to be the dominant force on the island of Catan by building settlements, cities, and roads.",
    "minPlayers": 3,
    "maxPlayers": 4,
    "duration": 90,
    "yearPublished": 1995,
    "categoryId": "uuid",
    "categoryName": "Board Games",
    "libraryItems": [
      {
        "libraryItemId": "uuid",
        "quantity": 3,
        "available": 2,
        "condition": "Good"
      }
    ]
  }
  ```

#### Create Game (Admin)
- **POST** `/games`
- **Description**: Add a new game to the database
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "title": "Ticket to Ride",
    "publisher": "Days of Wonder",
    "description": "A cross-country train adventure game",
    "minPlayers": 2,
    "maxPlayers": 5,
    "duration": 60,
    "yearPublished": 2004,
    "categoryId": "uuid"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "gameId": "uuid",
    "title": "Ticket to Ride",
    "publisher": "Days of Wonder",
    "description": "A cross-country train adventure game",
    "minPlayers": 2,
    "maxPlayers": 5,
    "duration": 60,
    "yearPublished": 2004,
    "categoryId": "uuid",
    "categoryName": "Board Games"
  }
  ```

### Game Library

#### Add Game to Library (Admin)
- **POST** `/game-library`
- **Description**: Add a game to the convention library
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "gameId": "uuid",
    "quantity": 2,
    "condition": "New"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "libraryItemId": "uuid",
    "gameId": "uuid",
    "gameTitle": "Ticket to Ride",
    "quantity": 2,
    "available": 2,
    "condition": "New"
  }
  ```

#### List Library Items
- **GET** `/game-library`
- **Description**: Get a list of all games in the library
- **Authentication**: Optional
- **Query Parameters**:
  - `available`: Filter by availability
  - `categoryId`: Filter by game category
  - `search`: Search by game title
- **Response**: 200 OK
  ```json
  {
    "items": [
      {
        "libraryItemId": "uuid",
        "gameId": "uuid",
        "gameTitle": "Catan",
        "quantity": 3,
        "available": 2,
        "condition": "Good"
      }
    ],
    "total": 50,
    "page": 1,
    "limit": 20,
    "pages": 3
  }
  ```

### Game Checkouts

#### Checkout Game
- **POST** `/game-checkouts`
- **Description**: Check out a game to an attendee
- **Authentication**: Required (Staff or Admin)
- **Request Body**:
  ```json
  {
    "libraryItemId": "uuid",
    "attendeeId": "uuid"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "checkoutId": "uuid",
    "libraryItemId": "uuid",
    "gameTitle": "Catan",
    "attendeeId": "uuid",
    "badgeNumber": "CON2023-1234",
    "attendeeName": "John Doe",
    "checkoutTime": "2023-06-15T10:00:00Z",
    "returnTime": null,
    "staffUserId": "uuid",
    "staffName": "Jane Smith"
  }
  ```

#### Return Game
- **PUT** `/game-checkouts/{checkoutId}/return`
- **Description**: Mark a game as returned
- **Authentication**: Required (Staff or Admin)
- **Response**: 200 OK
  ```json
  {
    "checkoutId": "uuid",
    "libraryItemId": "uuid",
    "gameTitle": "Catan",
    "attendeeId": "uuid",
    "badgeNumber": "CON2023-1234",
    "checkoutTime": "2023-06-15T10:00:00Z",
    "returnTime": "2023-06-15T14:30:00Z",
    "staffUserId": "uuid",
    "staffName": "Jane Smith"
  }
  ```

#### List Active Checkouts
- **GET** `/game-checkouts`
- **Description**: Get a list of all active game checkouts
- **Authentication**: Required (Staff or Admin)
- **Query Parameters**:
  - `returned`: Boolean to filter by return status
  - `attendeeId`: Filter by attendee
  - `search`: Search by game title or attendee name
- **Response**: 200 OK
  ```json
  {
    "checkouts": [
      {
        "checkoutId": "uuid",
        "gameTitle": "Catan",
        "attendeeName": "John Doe",
        "badgeNumber": "CON2023-1234",
        "checkoutTime": "2023-06-15T10:00:00Z",
        "returnTime": null
      }
    ],
    "total": 10,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
  ```

### Volunteer Days

#### List Volunteer Days
- **GET** `/conventions/{conventionId}/volunteer-days`
- **Description**: Get a list of all volunteer days for a convention
- **Authentication**: Required (Staff or Admin)
- **Response**: 200 OK
  ```json
  {
    "volunteerDays": [
      {
        "volunteerDayId": "uuid",
        "date": "2023-06-15",
        "startTime": "08:00:00",
        "endTime": "20:00:00",
        "description": "Opening day volunteer shifts"
      },
      {
        "volunteerDayId": "uuid",
        "date": "2023-06-16",
        "startTime": "08:00:00",
        "endTime": "20:00:00",
        "description": "Main day volunteer shifts"
      }
    ]
  }
  ```

#### Create Volunteer Day (Admin)
- **POST** `/conventions/{conventionId}/volunteer-days`
- **Description**: Create a new volunteer day
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "date": "2023-06-17",
    "startTime": "08:00:00",
    "endTime": "20:00:00",
    "description": "Final day volunteer shifts"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "volunteerDayId": "uuid",
    "conventionId": "uuid",
    "date": "2023-06-17",
    "startTime": "08:00:00",
    "endTime": "20:00:00",
    "description": "Final day volunteer shifts"
  }
  ```

#### Update Volunteer Day (Admin)
- **PUT** `/volunteer-days/{volunteerDayId}`
- **Description**: Update a volunteer day
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "startTime": "09:00:00",
    "endTime": "21:00:00",
    "description": "Final day volunteer shifts - Extended hours"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "volunteerDayId": "uuid",
    "conventionId": "uuid",
    "date": "2023-06-17",
    "startTime": "09:00:00",
    "endTime": "21:00:00",
    "description": "Final day volunteer shifts - Extended hours"
  }
  ```

#### Delete Volunteer Day (Admin)
- **DELETE** `/volunteer-days/{volunteerDayId}`
- **Description**: Delete a volunteer day
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Volunteer Shifts

#### List Volunteer Shifts
- **GET** `/volunteer-days/{volunteerDayId}/shifts`
- **Description**: Get a list of all volunteer shifts for a day
- **Authentication**: Required (Staff or Admin)
- **Response**: 200 OK
  ```json
  {
    "shifts": [
      {
        "shiftId": "uuid",
        "badgeId": "uuid",
        "badgeNumber": "GAMECON2023-1234",
        "userName": "John Doe",
        "startTime": "08:00:00",
        "endTime": "12:00:00",
        "role": "Registration Desk",
        "location": "Main Entrance",
        "status": "Scheduled"
      }
    ]
  }
  ```

#### Create Volunteer Shift
- **POST** `/volunteer-days/{volunteerDayId}/shifts`
- **Description**: Create a new volunteer shift
- **Authentication**: Required (Staff or Admin)
- **Request Body**:
  ```json
  {
    "badgeId": "uuid",
    "startTime": "13:00:00",
    "endTime": "17:00:00",
    "role": "Game Library",
    "location": "Library Room",
    "notes": "Experienced with board games"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "shiftId": "uuid",
    "volunteerDayId": "uuid",
    "badgeId": "uuid",
    "badgeNumber": "GAMECON2023-1234",
    "userName": "John Doe",
    "startTime": "13:00:00",
    "endTime": "17:00:00",
    "role": "Game Library",
    "location": "Library Room",
    "notes": "Experienced with board games",
    "status": "Scheduled"
  }
  ```

#### Update Volunteer Shift
- **PUT** `/volunteer-shifts/{shiftId}`
- **Description**: Update a volunteer shift
- **Authentication**: Required (Staff or Admin)
- **Request Body**:
  ```json
  {
    "startTime": "14:00:00",
    "endTime": "18:00:00",
    "role": "Game Library Lead",
    "status": "Confirmed"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "shiftId": "uuid",
    "volunteerDayId": "uuid",
    "badgeId": "uuid",
    "badgeNumber": "GAMECON2023-1234",
    "userName": "John Doe",
    "startTime": "14:00:00",
    "endTime": "18:00:00",
    "role": "Game Library Lead",
    "location": "Library Room",
    "notes": "Experienced with board games",
    "status": "Confirmed"
  }
  ```

#### Delete Volunteer Shift
- **DELETE** `/volunteer-shifts/{shiftId}`
- **Description**: Delete a volunteer shift
- **Authentication**: Required (Staff or Admin)
- **Response**: 204 No Content

#### Get My Volunteer Shifts
- **GET** `/badges/{badgeId}/volunteer-shifts`
- **Description**: Get all volunteer shifts for a badge
- **Authentication**: Required (Badge owner or Staff/Admin)
- **Response**: 200 OK
  ```json
  {
    "shifts": [
      {
        "shiftId": "uuid",
        "volunteerDayId": "uuid",
        "date": "2023-06-15",
        "startTime": "08:00:00",
        "endTime": "12:00:00",
        "role": "Registration Desk",
        "location": "Main Entrance",
        "status": "Scheduled"
      }
    ]
  }
  ```

### Sponsors

#### List Sponsors
- **GET** `/sponsors`
- **Description**: Get a list of all sponsors
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "sponsors": [
      {
        "sponsorId": "uuid",
        "companyName": "Game Publisher Inc.",
        "sponsorshipLevel": "Gold",
        "description": "Major board game publisher",
        "website": "https://example.com",
        "logo": "/images/sponsors/publisher-logo.png"
      }
    ]
  }
  ```

#### Create Sponsor (Admin)
- **POST** `/sponsors`
- **Description**: Add a new sponsor
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "companyName": "Dice Manufacturer Ltd.",
    "contactName": "John Smith",
    "contactEmail": "john@example.com",
    "sponsorshipLevel": "Silver",
    "amount": 2500,
    "description": "Premium dice manufacturer",
    "website": "https://example.com"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "sponsorId": "uuid",
    "companyName": "Dice Manufacturer Ltd.",
    "contactName": "John Smith",
    "contactEmail": "john@example.com",
    "sponsorshipLevel": "Silver",
    "amount": 2500,
    "description": "Premium dice manufacturer",
    "website": "https://example.com"
  }
  ```

### Merchandise

#### List Merchandise
- **GET** `/conventions/{conventionId}/merchandise`
- **Description**: Get a list of all merchandise for a convention
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "merchandise": [
      {
        "merchandiseId": "uuid",
        "name": "Convention T-Shirt",
        "description": "Official convention t-shirt with logo",
        "category": "Apparel",
        "imageURL": "/images/merchandise/tshirt.jpg",
        "variations": [
          {
            "variationId": "uuid",
            "description": "Small, Black",
            "price": 20.00,
            "quantity": 50
          },
          {
            "variationId": "uuid",
            "description": "Medium, Black",
            "price": 20.00,
            "quantity": 75
          }
        ]
      }
    ]
  }
  ```

#### Get Merchandise Item
- **GET** `/merchandise/{merchandiseId}`
- **Description**: Get details for a specific merchandise item
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "merchandiseId": "uuid",
    "conventionId": "uuid",
    "name": "Convention T-Shirt",
    "description": "Official convention t-shirt with logo",
    "category": "Apparel",
    "imageURL": "/images/merchandise/tshirt.jpg",
    "variations": [
      {
        "variationId": "uuid",
        "description": "Small, Black",
        "price": 20.00,
        "quantity": 50
      },
      {
        "variationId": "uuid",
        "description": "Medium, Black",
        "price": 20.00,
        "quantity": 75
      }
    ]
  }
  ```

#### Create Merchandise (Admin)
- **POST** `/conventions/{conventionId}/merchandise`
- **Description**: Create a new merchandise item
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "Convention Mug",
    "description": "Ceramic mug with convention logo",
    "category": "Drinkware",
    "imageURL": "/images/merchandise/mug.jpg"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "merchandiseId": "uuid",
    "conventionId": "uuid",
    "name": "Convention Mug",
    "description": "Ceramic mug with convention logo",
    "category": "Drinkware",
    "imageURL": "/images/merchandise/mug.jpg"
  }
  ```

#### Update Merchandise (Admin)
- **PUT** `/merchandise/{merchandiseId}`
- **Description**: Update a merchandise item
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "name": "Premium Convention Mug",
    "description": "High-quality ceramic mug with convention logo",
    "category": "Drinkware",
    "imageURL": "/images/merchandise/premium-mug.jpg"
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "merchandiseId": "uuid",
    "conventionId": "uuid",
    "name": "Premium Convention Mug",
    "description": "High-quality ceramic mug with convention logo",
    "category": "Drinkware",
    "imageURL": "/images/merchandise/premium-mug.jpg"
  }
  ```

#### Delete Merchandise (Admin)
- **DELETE** `/merchandise/{merchandiseId}`
- **Description**: Delete a merchandise item
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Merchandise Variations

#### List Variations
- **GET** `/merchandise/{merchandiseId}/variations`
- **Description**: Get all variations for a merchandise item
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "variations": [
      {
        "variationId": "uuid",
        "description": "Red",
        "price": 15.00,
        "quantity": 30
      },
      {
        "variationId": "uuid",
        "description": "Blue",
        "price": 15.00,
        "quantity": 25
      }
    ]
  }
  ```

#### Create Variation (Admin)
- **POST** `/merchandise/{merchandiseId}/variations`
- **Description**: Add a variation to a merchandise item
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "description": "Green",
    "price": 15.00,
    "quantity": 20
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "variationId": "uuid",
    "merchandiseId": "uuid",
    "description": "Green",
    "price": 15.00,
    "quantity": 20
  }
  ```

#### Update Variation (Admin)
- **PUT** `/merchandise/variations/{variationId}`
- **Description**: Update a merchandise variation
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "description": "Green - Limited Edition",
    "price": 18.00,
    "quantity": 15
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "variationId": "uuid",
    "merchandiseId": "uuid",
    "description": "Green - Limited Edition",
    "price": 18.00,
    "quantity": 15
  }
  ```

#### Delete Variation (Admin)
- **DELETE** `/merchandise/variations/{variationId}`
- **Description**: Delete a merchandise variation
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Badge Tier Merchandise

#### List Badge Tier Merchandise
- **GET** `/badge-tiers/{tierId}/merchandise`
- **Description**: Get all merchandise included with a badge tier
- **Authentication**: Optional
- **Response**: 200 OK
  ```json
  {
    "merchandise": [
      {
        "badgeTierMerchandiseId": "uuid",
        "merchandiseId": "uuid",
        "merchandiseName": "Convention T-Shirt",
        "variationId": "uuid",
        "variationDescription": "Medium, Black",
        "quantity": 1,
        "isOptional": false
      },
      {
        "badgeTierMerchandiseId": "uuid",
        "merchandiseId": "uuid",
        "merchandiseName": "Convention Mug",
        "variationId": "uuid",
        "variationDescription": "Red",
        "quantity": 1,
        "isOptional": true
      }
    ]
  }
  ```

#### Add Badge Tier Merchandise (Admin)
- **POST** `/badge-tiers/{tierId}/merchandise`
- **Description**: Add merchandise to a badge tier
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "variationId": "uuid",
    "quantity": 1,
    "isOptional": true
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "badgeTierMerchandiseId": "uuid",
    "tierId": "uuid",
    "tierName": "VIP",
    "merchandiseId": "uuid",
    "merchandiseName": "Convention Poster",
    "variationId": "uuid",
    "variationDescription": "18x24 Glossy",
    "quantity": 1,
    "isOptional": true
  }
  ```

#### Update Badge Tier Merchandise (Admin)
- **PUT** `/badge-tier-merchandise/{badgeTierMerchandiseId}`
- **Description**: Update badge tier merchandise
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "quantity": 2,
    "isOptional": false
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "badgeTierMerchandiseId": "uuid",
    "tierId": "uuid",
    "tierName": "VIP",
    "merchandiseId": "uuid",
    "merchandiseName": "Convention Poster",
    "variationId": "uuid",
    "variationDescription": "18x24 Glossy",
    "quantity": 2,
    "isOptional": false
  }
  ```

#### Remove Badge Tier Merchandise (Admin)
- **DELETE** `/badge-tier-merchandise/{badgeTierMerchandiseId}`
- **Description**: Remove merchandise from a badge tier
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

### Coupons

#### List Coupons
- **GET** `/conventions/{conventionId}/coupons`
- **Description**: Get a list of all coupons for a convention
- **Authentication**: Required (Admin or Staff)
- **Query Parameters**:
  - `isActive`: Filter by active status
  - `appliesTo`: Filter by what the coupon applies to (Badges, Merchandise, Both)
- **Response**: 200 OK
  ```json
  {
    "coupons": [
      {
        "couponId": "uuid",
        "code": "EARLYBIRD2023",
        "description": "Early bird registration discount",
        "discountType": "Percentage",
        "discountValue": 15,
        "appliesTo": "Badges",
        "minimumPurchase": null,
        "maxUses": 100,
        "usedCount": 42,
        "startDate": "2023-01-01T00:00:00Z",
        "endDate": "2023-03-31T23:59:59Z",
        "isActive": true
      }
    ]
  }
  ```

#### Get Coupon
- **GET** `/coupons/{couponId}`
- **Description**: Get details for a specific coupon
- **Authentication**: Required (Admin or Staff)
- **Response**: 200 OK
  ```json
  {
    "couponId": "uuid",
    "conventionId": "uuid",
    "code": "EARLYBIRD2023",
    "description": "Early bird registration discount",
    "discountType": "Percentage",
    "discountValue": 15,
    "appliesTo": "Badges",
    "minimumPurchase": null,
    "maxUses": 100,
    "usedCount": 42,
    "startDate": "2023-01-01T00:00:00Z",
    "endDate": "2023-03-31T23:59:59Z",
    "isActive": true
  }
  ```

#### Create Coupon (Admin)
- **POST** `/conventions/{conventionId}/coupons`
- **Description**: Create a new coupon
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "code": "SUMMER2023",
    "description": "Summer promotion discount",
    "discountType": "FixedAmount",
    "discountValue": 10,
    "appliesTo": "Both",
    "minimumPurchase": 50,
    "maxUses": 200,
    "startDate": "2023-06-01T00:00:00Z",
    "endDate": "2023-08-31T23:59:59Z",
    "isActive": true
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "couponId": "uuid",
    "conventionId": "uuid",
    "code": "SUMMER2023",
    "description": "Summer promotion discount",
    "discountType": "FixedAmount",
    "discountValue": 10,
    "appliesTo": "Both",
    "minimumPurchase": 50,
    "maxUses": 200,
    "usedCount": 0,
    "startDate": "2023-06-01T00:00:00Z",
    "endDate": "2023-08-31T23:59:59Z",
    "isActive": true
  }
  ```

#### Update Coupon (Admin)
- **PUT** `/coupons/{couponId}`
- **Description**: Update a coupon
- **Authentication**: Required (Admin)
- **Request Body**:
  ```json
  {
    "description": "Extended summer promotion discount",
    "discountValue": 15,
    "endDate": "2023-09-30T23:59:59Z",
    "isActive": true
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "couponId": "uuid",
    "conventionId": "uuid",
    "code": "SUMMER2023",
    "description": "Extended summer promotion discount",
    "discountType": "FixedAmount",
    "discountValue": 15,
    "appliesTo": "Both",
    "minimumPurchase": 50,
    "maxUses": 200,
    "usedCount": 0,
    "startDate": "2023-06-01T00:00:00Z",
    "endDate": "2023-09-30T23:59:59Z",
    "isActive": true
  }
  ```

#### Delete Coupon (Admin)
- **DELETE** `/coupons/{couponId}`
- **Description**: Delete a coupon
- **Authentication**: Required (Admin)
- **Response**: 204 No Content

#### Validate Coupon
- **POST** `/coupons/validate`
- **Description**: Validate a coupon code
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "code": "SUMMER2023",
    "conventionId": "uuid",
    "appliesTo": "Badges",
    "purchaseAmount": 75
  }
  ```
- **Response**: 200 OK
  ```json
  {
    "valid": true,
    "couponId": "uuid",
    "discountType": "FixedAmount",
    "discountValue": 15,
    "discountedAmount": 60
  }
  ```

### Audit Log

#### List Audit Logs (Admin)
- **GET** `/audit-logs`
- **Description**: Get a list of audit log entries
- **Authentication**: Required (Admin)
- **Query Parameters**:
  - `entityType`: Filter by entity type (Badge, Event, User, etc.)
  - `entityId`: Filter by entity ID
  - `action`: Filter by action type (Create, Update, Delete, Transfer, etc.)
  - `userId`: Filter by user who performed the action
  - `startDate`: Filter by start date
  - `endDate`: Filter by end date
- **Response**: 200 OK
  ```json
  {
    "logs": [
      {
        "logId": "uuid",
        "entityType": "Badge",
        "entityId": "uuid",
        "action": "Transfer",
        "userId": "uuid",
        "userName": "John Doe",
        "timestamp": "2023-06-01T12:00:00Z",
        "oldValue": {
          "userId": "uuid-old-user"
        },
        "newValue": {
          "userId": "uuid-new-user",
          "transferDate": "2023-06-01T12:00:00Z"
        },
        "ipAddress": "192.168.1.1",
        "notes": "Badge transferred to another user"
      }
    ],
    "total": 150,
    "page": 1,
    "limit": 20,
    "pages": 8
  }
  ```

#### Get Audit Log Entry (Admin)
- **GET** `/audit-logs/{logId}`
- **Description**: Get details for a specific audit log entry
- **Authentication**: Required (Admin)
- **Response**: 200 OK
  ```json
  {
    "logId": "uuid",
    "entityType": "Badge",
    "entityId": "uuid",
    "action": "Transfer",
    "userId": "uuid",
    "userName": "John Doe",
    "timestamp": "2023-06-01T12:00:00Z",
    "oldValue": {
      "userId": "uuid-old-user",
      "userName": "Jane Smith"
    },
    "newValue": {
      "userId": "uuid-new-user",
      "userName": "Bob Johnson",
      "transferDate": "2023-06-01T12:00:00Z"
    },
    "ipAddress": "192.168.1.1",
    "notes": "Badge transferred to another user"
  }
  ```

## Error Handling

All endpoints follow a consistent error response format:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found",
    "details": {
      "resourceType": "Event",
      "resourceId": "invalid-uuid"
    }
  }
}
```

Common error codes:
- `UNAUTHORIZED`: Authentication required
- `FORBIDDEN`: Insufficient permissions
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `VALIDATION_ERROR`: Request validation failed
- `CONFLICT`: Request conflicts with existing data
- `INTERNAL_SERVER_ERROR`: Unexpected server error

## Pagination

Endpoints that return lists support pagination with the following query parameters:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)

Paginated responses include metadata:
```json
{
  "items": [
    {
      "id": "uuid",
      "name": "Example Item"
    }
  ],
  "total": 100,
  "page": 1,
  "limit": 20,
  "pages": 5
}
```

## Filtering and Sorting

Many list endpoints support filtering via query parameters specific to the resource type.
Common filter parameters include:
- `search`: Text search across relevant fields
- `startDate`/`endDate`: Date range filters
- Status filters specific to the resource

## Next Steps

This API design provides a foundation for the gaming convention application. The next steps would be to:

1. Review and refine the API based on specific requirements
2. Determine the technology stack for implementation (Node.js, Python, etc.)
3. Set up authentication and authorization middleware
4. Implement the API endpoints with proper validation
5. Create comprehensive API documentation with Swagger/OpenAPI
