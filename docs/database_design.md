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
- **Currency**: Currency used for this convention (USD, EUR, etc.)
- **CreatedAt**: Timestamp of creation
- **UpdatedAt**: Timestamp of last update

### ConventionDays
- **DayID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Date**: Date of this convention day
- **StartTime**: When the convention opens on this day
- **EndTime**: When the convention closes on this day
- **Description**: Additional information about this day
- **IsActive**: Boolean indicating if this day is active

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

### Friends
- **FriendshipID** (PK): Unique identifier
- **UserID** (FK): Reference to Users table (the user who initiated the friendship)
- **FriendUserID** (FK): Reference to Users table (the friend)
- **Status**: Pending, Accepted, Declined, Blocked
- **RequestDate**: When the friendship was requested
- **AcceptDate**: When the friendship was accepted (null if not accepted)
- **Notes**: Additional information about this friendship

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
- **PreviousUserID** (FK): Reference to Users table if badge was transferred
- **TransferDate**: When the badge was transferred to current user
- **AdditionalFields**: JSON object storing convention-specific custom fields

### VolunteerDays
- **VolunteerDayID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Date**: Date for volunteer shifts
- **StartTime**: When volunteer shifts start on this day
- **EndTime**: When volunteer shifts end on this day
- **Description**: Additional information about this volunteer day

### VolunteerShifts
- **ShiftID** (PK): Unique identifier
- **BadgeID** (FK): Reference to Badges table of the volunteer
- **VolunteerDayID** (FK): Reference to VolunteerDays table
- **StartTime**: When this volunteer shift starts
- **EndTime**: When this volunteer shift ends
- **Role**: Volunteer role during this shift
- **Location**: Where the volunteer is stationed
- **Notes**: Additional information about this shift
- **Status**: Scheduled, Completed, No-Show, etc.

### Events
- **EventID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Title**: Name of the event
- **Description**: Detailed description
- **StartTime**: When the event starts
- **EndTime**: When the event ends
- **LocationID** (FK): Where the event is held
- **SpaceID** (FK): Reference to Spaces table (optional)
- **CategoryID** (FK): Type of event (board game, RPG, etc.)
- **MaxAttendees**: Maximum capacity
- **CurrentAttendees**: Current number registered
- **Status**: Scheduled, Cancelled, Full, etc.
- **CreatedByUserID** (FK): Reference to Users table who created the event
- **ModifiedByUserID** (FK): Reference to Users table who last modified the event
- **IsPublic**: Boolean indicating if the event is publicly visible
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

### Spaces
- **SpaceID** (PK): Unique identifier
- **LocationID** (FK): Reference to Locations table
- **Name**: Name of the space within the location
- **Capacity**: Maximum number of people for this space
- **Description**: Additional details about the space
- **Status**: Available, Reserved, Maintenance, etc.

### EventHosts
- **HostID** (PK): Unique identifier
- **EventID** (FK): Reference to Events table
- **BadgeID** (FK): Reference to Badges table of the host
- **Role**: Host role (Main Host, Co-Host, Assistant, etc.)
- **Notes**: Additional information about this host

### EventTierRestrictions
- **RestrictionID** (PK): Unique identifier
- **EventID** (FK): Reference to Events table
- **TierID** (FK): Reference to BadgeTiers table
- **IsAllowed**: Boolean indicating if this tier is allowed to register for this event

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

### BadgeTierMerchandise
- **BadgeTierMerchandiseID** (PK): Unique identifier
- **TierID** (FK): Reference to BadgeTiers table
- **VariationID** (FK): Reference to MerchandiseVariations table
- **Quantity**: Number of this item included with the badge tier
- **IsOptional**: Boolean indicating if the attendee can choose to receive this item

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
- **Notes**: Additional information about this game in the library

### GameCheckouts
- **CheckoutID** (PK): Unique identifier
- **LibraryGameID** (FK): Reference to LibraryGames table
- **BadgeID** (FK): Who checked it out (reference to Badges table)
- **CheckoutTime**: When it was checked out
- **ReturnTime**: When it was returned (null if still out)
- **StaffUserID** (FK): Staff who processed the checkout

