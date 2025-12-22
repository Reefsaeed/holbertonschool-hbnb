# Task 2: Sequence Diagrams for API Calls

## Overview
This document contains sequence diagrams for four critical API operations that demonstrate the interaction flow between the different layers of the HBnB application.

---

## 1. User Registration (POST /api/users/register)

### Sequence Diagram

```mermaid
sequenceDiagram
    actor Client
    participant API as API Layer
    participant Facade as Facade
    participant User as User Model
    participant DB as Database

    Client->>API: POST /api/users/register<br/>{email, password, first_name, last_name}
    
    API->>API: Validate input format
    alt Invalid format
        API-->>Client: 400 Bad Request<br/>{error: "Invalid input"}
    end
    
    API->>Facade: create_user(user_data)
    Facade->>User: register(user_data)
    
    User->>User: Validate business rules<br/>(email format, password strength)
    
    User->>DB: Check email uniqueness<br/>SELECT * FROM users WHERE email=?
    
    alt Email already exists
        DB-->>User: Email found
        User-->>Facade: ValidationError
        Facade-->>API: ValidationError
        API-->>Client: 409 Conflict<br/>{error: "Email already exists"}
    end
    
    DB-->>User: Email available
    
    User->>User: Hash password (bcrypt)
    User->>User: Generate UUID
    User->>User: Set created_at, updated_at
    
    User->>DB: INSERT INTO users<br/>(id, email, password_hash, ...)
    DB-->>User: Insert successful
    
    User-->>Facade: User object (without password)
    Facade-->>API: User object
    
    API->>API: Format response<br/>Remove sensitive data
    API-->>Client: 201 Created<br/>{id, email, first_name, last_name, created_at}
```

### Request Example
```http
POST /api/users/register HTTP/1.1
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "SecurePass123!",
  "first_name": "John",
  "last_name": "Doe"
}
```

### Response Example (Success)
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "john.doe@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_admin": false,
  "created_at": "2024-12-19T10:30:00Z"
}
```

### Key Validations
1. **Input Format**: Email format, password length
2. **Business Rules**: Email uniqueness
3. **Security**: Password hashing before storage
4. **Response**: Sensitive data (password) excluded

---

## 2. Place Creation (POST /api/places)

### Sequence Diagram

```mermaid
sequenceDiagram
    actor Client
    participant API as API Layer
    participant Auth as Auth Middleware
    participant Facade as Facade
    participant User as User Model
    participant Place as Place Model
    participant DB as Database

    Client->>API: POST /api/places<br/>Authorization: Bearer <token><br/>{title, description, price, latitude, longitude}
    
    API->>Auth: Verify JWT token
    
    alt Invalid/Expired token
        Auth-->>API: Authentication failed
        API-->>Client: 401 Unauthorized<br/>{error: "Invalid token"}
    end
    
    Auth->>DB: Get user from token<br/>SELECT * FROM users WHERE id=?
    DB-->>Auth: User data
    Auth-->>API: Authenticated user
    
    API->>API: Validate input data
    
    API->>Facade: create_place(user, place_data)
    Facade->>Place: create(user, place_data)
    
    Place->>Place: Validate business rules<br/>(price > 0, valid coordinates)
    
    alt Invalid coordinates
        Place-->>Facade: ValidationError
        Facade-->>API: ValidationError
        API-->>Client: 400 Bad Request<br/>{error: "Invalid coordinates"}
    end
    
    Place->>Place: validate_coordinates()<br/>lat: -90 to 90, lng: -180 to 180
    Place->>Place: Generate UUID
    Place->>Place: Set owner_id = user.id
    Place->>Place: Set created_at, updated_at
    
    Place->>DB: INSERT INTO places<br/>(id, title, description, price, ...)
    DB-->>Place: Insert successful
    
    Place-->>Facade: Place object
    Facade-->>API: Place object
    API-->>Client: 201 Created<br/>{id, title, price, owner, ...}
```

### Request Example
```http
POST /api/places HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "title": "Cozy Downtown Apartment",
  "description": "Beautiful 2BR apartment in the heart of the city",
  "price": 120.00,
  "latitude": 40.7128,
  "longitude": -74.0060
}
```

### Response Example (Success)
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "title": "Cozy Downtown Apartment",
  "description": "Beautiful 2BR apartment in the heart of the city",
  "price": 120.00,
  "latitude": 40.7128,
  "longitude": -74.0060,
  "owner": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "first_name": "John",
    "last_name": "Doe"
  },
  "created_at": "2024-12-19T10:35:00Z"
}
```

