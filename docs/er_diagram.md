# Entity-Relationship Diagram for Gaming Convention Database

## Overview
This document provides a visual representation of the database schema for the gaming convention web application. The diagram illustrates the relationships between the various entities defined in the database_design.md document.

## ER Diagram (Text-Based)

```
+---------------+       +----------------+       +---------------+
|  Conventions  |       |   BadgeTiers   |       |    Users      |
+---------------+       +----------------+       +---------------+
| PK: ConvID    |------>| PK: TierID     |       | PK: UserID    |
| Name          |       | FK: ConvID     |       | Email         |
| StartDate     |       | Name           |       | Password      |
| EndDate       |       | Description    |       | FirstName     |
| Location      |       | Price          |       | LastName      |
| Description   |       | MaxAvailable   |       | Role          |
| Website       |       | SoldCount      |       | CreatedAt     |
| Logo          |       | Benefits       |       | UpdatedAt     |
| Status        |       +----------------+       +---------------+
| CreatedAt     |              |                        |
| UpdatedAt     |              |                        |
+---------------+              |                        |
       |                       |                +-------v-------+
       |                       |                | SocialLogins  |
       |                       |                +---------------+
       |                       |                | PK: LoginID   |
       |                       |                | FK: UserID    |
       |                       |                | Provider      |
       |                       |                | ProviderUserID|
       |                       |                | AccessToken   |
       |                       |                | RefreshToken  |
       |                       |                | ExpiresAt     |
       |                       |                +---------------+
       |                       |
       |                       |
       |                +------v------+
       |                |   Badges    |
       |                +-------------+
       |--------------->| PK: BadgeID |<---------+
       |                | FK: UserID  |          |
       |                | FK: ConvID  |          |
       |                | FK: TierID  |          |
       |                | BadgeNumber |          |
       |                | PurchaseDate|          |
       |                | Price       |          |
       |                | Status      |          |
       |                | CheckedIn   |          |
       |                | CheckInTime |          |
       |                | AddFields   |          |
       |                +-------------+          |
       |                      |                  |
       |                      |                  |
       |                      v                  |
       |               +-------------+           |
       |               |    Event    |           |
       |               | Registrations|           |
       |               +-------------+           |
       |               | PK: RegID   |           |
       |               | FK: EventID |           |
       |               | FK: BadgeID |           |
       |               | RegTime     |           |
       |               | Status      |           |
       |               +-------------+           |
       |                      ^                  |
       |                      |                  |
       |                      |                  |
       |               +------v------+           |
       |               |   Events    |           |
       |               +-------------+           |
       |-------------->| PK: EventID |           |
       |               | FK: ConvID  |           |
       |               | Title       |           |
       |               | Description |           |
       |               | StartTime   |           |
       |               | EndTime     |           |
       |               | FK: LocID   |           |
       |               | FK: CatID   |           |
       |               | MaxAttendees|           |
       |               | CurrAttendees|          |
       |               | Status      |           |
       |               | AddFields   |           |
       |               +-------------+           |
       |                  |       |              |
       |                  |       |              |
       |                  v       v              |
       |        +-----------+  +-----------+     |
       |        | Locations |  |   Event   |     |
       |        +-----------+  | Categories|     |
       |------->| PK: LocID |  +-----------+     |
       |        | FK: ConvID|  | PK: CatID |<----+
       |        | Name      |  | FK: ConvID|
       |        | Capacity  |  | Name      |
       |        | Description| | Description|
       |        | Floor     |  +-----------+
       |        +-----------+
       |
       |
+------v------+       +----------------+       +---------------+
| GameLibraries|       | LibraryGames   |       |    Games     |
+-------------+       +----------------+       +---------------+
| PK: LibID   |------>| PK: LibGameID  |<------| PK: GameID   |
| FK: ConvID  |       | FK: LibID      |       | Title        |
| Name        |       | FK: GameID     |       | Publisher    |
| Description |       | Quantity       |       | Description  |
| Status      |       | Available      |       | MinPlayers   |
| CreatedAt   |       | Condition      |       | MaxPlayers   |
| UpdatedAt   |       +----------------+       | Duration     |
+-------------+              |                 | YearPublished|
       |                     |                 | FK: CategoryID|
       |                     |                 +---------------+
       |                     |
       |                     v
       |              +-------------+
       |              |    Game     |
       |              |  Checkouts  |
       |              +-------------+
       |              | PK: CheckoutID|
       |              | FK: LibGameID|
       |              | FK: BadgeID  |
       |              | CheckoutTime |
       |              | ReturnTime   |
       |              | FK: StaffUserID|
       |              +-------------+
       |
       |
+------v------+       +----------------+       +---------------+
|   Vendors   |       |   Sponsors     |       | Merchandise   |
+-------------+       +----------------+       +---------------+
| PK: VendorID|       | PK: SponsorID  |       | PK: MerchID   |
| FK: ConvID  |       | FK: ConvID     |       | FK: ConvID    |
| CompanyName |       | CompanyName    |       | Name          |
| ContactName |       | ContactName    |       | Description   |
| ContactEmail|       | ContactEmail   |       | Category      |
| ContactPhone|       | SponsorLevel   |       | ImageURL      |
| BoothNumber |       | Amount         |       +---------------+
| Description |       | Description    |              |
| Website     |       | Website        |              |
+-------------+       | Logo           |              |
                      +----------------+              |
                                                      v
                                               +---------------+
                                               | MerchVariations|
                                               +---------------+
                                               | PK: VarID     |
                                               | FK: MerchID   |
                                               | Description   |
                                               | Price         |
                                               | Quantity      |
                                               +---------------+
```

