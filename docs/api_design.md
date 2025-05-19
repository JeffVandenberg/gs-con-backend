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

### Attendees

#### Register as Attendee
- **POST** `/attendees`
- **Description**: Register the current user as an attendee
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "registrationType": "Regular"
  }
  ```
- **Response**: 201 Created
  ```json
  {
    "attendeeId": "uuid",
    "userId": "uuid",
    "badgeNumber": "CON2023-1234",
    "registrationType": "Regular",
    "registrationDate": "2023-05-19T12:00:00Z",
    "checkedIn": false
  }
  ```

#### Get Attendee Profile
- **GET** `/attendees/me`
- **Description**: Get the current user's attendee profile
- **Authentication**: Required
- **Response**: 200 OK
  ```json
  {
    "attendeeId": "uuid",
    "userId": "uuid",
    "badgeNumber": "CON2023-1234",
    "registrationType": "Regular",
    "registrationDate": "2023-05-19T12:00:00Z",
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
    "categoryId": "uuid",
    "maxAttendees": 6,
    "status": "Scheduled"
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
    "categoryId": "uuid",
    "categoryName": "RPGs",
    "maxAttendees": 6,
    "currentAttendees": 0,
    "status": "Scheduled"
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
