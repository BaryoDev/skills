---
name: baryo-api
description: RESTful and GraphQL API design standards covering versioning, pagination, error handling, documentation, and rate limiting for production APIs.
---

# BaryoDev API Standard

You are the API Architect. **APIs are the contract with the world—make them unbreakable.**

## Core Philosophy

"An API is a user interface for developers. Design it like you would any great UX."

---

## Part 1: RESTful Design

### Resource Naming

```
# ✅ Good: Nouns, plural
GET    /users           # List users
GET    /users/123       # Get user 123
POST   /users           # Create user
PUT    /users/123       # Update user 123
DELETE /users/123       # Delete user 123

# ❌ Bad: Verbs, actions
GET    /getUsers
POST   /createUser
GET    /deleteUser/123
```

### Nested Resources

```
# ✅ Good: Clear hierarchy (max 2 levels deep)
GET /users/123/orders           # User's orders
GET /orders/456/items           # Order's items

# ❌ Bad: Too deep
GET /users/123/orders/456/items/789/reviews
```

### HTTP Methods

| Method | Purpose          | Idempotent | Safe |
| ------ | ---------------- | ---------- | ---- |
| GET    | Read resource    | ✅          | ✅    |
| POST   | Create resource  | ❌          | ❌    |
| PUT    | Replace resource | ✅          | ❌    |
| PATCH  | Partial update   | ❌          | ❌    |
| DELETE | Remove resource  | ✅          | ❌    |

---

## Part 2: Response Format

### Standard Response Structure

```json
// Success response
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "requestId": "abc-123",
    "timestamp": "2025-01-15T10:30:00Z"
  }
}

// Collection response
{
  "data": [
    { "id": "123", "name": "John" },
    { "id": "456", "name": "Jane" }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 100,
    "totalPages": 5
  },
  "meta": {
    "requestId": "abc-123"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  },
  "meta": {
    "requestId": "abc-123"
  }
}
```

### HTTP Status Codes

| Code | When to Use                          |
| ---- | ------------------------------------ |
| 200  | Success (GET, PUT, PATCH)            |
| 201  | Created (POST)                       |
| 204  | No Content (DELETE)                  |
| 400  | Bad Request (validation error)       |
| 401  | Unauthorized (not authenticated)     |
| 403  | Forbidden (not authorized)           |
| 404  | Not Found                            |
| 409  | Conflict (duplicate, state conflict) |
| 429  | Too Many Requests (rate limited)     |
| 500  | Internal Server Error                |
| 503  | Service Unavailable                  |

---

## Part 3: Pagination

### Cursor-Based Pagination (Recommended)

```json
// Request
GET /users?limit=20&cursor=eyJpZCI6MTIzfQ

// Response
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTQzfQ",
    "hasMore": true
  }
}
```

### Offset Pagination (Simple Cases)

```json
// Request
GET /users?page=2&pageSize=20

// Response
{
  "data": [...],
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "totalItems": 100,
    "totalPages": 5
  }
}
```

---

## Part 4: Versioning

### URL Path Versioning (Recommended)

```
GET /v1/users
GET /v2/users
```

### Version Header (Alternative)

```
GET /users
Accept: application/vnd.myapi.v2+json
```

### Versioning Rules

- Always start with v1
- Major version for breaking changes only
- Support N-1 versions minimum
- Provide deprecation warnings

---

## Part 5: Rate Limiting

### Rate Limit Headers

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

### Rate Limit Response (429)

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

### Implementation

```typescript
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: {
        code: 'RATE_LIMIT_EXCEEDED',
        message: 'Too many requests, please try again later',
        retryAfter: Math.ceil(req.rateLimit.resetTime / 1000),
      },
    });
  },
});

app.use('/api/', apiLimiter);
```

---

## Part 6: Error Handling

### Standard Error Codes

```typescript
enum ErrorCode {
  // Client errors
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  AUTHENTICATION_REQUIRED = 'AUTHENTICATION_REQUIRED',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  RESOURCE_NOT_FOUND = 'RESOURCE_NOT_FOUND',
  RESOURCE_CONFLICT = 'RESOURCE_CONFLICT',
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED',
  
  // Server errors
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  SERVICE_UNAVAILABLE = 'SERVICE_UNAVAILABLE',
  DEPENDENCY_ERROR = 'DEPENDENCY_ERROR',
}
```

### Error Response Examples

```json
// Validation error
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      { "field": "email", "code": "INVALID_FORMAT", "message": "Must be valid email" },
      { "field": "age", "code": "OUT_OF_RANGE", "message": "Must be between 0 and 150" }
    ]
  }
}

// Not found
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "User not found",
    "resource": "User",
    "id": "123"
  }
}
```

---

## Part 7: API Documentation

### OpenAPI/Swagger Required

Every API endpoint MUST have OpenAPI documentation:

```yaml
openapi: 3.0.0
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

---

## Part 8: GraphQL Standards

### Schema Design

```graphql
type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!
}

type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

### Error Handling

```json
{
  "data": null,
  "errors": [
    {
      "message": "User not found",
      "extensions": {
        "code": "RESOURCE_NOT_FOUND",
        "resource": "User",
        "id": "123"
      }
    }
  ]
}
```

---

## Part 9: API Checklist

### Before Release

- [ ] All endpoints documented in OpenAPI
- [ ] Consistent response format
- [ ] Proper error codes
- [ ] Rate limiting configured
- [ ] Versioning in place
- [ ] Pagination for lists
- [ ] Authentication/authorization
- [ ] Input validation
- [ ] CORS configured
- [ ] Content-Type headers

---

**Remember**: Your API is a product. Treat it with the same care as your UI.
