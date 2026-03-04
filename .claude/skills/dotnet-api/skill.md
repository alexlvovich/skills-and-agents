---
name: dotnet-api
description: REST API design principles including standards, resource naming, status codes, pagination, filtering, error responses and versioning for production APIs.

---

# REST API Design & Delivery Principles

## Overview

The REST API is one of the most common web interfaces used for communication between services. Despite its popularity, many implementations overlook fundamental principles such as security, performance, usability, and consistency.

Poor API design forces clients to solve avoidable technical problems instead of focusing on business logic.

This skill covers best practices for designing and delivering REST APIs in .NET environments

## When to use

- When designing new API endpoints
- When reviewing existing API contracts for improvements
- When implementing pagination, filtering, or sorting
- When implementing error handling for APIs
- When planning API versioning strategy
- When building public or partner-facing APIs

##  Instructions

APIs are developer products. They must be designed with care and attention to detail.

Responses should:

- Return only necessary data
- Avoid over-fetching
- Use meaningful HTTP status codes
- Provide structured error information
- Clear naming
- Predictable behavior
- Consistent structure
- Helpful error messages
- Sample requests and responses

### Standardization

Lack of standards is one of the most damaging issues in API development.

- Use consistent HTTP methods according to their semantics (GET for retrieval, POST for creation, etc.)
- Use consistent status codes to indicate success or failure (200/201 for success, 400 for validation errors, 500 for server errors, etc.)
- Use consistent response formats across all endpoints

Even a weak standard is better than none.

### Naming

API naming should be:

- Clear
- Predictable
- Resource-oriented
- Consistent

**Principles:**

- Use nouns, not verbs
- Use plural resource names
- Follow consistent casing (e.g., kebab-case or snake_case)
- Maintain consistent URL structure

```
GET    /api/members?skip=0&take=10
GET    /api/members/:id
POST   /api/members
PUT    /api/members/:id
DELETE /api/members/:id

# Sub-resources
GET    /api/members/:id/orders
POST   /api/members/:id/orders

# Actions that don't map to CRUD (use verbs sparingly)
POST   /api/orders/:id/cancel
POST   /api/auth/login
POST   /api/auth/refresh
```

### HTTP Methods

| Method | Idempotent | Safe | Use For |
|--------|-----------|------|---------|
| GET | Yes | Yes | Retrieve objects |
| POST | No | No | Create object, enqueue, trigger actions |
| PUT | Yes | No | Partial or full replacement of an object |
| DELETE | Yes | No | Remove an object |

### HTTP Codes

**Success**

200 OK                    — GET, PUT, PATCH (with response body)
201 Created               — POST
204 No Content            — DELETE (with response body)

**Validation Errors**

400 Bad Request           — Validation failure
401 Unauthorized          — Missing or invalid authentication
403 Forbidden             — Authenticated but not authorized
404 Not Found             — Resource doesn't exist
429 Too Many Requests     — Rate limit exceeded

**Server Errors**

500 Internal Server Error  — Unhandled exception

### Response Formats

Classes and abstractions can be found [https://github.com/alexlvovich/GetInfra.WebApi.Abstractions](https://github.com/alexlvovich/GetInfra.WebApi.Abstractions)

validation error:

```json
{
  "name": string,
  "attemptedValue": object,
  "message": string
}
```

Error:

```json
{
  "correlationId": string,
  "stack": string,
  "message": string
}
```

GenericResultResponse<T> response:

```json
{
  "errors": [array of error objects],
  "validationErrors": [validation error object],
  "result": T,
  "succeeded": boolean
}
```

`PagedResponse<T>` response including pagination metadata:

```json
{
  "list": [array of T objects],
  "total": number
}
```

#### POST

- Use request class for the call e.g. `CreateUserRequest` which can be read from the request body ([FromBody] attribute in .NET)
- Return `GenericResultResponse<T>` with the create id in the `Result` property


Example:

POST /api/members 
Request body:

```json
{
  "email": "minerva@example.com",
  "name": "Minerva"
}
```

Response body with 201 Created status code:

```json
{
  "succeeded": true,
  "result": 123
}
```

Response body with 400 Bad Request status code (validation error):

```json
{
  "succeeded": false,
  "validationErrors": [ 
    {
      "name": "Email",
      "attemptedValue": "invalid-email",
      "message": "The Email field is not a valid e-mail address."
    }
  ]
}
```


#### PUT

- Use request class for the call e.g. `UpdateUserRequest` which can be read from the request body ([FromBody] attribute in .NET)
- Return `GenericResultResponse<bool>` with bool in the `Result` property

PUT /api/members 
Request body:

```json
{
  "id": 123,
  "email": "minerva@example.com",
  "name": "Minerva"
}
```

Response body with 200 Created status code:

```json
{
  "succeeded": true,
  "result": true
}
```

Response body with 400 Bad Request status code (validation error):

```json
{
  "succeeded": false,
  "validationErrors": [ 
    {
      "name": "Email",
      "attemptedValue": "invalid-email",
      "message": "The Email field is not a valid e-mail address."
    }
  ]
}
```

#### GET

Used for retrieval of single items or lists.

- Return `PagedResponse<T>` for list endpoints, with the list of items in the `List` property and total count in the `Total` property with ok status code (200)
- Implement pagination using `from` and `to` query parameters for list endpoints
- Implement filtering and sorting using query parameters when applicable

Example:

GET /api/members?from=0&to=1

Response body with 200 OK status code:

```json
{
  "list": [
    {
      "id": 123,
      "email": "minerva@example.com",
      "name": "Minerva"
    }
  ],
  "total": 10
}
```

#### DELETE

- Return `GenericResultResponse<bool>` with bool in the `Result` or just in `succeeded` property
- Validate that user allowed to delete the resource and return 403 Forbidden if not allowed
- Return 404 Not Found if the resource doesn't exist

Example:  

DELETE /api/members/123

Response body with 200 `OK` or 204 `No Content` status code:

```json
{
  "succeeded": true,
  "result": true
}
```

### Authentication & Authorization

Authentication verifies identity.  
Authorization controls access.

A proper API endpoint must:

- Use secure authentication mechanisms (e.g., OAuth2, JWT)
- Enforce role-based or permission-based access, when it possible
- Protect sensitive endpoints
- Avoid exposing internal resources

### Fuctionalities for offboarding 

Leverage an API Gateway for offloading infrastructure-related (platform-level) concerns:

- HTTPS/TLS enforcement
- Authentication and authorization
- Rate limiting and throttling
- Caching and response compression
- Request routing
- Observability (logging, monitoring, tracing)

### Versioning

Versioning enables safe evolution.

- Implement versioning only when necessary (e.g., breaking changes, new major features)
- No version in url - meaning v1 
- Use URL versioning (e.g., /api/v2/resource) for simplicity and discover 
- Maintain backward compatibility create new endpoints instead of modifying existing ones when possible
- Deprecate versions responsibly
- Avoid silent breaking changes.


### Security

Security extends beyond authentication.

- Input validation
- SQL injection prevention
- XSS protection
- CSRF protection where applicable
- Proper secrets management
- Sensitive data encryption

### Testing Strategy

API quality requires:

- Unit testing
- Integration testing
- Contract testing
- Load testing

APIs are contracts — breaking them without testing damages trust.