## Entity Relationships

### One-to-Many Relationships
- **Conventions to BadgeTiers**: One convention can have multiple badge tiers.
- **Users to SocialLogins**: One user can have multiple social login providers.
- **Users to Badges**: One user can have multiple badges (for different conventions).
- **Conventions to Badges**: One convention can have many badges issued.
- **BadgeTiers to Badges**: One badge tier can be assigned to many badges.
- **Conventions to Events**: One convention can have many events.
- **Conventions to EventCategories**: One convention can have many event categories.
- **Conventions to Locations**: One convention can have many locations.
- **Events to EventRegistrations**: One event can have many registrations.
- **Badges to EventRegistrations**: One badge can be used to register for many events.
- **EventCategories to Events**: One category can include many events.
- **Locations to Events**: One location can host many events.
- **Conventions to GameLibraries**: One convention can have multiple game libraries.
- **GameLibraries to LibraryGames**: One game library can contain many games.
- **Games to LibraryGames**: One game can be in multiple libraries.
- **LibraryGames to GameCheckouts**: One library game can have multiple checkout records over time.
- **Badges to GameCheckouts**: One badge can be used to check out multiple games.
- **Staff (Users) to GameCheckouts**: One staff member can process multiple game checkouts.
- **Conventions to Vendors**: One convention can have many vendors.
- **Conventions to Sponsors**: One convention can have many sponsors.
- **Conventions to Merchandise**: One convention can have many merchandise items.
- **Merchandise to MerchandiseVariations**: One-to-many relationship. One merchandise item can have multiple variations (sizes, colors, etc.).

### Many-to-Many Relationships
- **Users to Conventions**: Many users can attend many conventions (through Badges).
- **Games to GameLibraries**: Many games can be in many libraries (through LibraryGames).
- **Badges to Events**: Many badges can register for many events (through EventRegistrations).
- **Games to Badges**: Many games can be checked out by many badges (through GameCheckouts).

## Key Constraints

1. **Referential Integrity**:
   - A BadgeTier must reference a valid Convention
   - A SocialLogin must reference a valid User
   - A Badge must reference a valid User, Convention, and BadgeTier
   - An Event must reference a valid Convention, Location, and EventCategory
   - A Location must reference a valid Convention
   - An EventCategory must reference a valid Convention
   - An EventRegistration must reference a valid Event and Badge
   - A GameLibrary must reference a valid Convention
   - A LibraryGame must reference a valid GameLibrary and Game
   - A GameCheckout must reference a valid LibraryGame, Badge, and Staff User
   - A Vendor must reference a valid Convention
   - A Sponsor must reference a valid Convention
   - A Merchandise item must reference a valid Convention
   - A MerchandiseVariation must reference a valid Merchandise item

2. **Unique Constraints**:
   - User Email must be unique
   - Badge BadgeNumber must be unique within a Convention
   - Event Title + StartTime combination should be unique within a Convention
   - Vendor BoothNumber must be unique within a Convention
   - SocialLogin Provider + ProviderUserID combination must be unique

3. **Not Null Constraints**:
   - Critical fields like UserID, Email in Users table (Password can be null for social login only users)
   - ConventionID, Name, StartDate, EndDate in Conventions table
   - BadgeID, UserID, ConventionID, TierID, BadgeNumber in Badges table
   - EventID, ConventionID, Title, StartTime, EndTime in Events table
   - And similar critical fields in other tables

## Notes on Implementation

When implementing this schema in a specific database system:

1. All Primary Keys (PK) should be implemented as UUIDs or auto-incrementing integers
2. Foreign Keys (FK) should have appropriate ON DELETE and ON UPDATE behaviors defined
3. Timestamps should be stored in UTC format
4. Consider adding indexes on frequently queried fields
5. Implement appropriate validation at both database and application levels

This diagram provides a high-level overview of the database structure. The actual implementation may require additional junction tables or modifications based on specific requirements and the chosen database technology.
