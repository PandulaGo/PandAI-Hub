# API Design Standard

## URL Patterns

### RESTful Conventions
- Use plural nouns for resources: `/api/v1/users` (not `/user`)
- Use hyphens for multi-word resources: `/api/v1/order-items`
- Nest related resources: `/api/v1/users/:id/orders`
- Maximum 2 levels of nesting to avoid complexity

### HTTP Methods
```
GET    /api/v1/users          # List all users
GET    /api/v1/users/:id      # Get specific user
POST   /api/v1/users          # Create new user
PUT    /api/v1/users/:id      # Update entire user
PATCH  /api/v1/users/:id      # Partial update
DELETE /api/v1/users/:id      # Delete user
```

### Query Parameters
```
# Pagination
GET /api/v1/users?page=1&pageSize=20

# Sorting
GET /api/v1/users?sortBy=name&sortOrder=asc

# Filtering
GET /api/v1/users?status=active&role=admin

# Search
GET /api/v1/users?search=john
```

## Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

### List Response
```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "John Doe" },
    { "id": 2, "name": "Jane Smith" }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

## Error Format

### Standard Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Email is required",
        "code": "REQUIRED"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters",
        "code": "MIN_LENGTH"
      }
    ]
  }
}
```

### Error Codes
- `VALIDATION_ERROR` - Request validation failed
- `NOT_FOUND` - Resource not found
- `UNAUTHORIZED` - Authentication required
- `FORBIDDEN` - Insufficient permissions
- `CONFLICT` - Resource conflict (duplicate, etc.)
- `INTERNAL_ERROR` - Server error
- `BAD_REQUEST` - Invalid request format

## HTTP Status Codes

### Success Codes
- `200 OK` - Successful GET, PUT, PATCH
- `201 Created` - Successful POST
- `204 No Content` - Successful DELETE (or update with no response body)

### Client Error Codes
- `400 Bad Request` - Invalid request format or validation error
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Resource conflict (e.g., duplicate email)
- `422 Unprocessable Entity` - Validation error with details

### Server Error Codes
- `500 Internal Server Error` - Unexpected server error
- `503 Service Unavailable` - Service temporarily unavailable

## Pagination

### Standard Approach
- Use `page` and `pageSize` query parameters
- Default: `page=1`, `pageSize=20`
- Maximum: `pageSize=100` (prevent abuse)
- Always return `meta` object with pagination info

### Response Headers (Optional)
```
X-Page: 1
X-Page-Size: 20
X-Total: 100
X-Total-Pages: 5
```

## Authentication

### Bearer Token (JWT)
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Rules
- Always use HTTPS in production
- Never pass tokens in query strings
- Include token expiration in response
- Support token refresh mechanism
- Use short-lived access tokens + long-lived refresh tokens

## Versioning

### URL Path Versioning
```
/api/v1/users
/api/v2/users
```

### Rules
- Breaking changes require new version
- Support previous version for at least 6 months
- Document version differences
- Use semantic versioning: v1, v2, v3

## Rate Limiting

### Standard Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1609459200
```

### Response (when exceeded)
```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

## CORS (Cross-Origin Resource Sharing)

### Required Headers
```
Access-Control-Allow-Origin: https://your-frontend.com
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

## Content Negotiation

### Request Headers
```
Accept: application/json
Content-Type: application/json
```

### Response Headers
```
Content-Type: application/json; charset=utf-8
```

## Best Practices

### Do
- Use consistent naming across all endpoints
- Return appropriate HTTP status codes
- Include pagination for list endpoints
- Validate all input data
- Use proper error handling
- Document all endpoints
- Version your API from day one

### Don't
- Don't use verbs in URLs (use `/users` not `/getUsers`)
- Don't expose stack traces in production
- Don't return sensitive data (passwords, tokens in response)
- Don't mix plural/singular naming
- Don't nest more than 2 levels deep
- Don't use custom status codes (stick to HTTP standards)