### Coupons
- **CouponID** (PK): Unique identifier
- **ConventionID** (FK): Reference to Conventions table
- **Code**: Unique coupon code
- **Description**: Description of the coupon
- **DiscountType**: Percentage, Fixed Amount, etc.
- **DiscountValue**: Value of the discount
- **AppliesTo**: Badges, Merchandise, Both
- **MinimumPurchase**: Minimum purchase amount required (optional)
- **MaxUses**: Maximum number of times this coupon can be used
- **UsedCount**: Number of times this coupon has been used
- **StartDate**: When the coupon becomes valid
- **EndDate**: When the coupon expires
- **IsActive**: Boolean indicating if the coupon is currently active

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

9. **Locations to Spaces**: One-to-many relationship. One location can have many spaces.

10. **Events to EventCategories**: Many-to-one relationship. Many events can belong to one category.

11. **Events to Locations**: Many-to-one relationship. Many events can be held in one location.

12. **Events to Spaces**: Many-to-one relationship. Many events can be held in one space.

13. **Events to EventRegistrations**: One-to-many relationship. One event can have many registrations.

14. **Events to EventHosts**: One-to-many relationship. One event can have multiple hosts.

15. **Badges to EventHosts**: One-to-many relationship. One badge can host multiple events.

16. **Events to EventTierRestrictions**: One-to-many relationship. One event can have restrictions for multiple badge tiers.

17. **BadgeTiers to EventTierRestrictions**: One-to-many relationship. One badge tier can be restricted from multiple events.

18. **Badges to EventRegistrations**: One-to-many relationship. One badge can be used to register for many events.

19. **Users to Events**: One-to-many relationship. One user can create or modify multiple events.

20. **Conventions to ConventionDays**: One-to-many relationship. One convention can have multiple days.

21. **Conventions to VolunteerDays**: One-to-many relationship. One convention can have multiple volunteer days.

22. **VolunteerDays to VolunteerShifts**: One-to-many relationship. One volunteer day can have multiple shifts.

23. **Badges to VolunteerShifts**: One-to-many relationship. One badge can be assigned to multiple volunteer shifts.

24. **Users to Friends**: One-to-many relationship. One user can have multiple friends.

25. **Conventions to GameLibraries**: One-to-many relationship. One convention can have multiple game libraries.

26. **GameLibraries to LibraryGames**: One-to-many relationship. One game library can contain many games.

27. **Games to LibraryGames**: One-to-many relationship. One game can be in multiple libraries.

28. **LibraryGames to GameCheckouts**: One-to-many relationship. One library game can have multiple checkout records over time.

29. **Badges to GameCheckouts**: One-to-many relationship. One badge can be used to check out multiple games.

30. **Conventions to Vendors**: One-to-many relationship. One convention can have many vendors.

31. **Conventions to Sponsors**: One-to-many relationship. One convention can have many sponsors.

32. **Conventions to Merchandise**: One-to-many relationship. One convention can have many merchandise items.

33. **Merchandise to MerchandiseVariations**: One-to-many relationship. One merchandise item can have multiple variations.

34. **BadgeTiers to BadgeTierMerchandise**: One-to-many relationship. One badge tier can include multiple merchandise items.

35. **MerchandiseVariations to BadgeTierMerchandise**: One-to-many relationship. One merchandise variation can be included in multiple badge tiers.

36. **Conventions to Coupons**: One-to-many relationship. One convention can have multiple coupons.

37. **Users to AuditLog**: One-to-many relationship. One user can have multiple audit log entries.

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

### AuditLog
- **LogID** (PK): Unique identifier
- **EntityType**: Type of entity being audited (Badge, Event, User, etc.)
- **EntityID**: ID of the entity being audited
- **Action**: Type of action (Create, Update, Delete, Transfer, etc.)
- **UserID** (FK): Reference to Users table who performed the action
- **Timestamp**: When the action occurred
- **OldValue**: Previous value (JSON format)
- **NewValue**: New value (JSON format)
- **IPAddress**: IP address of the user who performed the action
- **Notes**: Additional information about this action

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
