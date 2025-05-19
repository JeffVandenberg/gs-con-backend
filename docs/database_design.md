# Gaming Convention Database Design

## Overview
This document outlines a proposed database schema for a gaming convention web application. The design aims to support common features needed for organizing and managing multiple gaming conventions, including attendee registration, event scheduling, vendor management, and more.

The database will be implemented using PostgreSQL, which provides robust support for relational data, JSON data types (for Additional Fields), and other advanced features needed for this application.

## Key Entities

### Conventions
- **ConventionID** (PK): Unique identifier
- **Name**: Convention name
- **StartDate**: When the convention begins
- **EndDate**: When the convention ends
- **Location**: Physical location of the convention
- **Description**: Detailed description
- **Website**: Convention website
- **Logo**: Path to logo image
- **Status**: Planning, Active, Completed, Cancelled
- **CreatedAt**: Timestamp of creation
- **UpdatedAt**: Timestamp of last update

### BadgeTiers
- **TierID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Name**: Tier name (Standard, VIP, Premium, etc.)
- **Description**: Detailed description of benefits
- **Price**: Cost of this badge tier
- **MaxAvailable**: Maximum number available (null for unlimited)
- **SoldCount**: Number sold so far
- **Benefits**: List of benefits (could be JSON or separate table)

### Users
- **UserID** (PK): Unique identifier
- **Email**: User's email address (unique)
- **Password**: Hashed password (null if using social login only)
- **FirstName**: User's first name
- **LastName**: User's last name
- **Role**: Admin, Staff, Attendee, etc. (platform-wide role)
- **CreatedAt**: Timestamp of account creation
- **UpdatedAt**: Timestamp of last update

### SocialLogins
- **LoginID** (PK): Unique identifier
- **UserID** (FK): Reference to Users table
- **Provider**: Social login provider (Google, Facebook, Twitter, etc.)
- **ProviderUserID**: User ID from the provider
- **AccessToken**: Current access token (encrypted)
- **RefreshToken**: Refresh token (encrypted)
- **ExpiresAt**: When the access token expires
- **CreatedAt**: Timestamp of creation
- **UpdatedAt**: Timestamp of last update

### Badges
- **BadgeID** (PK): Unique identifier
- **UserID** (FK): Reference to Users table
- **ConventionID** (FK): Reference to Conventions table
- **TierID** (FK): Reference to BadgeTiers table
- **BadgeNumber**: Unique identifier for physical badge
- **PurchaseDate**: When the badge was purchased
- **Price**: Amount paid
- **Status**: Pending, Confirmed, Cancelled, Refunded
- **CheckedIn**: Boolean indicating if they've checked in
- **CheckInTime**: When they checked in
- **AdditionalFields**: JSON object storing convention-specific custom fields

### Events
- **EventID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Title**: Name of the event
- **Description**: Detailed description
- **StartTime**: When the event starts
- **EndTime**: When the event ends
- **LocationID** (FK): Where the event is held
- **CategoryID** (FK): Type of event (board game, RPG, etc.)
- **MaxAttendees**: Maximum capacity
- **CurrentAttendees**: Current number registered
- **Status**: Scheduled, Cancelled, Full, etc.
- **AdditionalFields**: JSON object storing convention-specific custom fields

### EventCategories
- **CategoryID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Name**: Category name (Board Games, RPGs, Card Games, etc.)
- **Description**: Category description

### Locations
- **LocationID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Name**: Room or area name
- **Capacity**: Maximum number of people
- **Description**: Additional details about the location
- **Floor**: Which floor of the venue

### EventRegistrations
- **RegistrationID** (PK): Unique identifier
- **EventID** (FK): Reference to Events table
- **BadgeID** (FK): Reference to Badges table
- **RegistrationTime**: When they registered for this event
- **Status**: Registered, Waitlisted, Cancelled, Attended

### Vendors
- **VendorID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **CompanyName**: Name of the company
- **ContactName**: Name of primary contact
- **ContactEmail**: Email of primary contact
- **ContactPhone**: Phone number
- **BoothNumber**: Assigned booth location
- **Description**: Company description
- **Website**: Company website

### Merchandise
- **MerchandiseID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Name**: Product name
- **Description**: Product description
- **Category**: Type of merchandise (T-shirt, Mug, Poster, etc.)
- **ImageURL**: Path to product image

### MerchandiseVariations
- **VariationID** (PK): Unique identifier
- **MerchandiseID** (FK): Reference to Merchandise table
- **Description**: Variation description (Large, Small, Red, Blue, etc.)
- **Price**: Cost of this variation
- **Quantity**: Number of items available for this variation

