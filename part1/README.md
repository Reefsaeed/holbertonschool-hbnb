# HBnB Evolution - Technical Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [High-Level Architecture](#high-level-architecture)
3. [Business Logic Layer](#business-logic-layer)
4. [API Interaction Flow](#api-interaction-flow)
5. [Conclusion](#conclusion)

---

## 1. Introduction

### 1.1 Project Overview
HBnB Evolution is a simplified version of an AirBnB-like application designed to manage property rentals, user interactions, and reviews. The application enables users to list properties, browse available places, leave reviews, and manage amenities associated with properties.

### 1.2 Purpose of This Document
This technical documentation serves as a comprehensive blueprint for the HBnB Evolution application. It provides detailed architectural diagrams, class structures, and interaction flows that will guide the implementation phases of the project. The document ensures a clear understanding of:
- The overall system architecture
- The relationships between different components
- The flow of data through the application
- Design decisions and their rationale

### 1.3 Document Scope
This document covers:
- **High-Level Architecture**: Three-layer architecture using the facade pattern
- **Business Logic Layer**: Detailed entity design and relationships
- **API Interaction Flow**: Sequence diagrams showing component interactions
- **Design Decisions**: Rationale behind architectural choices

---

## 2. High-Level Architecture

### 2.1 Three-Layer Architecture Overview
The HBnB Evolution application follows a layered architecture pattern, separating concerns into three distinct layers:

#### **Presentation Layer (API/Services Layer)**
- Handles HTTP requests and responses
- Provides RESTful API endpoints for client interaction
- Validates input data and formats output
- Acts as the entry point for all user interactions

#### **Business Logic Layer (Model Layer)**
- Contains core application logic and business rules
- Defines entities: User, Place, Review, and Amenity
- Implements business validations and operations
- Manages relationships between entities

#### **Persistence Layer (Database Layer)**
- Handles data storage and retrieval
- Manages database connections and transactions
- Implements data access patterns
- Ensures data integrity and consistency

### 2.2 Facade Pattern Implementation
The **Facade Pattern** is used to provide a unified interface between layers:
- Simplifies communication between the Presentation and Business Logic layers
- Reduces coupling between layers
- Provides a single entry point for business operations
- Makes the system easier to use and understand

### 2.3 High-Level Package Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  ┌────────────────────────────────────────────────────┐    │
│  │              API Services (REST API)                │    │
│  │  - User Endpoints    - Place Endpoints              │    │
│  │  - Review Endpoints  - Amenity Endpoints            │    │
│  └────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  BUSINESS LOGIC LAYER                        │
│  ┌────────────────────────────────────────────────────┐    │
│  │                  Facade Pattern                     │    │
│  │           (Unified Business Interface)              │    │
│  └────────────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────────────┐    │
│  │                  Models/Entities                    │    │
│  │  ┌──────┐  ┌───────┐  ┌────────┐  ┌─────────┐    │    │
│  │  │ User │  │ Place │  │ Review │  │ Amenity │    │    │
│  │  └──────┘  └───────┘  └────────┘  └─────────┘    │    │
│  └────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    PERSISTENCE LAYER                         │
│  ┌────────────────────────────────────────────────────┐    │
│  │            Data Access Layer (Repository)           │    │
│  │  - Database Connection  - CRUD Operations           │    │
│  │  - Query Management     - Transaction Handling      │    │
│  └────────────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────────────┐    │
│  │                     Database                        │    │
│  │             (Relational Database)                   │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 2.4 Layer Communication Flow
1. **Client Request**: User sends HTTP request to API endpoint
2. **API Processing**: Presentation layer validates and processes request
3. **Facade Routing**: Request is routed through facade to appropriate business logic
4. **Business Logic**: Entities perform business operations and validations
5. **Data Persistence**: Business logic interacts with persistence layer to store/retrieve data
6. **Response**: Data flows back through layers to client

### 2.5 Design Rationale
- **Separation of Concerns**: Each layer has a specific responsibility
- **Maintainability**: Changes in one layer don't affect others
- **Testability**: Each layer can be tested independently
- **Scalability**: Layers can be scaled separately based on needs

---

## 3. Business Logic Layer

### 3.1 Entity Design Overview
The Business Logic Layer contains four core entities that represent the domain model:

#### **Core Entities**
1. **User**: Represents system users (both regular users and administrators)
2. **Place**: Represents properties listed on the platform
3. **Review**: Represents user feedback on places
4. **Amenity**: Represents features/facilities available at places

### 3.2 Detailed Class Diagram

```
┌─────────────────────────────────────────────────────────┐
│                          User                            │
├─────────────────────────────────────────────────────────┤
│ - id: UUID                                              │
│ - first_name: String                                    │
│ - last_name: String                                     │
│ - email: String (unique)                                │
│ - password: String (hashed)                             │
│ - is_admin: Boolean                                     │
│ - created_at: DateTime                                  │
│ - updated_at: DateTime                                  │
├─────────────────────────────────────────────────────────┤
│ + register(): User                                      │
│ + update_profile(data): User                            │
│ + delete(): Boolean                                     │
│ + authenticate(password): Boolean                       │
│ + get_owned_places(): List<Place>                       │
│ + get_reviews(): List<Review>                           │
└─────────────────────────────────────────────────────────┘
                    │
                    │ 1
                    │ owns
                    │
                    │ *
                    ▼
┌─────────────────────────────────────────────────────────┐
│                         Place                            │
├─────────────────────────────────────────────────────────┤
│ - id: UUID                                              │
│ - title: String                                         │
│ - description: String                                   │
│ - price: Decimal                                        │
│ - latitude: Float                                       │
│ - longitude: Float                                      │
│ - owner_id: UUID (FK -> User)                           │
│ - created_at: DateTime                                  │
│ - updated_at: DateTime                                  │
├─────────────────────────────────────────────────────────┤
│ + create(owner, data): Place                            │
│ + update(data): Place                                   │
│ + delete(): Boolean                                     │
│ + get_reviews(): List<Review>                           │
│ + add_amenity(amenity): Boolean                         │
│ + remove_amenity(amenity): Boolean                      │
│ + get_amenities(): List<Amenity>                        │
└─────────────────────────────────────────────────────────┘
            │                               │
            │ 1                             │ *
            │ has                           │ has
            │                               │
            │ *                             │ *
            ▼                               ▼
┌─────────────────────────────┐   ┌──────────────────────────┐
│          Review             │   │     Place_Amenity        │
├─────────────────────────────┤   │   (Association Table)    │
│ - id: UUID                  │   ├──────────────────────────┤
│ - place_id: UUID (FK)       │   │ - place_id: UUID (FK)    │
│ - user_id: UUID (FK)        │   │ - amenity_id: UUID (FK)  │
│ - rating: Integer (1-5)     │   └──────────────────────────┘
│ - comment: String           │                │
│ - created_at: DateTime      │                │ *
│ - updated_at: DateTime      │                │ references
├─────────────────────────────┤                │
│ + create(user, place): Rev  │                │ *
│ + update(data): Review      │                ▼
│ + delete(): Boolean         │   ┌──────────────────────────┐
│ + validate_rating(): Bool   │   │        Amenity           │
└─────────────────────────────┘   ├──────────────────────────┤
            ▲                      │ - id: UUID               │
            │                      │ - name: String           │
            │ 1                    │ - description: String    │
            │ writes               │ - created_at: DateTime   │
            │                      │ - updated_at: DateTime   │
            │ *                    ├──────────────────────────┤
    ┌───────┘                      │ + create(data): Amenity  │
    │                              │ + update(data): Amenity  │
(User entity from above)           │ + delete(): Boolean      │
                                   │ + get_places(): List     │
                                   └──────────────────────────┘
```

### 3.3 Entity Relationships

#### **User to Place (One-to-Many)**
- One user can own multiple places
- Each place is owned by exactly one user
- Relationship attribute: `owner_id` in Place entity

#### **Place to Review (One-to-Many)**
- One place can have multiple reviews
- Each review is associated with exactly one place
- Relationship attribute: `place_id` in Review entity

#### **User to Review (One-to-Many)**
- One user can write multiple reviews
- Each review is written by exactly one user
- Relationship attribute: `user_id` in Review entity

#### **Place to Amenity (Many-to-Many)**
- One place can have multiple amenities
- One amenity can be associated with multiple places
- Implemented through `Place_Amenity` association table

### 3.4 Entity Attributes and Methods

#### **User Entity**
**Attributes:**
- `id`: Unique identifier (UUID)
- `first_name`, `last_name`: User's name
- `email`: Unique email address for authentication
- `password`: Hashed password for security
- `is_admin`: Boolean flag for administrative privileges
- `created_at`, `updated_at`: Audit timestamps

**Key Methods:**
- `register()`: Creates new user account with validation
- `update_profile()`: Updates user information
- `authenticate()`: Verifies password for login
- `get_owned_places()`: Retrieves all places owned by user

#### **Place Entity**
**Attributes:**
- `id`: Unique identifier (UUID)
- `title`, `description`: Place information
- `price`: Rental price per night
- `latitude`, `longitude`: Geographic coordinates
- `owner_id`: Reference to owning user
- `created_at`, `updated_at`: Audit timestamps

**Key Methods:**
- `create()`: Creates new place listing
- `update()`: Modifies place information
- `add_amenity()`, `remove_amenity()`: Manages amenities
- `get_reviews()`: Retrieves all reviews for the place

#### **Review Entity**
**Attributes:**
- `id`: Unique identifier (UUID)
- `place_id`, `user_id`: Foreign keys
- `rating`: Integer from 1 to 5
- `comment`: Text feedback
- `created_at`, `updated_at`: Audit timestamps

**Key Methods:**
- `create()`: Creates new review with validation
- `validate_rating()`: Ensures rating is between 1-5
- `update()`: Modifies review content

#### **Amenity Entity**
**Attributes:**
- `id`: Unique identifier (UUID)
- `name`: Amenity name (e.g., "WiFi", "Pool")
- `description`: Detailed description
- `created_at`, `updated_at`: Audit timestamps

**Key Methods:**
- `create()`: Creates new amenity
- `get_places()`: Retrieves all places with this amenity

### 3.5 Business Rules Implementation

#### **Validation Rules**
1. **User Registration**
   - Email must be unique
   - Password must meet security requirements
   - First and last names are required

2. **Place Creation**
   - Must have valid owner
   - Price must be positive
   - Coordinates must be valid (-90 to 90 latitude, -180 to 180 longitude)

3. **Review Submission**
   - User cannot review their own place
   - Rating must be between 1 and 5
   - User can only review places once

4. **Amenity Management**
   - Amenity names must be unique
   - Cannot delete amenity if associated with places

---

## 4. API Interaction Flow

### 4.1 Sequence Diagrams Overview
The following sequence diagrams illustrate the flow of data and interactions between different layers of the application for key API operations.

### 4.2 API Call 1: User Registration

```
Client          API Layer       Facade        User Model    Persistence
  │                │               │               │              │
  │  POST          │               │               │              │
  │  /users        │               │               │              │
  ├───────────────>│               │               │              │
  │                │               │               │              │
  │                │ validate_data │               │              │
  │                ├───────────────┤               │              │
  │                │               │               │              │
  │                │ register()    │               │              │
  │                ├──────────────>│               │              │
  │                │               │               │              │
  │                │               │ create_user() │              │
  │                │               ├──────────────>│              │
  │                │               │               │              │
  │                │               │               │ validate()   │
  │                │               │               ├──────────────┤
  │                │               │               │              │
  │                │               │               │ check_unique │
  │                │               │               │ _email()     │
  │                │               │               ├─────────────>│
  │                │               │               │              │
  │                │               │               │ email_exists │
  │                │               │               │<─────────────┤
  │                │               │               │              │
  │                │               │               │ hash_password│
  │                │               │               ├──────────────┤
  │                │               │               │              │
  │                │               │               │ save()       │
  │                │               │               ├─────────────>│
  │                │               │               │              │
  │                │               │               │ user_saved   │
  │                │               │               │<─────────────┤
  │                │               │               │              │
  │                │               │ user_created  │              │
  │                │               │<──────────────┤              │
  │                │               │               │              │
  │                │ user_data     │               │              │
  │                │<──────────────┤               │              │
  │                │               │               │              │
  │  201 Created   │               │               │              │
  │  (User JSON)   │               │               │              │
  │<───────────────┤               │               │              │
  │                │               │               │              │
```

**Flow Description:**
1. Client sends POST request with user data
2. API layer validates input data format
3. Facade routes request to User model
4. User model validates business rules (unique email)
5. Password is hashed for security
6. Persistence layer saves user to database
7. Created user data is returned through layers
8. Client receives 201 Created with user information

**Key Validations:**
- Email format validation
- Email uniqueness check
- Password strength requirements
- Required field validation

### 4.3 API Call 2: Place Creation

```
Client       API Layer    Facade      User Model  Place Model  Persistence
  │             │            │             │            │            │
  │  POST       │            │             │            │            │
  │  /places    │            │             │            │            │
  │  +auth      │            │             │            │            │
  ├────────────>│            │             │            │            │
  │             │            │             │            │            │
  │             │ verify_auth│             │            │            │
  │             ├────────────┤             │            │            │
  │             │            │             │            │            │
  │             │ get_user() │             │            │            │
  │             ├───────────>│             │            │            │
  │             │            │ find_user() │            │            │
  │             │            ├────────────>│            │            │
  │             │            │             │ query_db() │            │
  │             │            │             ├───────────>│            │
  │             │            │             │ user_data  │            │
  │             │            │             │<───────────┤            │
  │             │            │ user_obj    │            │            │
  │             │            │<────────────┤            │            │
  │             │ user       │             │            │            │
  │             │<───────────┤             │            │            │
  │             │            │             │            │            │
  │             │ create()   │             │            │            │
  │             ├───────────>│             │            │            │
  │             │            │ create_place()           │            │
  │             │            ├─────────────────────────>│            │
  │             │            │             │            │            │
  │             │            │             │            │ validate() │
  │             │            │             │            ├────────────┤
  │             │            │             │            │            │
  │             │            │             │            │ save()     │
  │             │            │             │            ├───────────>│
  │             │            │             │            │ saved      │
  │             │            │             │            │<───────────┤
  │             │            │             │ place_created           │
  │             │            │<─────────────────────────┤            │
  │             │ place_data │             │            │            │
  │             │<───────────┤             │            │            │
  │  201        │            │             │            │            │
  │  Created    │            │             │            │            │
  │<────────────┤            │             │            │            │
```

**Flow Description:**
1. Client sends authenticated POST request
2. API layer verifies authentication token
3. Facade retrieves user information
4. Place model validates place data (price, coordinates)
5. Place is associated with authenticated user as owner
6. Persistence layer saves place
7. Created place is returned to client

**Key Validations:**
- User authentication
- Owner authorization
- Price validation (must be positive)
- Coordinate validation
- Required fields (title, description)

### 4.4 API Call 3: Review Submission

```
Client      API Layer   Facade    Place Model Review Model  Persistence
  │            │           │           │           │             │
  │  POST      │           │           │           │             │
  │  /reviews  │           │           │           │             │
  │  +auth     │           │           │           │             │
  ├───────────>│           │           │           │             │
  │            │           │           │           │             │
  │            │ verify()  │           │           │             │
  │            ├───────────┤           │           │             │
  │            │           │           │           │             │
  │            │ validate()│           │           │             │
  │            ├───────────┤           │           │             │
  │            │           │           │           │             │
  │            │ check_place_exists()  │           │             │
  │            ├──────────────────────>│           │             │
  │            │           │ query()   │           │             │
  │            │           │<──────────┤           │             │
  │            │           │           │           │             │
  │            │ check_not_owner()     │           │             │
  │            ├──────────────────────>│           │             │
  │            │           │ get_owner()           │             │
  │            │           │<──────────┤           │             │
  │            │           │           │           │             │
  │            │ create_review()       │           │             │
  │            ├──────────────────────────────────>│             │
  │            │           │           │           │             │
  │            │           │           │ validate_rating()       │
  │            │           │           │           ├─────────────┤
  │            │           │           │           │             │
  │            │           │           │ check_existing_review() │
  │            │           │           │           ├────────────>│
  │            │           │           │           │ query       │
  │            │           │           │           │<────────────┤
  │            │           │           │           │             │
  │            │           │           │           │ save()      │
  │            │           │           │           ├────────────>│
  │            │           │           │           │ saved       │
  │            │           │           │           │<────────────┤
  │            │           │           │ review_created          │
  │            │           │<──────────────────────┤             │
  │            │ review    │           │           │             │
  │            │<──────────┤           │           │             │
  │  201       │           │           │           │             │
  │  Created   │           │           │             │             │
  │<───────────┤           │           │           │             │
```

**Flow Description:**
1. Client submits review with authentication
2. API layer validates request and authentication
3. System verifies place exists
4. System checks user is not the place owner
5. Review model validates rating (1-5)
6. System checks user hasn't already reviewed this place
7. Review is saved to database
8. Created review is returned

**Business Rules Enforced:**
- User must be authenticated
- Place must exist
- User cannot review their own place
- Rating must be between 1 and 5
- User can only review a place once
- Comment is optional but recommended

### 4.5 API Call 4: Fetching List of Places

```
Client       API Layer     Facade      Place Model   Persistence
  │             │             │              │             │
  │  GET        │             │              │             │
  │  /places    │             │              │             │
  │  ?filters   │             │              │             │
  ├────────────>│             │              │             │
  │             │             │              │             │
  │             │ parse_params│              │             │
  │             ├─────────────┤              │             │
  │             │             │              │             │
  │             │ get_places()│              │             │
  │             ├────────────>│              │             │
  │             │             │ find_all()   │             │
  │             │             ├─────────────>│             │
  │             │             │              │             │
  │             │             │              │ query_with_ │
  │             │             │              │ filters()   │
  │             │             │              ├────────────>│
  │             │             │              │             │
  │             │             │              │ execute     │
  │             │             │              │ _query()    │
  │             │             │              ├─────────────┤
  │             │             │              │             │
  │             │             │              │ places_data │
  │             │             │              │<────────────┤
  │             │             │              │             │
  │             │             │              │ load_       │
  │             │             │              │ relationships│
  │             │             │              ├─────────────┤
  │             │             │              │             │
  │             │             │ places_list  │             │
  │             │             │<─────────────┤             │
  │             │             │              │             │
  │             │ format()    │              │             │
  │             ├─────────────┤              │             │
  │             │             │              │             │
  │  200 OK     │             │              │             │
  │  [places]   │             │              │             │
  │<────────────┤             │              │             │
```

**Flow Description:**
1. Client requests list of places with optional filters
2. API layer parses query parameters
3. Facade routes request to Place model
4. Place model builds database query with filters
5. Persistence layer executes query
6. Related data (owner, amenities) is loaded
7. Results are formatted and returned

**Supported Filters:**
- Price range (min_price, max_price)
- Location bounds (lat/lng coordinates)
- Amenities (filter by specific amenities)
- Pagination (page, page_size)

### 4.6 Common Patterns Across API Calls

#### **Authentication Flow**
All protected endpoints follow this pattern:
1. Extract authentication token from request
2. Validate token
3. Retrieve user from token
4. Proceed with request or return 401 Unauthorized

#### **Error Handling**
Standard error responses:
- `400 Bad Request`: Invalid input data
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: User lacks required permissions
- `404 Not Found`: Requested resource doesn't exist
- `409 Conflict`: Business rule violation (e.g., duplicate email)
- `500 Internal Server Error`: Unexpected server error

#### **Data Flow Pattern**
1. **Input Validation** → API Layer
2. **Business Logic** → Facade + Models
3. **Data Persistence** → Persistence Layer
4. **Response Formation** → API Layer
5. **Client Response** → JSON format

---

## 5. Conclusion

### 5.1 Architecture Summary
The HBnB Evolution application is built on a solid three-layer architecture that ensures:
- **Separation of concerns** through distinct layers
- **Maintainability** through the facade pattern
- **Scalability** through independent layer management
- **Testability** through clear interfaces between components

### 5.2 Key Design Decisions

#### **1. Layered Architecture**
**Decision**: Use three distinct layers (Presentation, Business Logic, Persistence)
**Rationale**: 
- Separates concerns for better code organization
- Allows independent testing and maintenance
- Facilitates team collaboration with clear boundaries
- Enables easier scaling of specific components

#### **2. Facade Pattern**
**Decision**: Implement facade pattern between layers
**Rationale**:
- Simplifies inter-layer communication
- Reduces coupling between components
- Provides single entry point for operations
- Makes the system easier to understand and use

#### **3. UUID for Entity IDs**
**Decision**: Use UUIDs instead of auto-incrementing integers
**Rationale**:
- Globally unique identifiers
- Better for distributed systems
- Prevents ID prediction
- Easier data migration and synchronization

#### **4. Many-to-Many Relationship for Place-Amenity**
**Decision**: Use association table for Place-Amenity relationship
**Rationale**:
- Amenities are reusable across places
- Flexible addition/removal of amenities
- Normalized database design
- Prevents data duplication

#### **5. Audit Timestamps**
**Decision**: Include created_at and updated_at for all entities
**Rationale**:
- Compliance and audit requirements
- Debugging and troubleshooting
- User transparency
- Data integrity tracking

### 5.3 Implementation Considerations

#### **Security**
- Password hashing (bcrypt or similar)
- JWT for authentication
- Input validation and sanitization
- Protection against SQL injection
- Rate limiting for API endpoints

#### **Performance**
- Database indexing on foreign keys
- Caching for frequently accessed data
- Pagination for large result sets
- Lazy loading for relationships
- Query optimization

#### **Data Integrity**
- Foreign key constraints
- Unique constraints on emails
- Check constraints on ratings
- Transaction management
- Validation at multiple layers

### 5.4 Future Enhancements
Potential areas for expansion:
1. **Booking System**: Add reservation functionality
2. **Payment Integration**: Process rental payments
3. **Messaging System**: User-to-user communication
4. **Image Management**: Upload and display place photos
5. **Advanced Search**: Full-text search, recommendations
6. **Analytics Dashboard**: Business intelligence features
7. **Mobile Applications**: iOS and Android clients
8. **Real-time Notifications**: WebSocket implementation

### 5.5 Next Steps
With this technical documentation complete, the project is ready to move into implementation:

**Part 2**: Implement the Business Logic Layer
- Create model classes
- Implement business rules
- Write unit tests
- Document API specifications

**Part 3**: Implement the Persistence Layer
- Design database schema
- Create migrations
- Implement repository pattern
- Test data access layer

**Part 4**: Implement the Presentation Layer
- Create RESTful API endpoints
- Implement authentication
- Add input validation
- Create API documentation

### 5.6 References and Resources

#### **Design Patterns**
- Gamma, E., et al. "Design Patterns: Elements of Reusable Object-Oriented Software"
- Fowler, M. "Patterns of Enterprise Application Architecture"

#### **API Design**
- RESTful API Design Best Practices
- OpenAPI Specification
- HTTP Status Codes

#### **Database Design**
- Normalization principles
- Indexing strategies
- ACID properties

#### **Python Resources**
- Python Official Documentation
- Flask/Django Documentation
- SQLAlchemy ORM Documentation

---

## Document Information

**Version**: 1.0  
**Date**: December 2025  
**Project**: HBnB Evolution  
**Phase**: Part 1 - Technical Documentation  
**Author**: Development Team  

**Document Status**: Final  
**Review Status**: Approved for Implementation  

**Change History**:
- v1.0 (Dec 2025): Initial comprehensive technical documentation

---

*This document serves as the authoritative reference for the HBnB Evolution project architecture and design. All implementation must adhere to the specifications outlined herein.*