### Key Validations
1. **Authentication**: Valid JWT token required
2. **Authorization**: User must be authenticated
3. **Price**: Must be positive number
4. **Coordinates**: Valid latitude/longitude ranges
5. **Owner**: Automatically set from authenticated user

---

## 3. Review Submission (POST /api/reviews)

### Sequence Diagram

```mermaid
sequenceDiagram
    actor Client
    participant API as API Layer
    participant Auth as Auth Middleware
    participant Facade as Facade
    participant Place as Place Model
    participant Review as Review Model
    participant DB as Database

    Client->>API: POST /api/reviews<br/>Authorization: Bearer <token><br/>{place_id, rating, comment}
    
    API->>Auth: Verify JWT token
    Auth->>DB: Get user from token
    DB-->>Auth: User data
    Auth-->>API: Authenticated user
    
    API->>API: Validate input<br/>(rating 1-5, place_id format)
    
    API->>Facade: create_review(user, review_data)
    Facade->>Place: get_by_id(place_id)
    
    Place->>DB: SELECT * FROM places WHERE id=?
    
    alt Place not found
        DB-->>Place: No results
        Place-->>Facade: NotFoundError
        Facade-->>API: NotFoundError
        API-->>Client: 404 Not Found<br/>{error: "Place not found"}
    end
    
    DB-->>Place: Place data
    Place-->>Facade: Place object
    
    Facade->>Place: Check if user is owner
    
    alt User is place owner
        Place-->>Facade: OwnershipError
        Facade-->>API: ValidationError
        API-->>Client: 403 Forbidden<br/>{error: "Cannot review own place"}
    end
    
    Facade->>Review: create(user, place, review_data)
    
    Review->>Review: validate_rating()<br/>Check 1-5 range
    
    Review->>DB: Check existing review<br/>SELECT * FROM reviews<br/>WHERE user_id=? AND place_id=?
    
    alt Review already exists
        DB-->>Review: Review found
        Review-->>Facade: DuplicateError
        Facade-->>API: ValidationError
        API-->>Client: 409 Conflict<br/>{error: "Already reviewed"}
    end
    
    DB-->>Review: No existing review
    
    Review->>Review: Generate UUID
    Review->>Review: Set created_at, updated_at
    
    Review->>DB: INSERT INTO reviews<br/>(id, place_id, user_id, rating, comment, ...)
    DB-->>Review: Insert successful
    
    Review-->>Facade: Review object
    Facade-->>API: Review object
    API-->>Client: 201 Created<br/>{id, place_id, rating, comment, ...}
```

### Request Example
```http
POST /api/reviews HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "place_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "rating": 5,
  "comment": "Amazing place! Great location and very clean."
}
```

### Response Example (Success)
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "place_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "user": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "first_name": "Jane",
    "last_name": "Smith"
  },
  "rating": 5,
  "comment": "Amazing place! Great location and very clean.",
  "created_at": "2024-12-19T11:00:00Z"
}
```

### Business Rules Enforced
1. **Authentication**: User must be logged in
2. **Place Existence**: Place must exist
3. **Ownership**: User cannot review their own place
4. **Rating Range**: Must be 1-5
5. **Uniqueness**: One review per user per place

---

## 4. Fetching List of Places (GET /api/places)

### Sequence Diagram

```mermaid
sequenceDiagram
    actor Client
    participant API as API Layer
    participant Facade as Facade
    participant Place as Place Model
    participant Amenity as Amenity Model
    participant DB as Database

    Client->>API: GET /api/places?<br/>min_price=50&max_price=200&<br/>amenities=wifi,pool&page=1&page_size=10
    
    API->>API: Parse query parameters<br/>Extract filters and pagination
    
    API->>Facade: get_places(filters, pagination)
    Facade->>Place: find_all(filters, pagination)
    
    Place->>Place: Build SQL query<br/>Apply filters
    
    Place->>DB: SELECT * FROM places<br/>WHERE price BETWEEN ? AND ?<br/>LIMIT ? OFFSET ?
    DB-->>Place: List of place records
    
    Place->>Place: Load owner information<br/>for each place
    
    Place->>DB: SELECT * FROM users<br/>WHERE id IN (owner_ids)
    DB-->>Place: User data
    
    alt Amenity filter provided
        Place->>Amenity: filter_by_amenities(place_ids, amenity_names)
        Amenity->>DB: SELECT p.* FROM places p<br/>JOIN place_amenity pa ON p.id = pa.place_id<br/>JOIN amenities a ON pa.amenity_id = a.id<br/>WHERE a.name IN (?)
        DB-->>Amenity: Filtered place IDs
        Amenity-->>Place: Filtered places
    end
    
    Place->>Place: Load amenities for each place
    
    Place->>DB: SELECT a.* FROM amenities a<br/>JOIN place_amenity pa ON a.id = pa.amenity_id<br/>WHERE pa.place_id IN (?)
    DB-->>Place: Amenity data
    
    Place->>Place: Calculate total count<br/>for pagination metadata
    
    Place->>DB: SELECT COUNT(*) FROM places<br/>WHERE price BETWEEN ? AND ?
    DB-->>Place: Total count
    
    Place-->>Facade: Places list + metadata
    Facade-->>API: Places list + metadata
    
    API->>API: Format response<br/>Add pagination links
    
    API-->>Client: 200 OK<br/>{data: [...], pagination: {...}}
