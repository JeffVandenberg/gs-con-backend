# Gaming Convention Web Application - Project Summary

## Project Overview
This project aims to develop a comprehensive web application for managing gaming conventions. The application will handle attendee registration, event scheduling, game library management, vendor coordination, and other aspects of running a successful gaming convention.

## Key Documents

The following documents have been created to guide the development of this application:

1. **Database Design** (`database_design.md`): Defines the database schema, including entities, attributes, and relationships.

2. **API Design** (`api_design.md`): Outlines the REST API endpoints that will interact with the database.

3. **Entity-Relationship Diagram** (`er_diagram.md`): Provides a visual representation of the database schema and relationships.

4. **Technology Stack Recommendations** (`tech_stack.md`): Suggests appropriate technologies for implementing the application.

## Core Features

The application will support the following core features:

### Multi-Convention Support
- Management of multiple gaming conventions
- Convention-specific configuration and settings
- Convention days scheduling with operating hours
- Data partitioning by convention
- Currency configuration per convention

### User Management
- User registration and authentication
- Social login support (Google, Facebook, etc.)
- Platform-wide role-based access control (Admin, Staff, Attendee)
- User profile management
- Friend connections between users

### Badge Management
- Multiple badge tiers per convention
- Badge purchase and assignment
- Badge transfer between users
- Convention-specific check-in process
- Flexible custom fields for different conventions
- Complimentary merchandise with badge tiers

### Event Management
- Creation and scheduling of convention events
- Event categorization
- Location and space assignment
- Multiple hosts per event
- Badge tier restrictions for events
- Public/private event visibility
- Badge-based event registration
- Configurable additional fields per convention

### Volunteer Management
- Volunteer day scheduling
- Volunteer shift assignments
- Shift tracking and management

### Game Library
- Game database with detailed information
- Multiple game libraries per convention
- Game checkout system using badges
- Notes and condition tracking for library games

### Vendor and Sponsor Management
- Vendor registration and booth assignment
- Sponsor management with different sponsorship levels
- Public listing of vendors and sponsors

### Merchandise
- Convention-specific merchandise management
- Product categorization
- Support for product variations (sizes, colors, etc.)
- Per-variation inventory tracking and pricing
- Badge tier merchandise associations

### Promotions
- Coupon code generation and management
- Discount types (percentage, fixed amount)
- Restrictions by merchandise or badges
- Usage tracking and limits

### Auditing and Security
- Comprehensive audit logging
- Badge activity tracking
- User action history

## Database Schema Summary

The database design includes the following key entities:

- **Conventions**: Stores information about different gaming conventions with currency settings
- **ConventionDays**: Defines the specific days and operating hours for each convention
- **BadgeTiers**: Defines different badge levels for each convention
- **Users**: Stores user account information with platform-wide roles
- **SocialLogins**: Manages third-party authentication providers
- **Friends**: Tracks friend relationships between users
- **Badges**: Links users to conventions through purchased badges, with transfer history
- **Events**: Manages scheduled events for each convention, with creator/modifier tracking and visibility settings
- **EventCategories**: Categorizes different types of events per convention
- **Locations**: Defines event venues and rooms for each convention
- **Spaces**: Defines specific spaces within locations (tables, areas, etc.)
- **EventHosts**: Tracks multiple hosts for each event
- **EventTierRestrictions**: Defines which badge tiers can access specific events
- **EventRegistrations**: Tracks badge registrations for events
- **VolunteerDays**: Defines days for volunteer shifts
- **VolunteerShifts**: Tracks volunteer assignments with roles and times
- **Games**: Stores information about games
- **GameLibraries**: Manages collections of games for each convention
- **LibraryGames**: Links games to specific libraries with quantity, condition, and notes
- **GameCheckouts**: Tracks game borrowing by badge holders
- **Vendors**: Manages vendor information for each convention
- **Sponsors**: Tracks sponsors for each convention
- **Merchandise**: Manages convention-specific merchandise items
- **MerchandiseVariations**: Tracks variations of merchandise items with specific pricing and inventory
- **BadgeTierMerchandise**: Links badge tiers to merchandise items for complimentary items
- **Coupons**: Manages discount codes for badges and merchandise
- **AuditLog**: Tracks all significant changes to entities for security and auditing

All convention-specific data is partitioned by ConventionID to support multiple conventions within the same database.

## API Design Summary

The API follows RESTful principles and includes endpoints for:

- Authentication (registration, login, token refresh, social login)
- Convention management and configuration
- Convention days scheduling
- User management with platform-wide roles
- User friend connections
- Badge tier configuration and management
- Badge purchase, assignment, transfer, and check-in
- Badge tier merchandise associations
- Event creation, retrieval, and management per convention
- Event space assignment
- Event host management
- Event tier restrictions
- Event registration using badges
- Volunteer day and shift management
- Game database management
- Game library creation and management per convention
- Game checkout system using badges
- Vendor and sponsor management per convention
- Merchandise inventory and management
- Merchandise variations
- Coupon code generation and validation
- Audit log retrieval and analysis

All endpoints use JSON for data exchange and follow consistent patterns for error handling, pagination, and filtering. Most endpoints include the convention context, either in the URL path or as a required parameter.

## Technology Recommendations

The recommended technology stack includes:

- **Database**: PostgreSQL (selected for its robust JSONB support for Additional Fields, relational capabilities, and advanced features)
- **Backend**: PHP 8.x (selected as the preferred programming language)
- **Frontend**: React
- **Authentication**: JWT with OAuth 2.0
- **Session Management**: Redis for caching and session storage
- **Documentation**: OpenAPI (Swagger)
- **Deployment**: Docker with Kubernetes

## Implementation Approach

The implementation is recommended to be phased:

1. **Phase 1**: Core infrastructure (database, authentication, user management)
2. **Phase 2**: Event management system
3. **Phase 3**: Game library and checkout system
4. **Phase 4**: Vendor and sponsor management, reporting

## Next Steps

To move forward with this project:

1. **Review and Refine**: Review the current design documents and refine based on specific requirements.

2. **Technology Selection**: Finalize technology choices based on the recommendations.

3. **Development Environment**: Set up the development environment with the selected technologies.

4. **Database Implementation**: Create the database schema based on the design.

5. **API Implementation**: Begin implementing the API endpoints, starting with authentication and core user management.

6. **Testing Strategy**: Develop a comprehensive testing strategy for the application.

7. **Frontend Design**: Design and implement the frontend interface for the application.

8. **Deployment Planning**: Plan the deployment strategy for development, staging, and production environments.

## Conclusion

The design documents provide a solid foundation for developing a comprehensive gaming convention management application. The modular design allows for phased implementation and future expansion as needs evolve.

The next phase of the project should focus on finalizing technology choices and beginning the implementation of the core infrastructure components.