### Games
- **GameID** (PK): Unique identifier
- **Title**: Game title
- **Publisher**: Game publisher
- **Description**: Game description
- **MinPlayers**: Minimum number of players
- **MaxPlayers**: Maximum number of players
- **Duration**: Typical play time in minutes
- **YearPublished**: When the game was published
- **CategoryID** (FK): Type of game

### GameLibraries
- **LibraryID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Name**: Name of this game library
- **Description**: Description of the library
- **Status**: Active, Archived, etc.
- **CreatedAt**: Timestamp of creation
- **UpdatedAt**: Timestamp of last update

### LibraryGames
- **LibraryGameID** (PK): Unique identifier
- **LibraryID** (FK): Reference to GameLibraries table
- **GameID** (FK): Reference to Games table
- **Quantity**: Number of copies available
- **Available**: Number currently available
- **Condition**: New, Good, Fair, etc.

### GameCheckouts
- **CheckoutID** (PK): Unique identifier
- **LibraryGameID** (FK): Reference to LibraryGames table
- **BadgeID** (FK): Who checked it out (reference to Badges table)
- **CheckoutTime**: When it was checked out
- **ReturnTime**: When it was returned (null if still out)
- **StaffUserID** (FK): Staff who processed the checkout

### Sponsors
- **SponsorID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **CompanyName**: Name of the company
- **ContactName**: Name of primary contact
- **ContactEmail**: Email of primary contact
- **SponsorshipLevel**: Platinum, Gold, Silver, etc.
- **Amount**: Monetary contribution
- **Description**: Company description
- **Website**: Company website
- **Logo**: Path to logo image

## Relationships

1. **Conventions to BadgeTiers**: One-to-many relationship. One convention can have multiple badge tiers.

2. **Users to SocialLogins**: One-to-many relationship. One user can have multiple social login providers.

3. **Users to Badges**: One-to-many relationship. One user can have multiple badges (for different conventions).

4. **Conventions to Badges**: One-to-many relationship. One convention can have many badges issued.

5. **BadgeTiers to Badges**: One-to-many relationship. One badge tier can be assigned to many badges.

6. **Conventions to Events**: One-to-many relationship. One convention can have many events.

7. **Conventions to EventCategories**: One-to-many relationship. One convention can have many event categories.

8. **Conventions to Locations**: One-to-many relationship. One convention can have many locations.

9. **Events to EventCategories**: Many-to-one relationship. Many events can belong to one category.

10. **Events to Locations**: Many-to-one relationship. Many events can be held in one location.

11. **Events to EventRegistrations**: One-to-many relationship. One event can have many registrations.

12. **Badges to EventRegistrations**: One-to-many relationship. One badge can be used to register for many events.

13. **Conventions to GameLibraries**: One-to-many relationship. One convention can have multiple game libraries.

14. **GameLibraries to LibraryGames**: One-to-many relationship. One game library can contain many games.

15. **Games to LibraryGames**: One-to-many relationship. One game can be in multiple libraries.

16. **LibraryGames to GameCheckouts**: One-to-many relationship. One library game can have multiple checkout records over time.

17. **Badges to GameCheckouts**: One-to-many relationship. One badge can be used to check out multiple games.

18. **Conventions to Vendors**: One-to-many relationship. One convention can have many vendors.

19. **Conventions to Sponsors**: One-to-many relationship. One convention can have many sponsors.

20. **Conventions to Merchandise**: One-to-many relationship. One convention can have many merchandise items.

21. **Merchandise to MerchandiseVariations**: One-to-many relationship. One merchandise item can have multiple variations.

## Additional Considerations

### Authentication and Authorization
- JWT-based authentication
- Role-based access control
- Session management

### Data Validation
- Email verification
- Registration limits
- Capacity constraints

### Flexible Data with Additional Fields
- JSON-based additional fields for Badges and Events
- Convention-specific custom fields without schema changes
- Field configuration managed at the convention level
- PostgreSQL's JSONB type for efficient querying and indexing

### Reporting
- Attendance metrics
- Popular events
- Financial tracking

### Future Extensions
- Payment processing
- Feedback and ratings
- Social features

## Next Steps
This initial schema provides a foundation for the gaming convention application. The next steps would be to:

1. Review and refine the schema based on specific requirements
2. Determine the technology stack (SQL vs NoSQL, specific database system)
3. Create migration scripts or ORM models
4. Implement API endpoints for CRUD operations