```

### Request Example
```http
GET /api/places?min_price=50&max_price=200&amenities=wifi,pool&page=1&page_size=10 HTTP/1.1
```

### Response Example (Success)
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    {
      "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
      "title": "Cozy Downtown Apartment",
      "description": "Beautiful 2BR apartment",
      "price": 120.00,
      "latitude": 40.7128,
      "longitude": -74.0060,
      "owner": {
        "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "first_name": "John",
        "last_name": "Doe"
      },
      "amenities": [
        {"id": "...", "name": "WiFi"},
        {"id": "...", "name": "Pool"}
      ],
      "created_at": "2024-12-19T10:35:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 10,
    "total_items": 45,
    "total_pages": 5,
    "has_next": true,
    "has_prev": false
  },
  "links": {
    "self": "/api/places?page=1&page_size=10&min_price=50&max_price=200",
    "next": "/api/places?page=2&page_size=10&min_price=50&max_price=200",
    "first": "/api/places?page=1&page_size=10&min_price=50&max_price=200",
    "last": "/api/places?page=5&page_size=10&min_price=50&max_price=200"
  }
}
```

### Supported Query Parameters
- `min_price`: Minimum price filter
- `max_price`: Maximum price filter
- `latitude`, `longitude`, `radius`: Location-based search
- `amenities`: Comma-separated amenity names
- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 10, max: 100)

### Query Optimizations
1. **Indexes**: On price, location fields
2. **Eager Loading**: Fetch related data efficiently
3. **Pagination**: Limit result size
4. **Caching**: Cache frequently accessed places

---

## Summary of API Interactions

### Common Patterns Across All APIs

#### 1. **Authentication Flow** (Protected Endpoints)
```
Request → Extract Token → Verify Token → Get User → Proceed
```

#### 2. **Validation Flow**
```
Input Data → Format Validation → Business Rule Validation → Database Validation
```

#### 3. **Error Handling**
```
Error Occurs → Log Error → Format Error Response → Return Appropriate HTTP Status
```

#### 4. **Success Flow**
```
Process Request → Save to Database → Format Response → Return Success Status
```

### HTTP Status Codes Used

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, DELETE |
| 201 | Created | Successful POST (resource created) |
| 400 | Bad Request | Invalid input format |
| 401 | Unauthorized | Missing/invalid authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Business rule violation (duplicate, etc.) |
| 500 | Internal Server Error | Unexpected server error |

### Data Flow Layers

```
Client Request
    ↓
API Layer (Validation, Auth)
    ↓
Facade (Routing)
    ↓
Business Logic (Models)
    ↓
Persistence Layer (Database)
    ↓
Response Back Through Layers
    ↓
Client Response
```

---

## File Information

**Task**: Task 2 - Sequence Diagrams for API Calls  
**Deliverable**: Four detailed sequence diagrams showing layer interactions  
**Format**: Mermaid sequence diagrams  
**Location**: `holbertonschool-hbnb/part1/`  
**API Operations Covered**:
1. User Registration (POST /api/users/register)
2. Place Creation (POST /api/places)
3. Review Submission (POST /api/reviews)
4. Fetching Places List (GET /api/places)
