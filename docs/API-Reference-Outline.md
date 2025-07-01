# API Reference Specification - STAR Community App

## 0. Version History

| API Version | Doc Version | Date       | Author(s)   | Summary of Changes                                                                 |
|-------------|-------------|------------|-------------|------------------------------------------------------------------------------------|
| v1.0        | 1.0         | YYYY-MM-DD | Jules (AI)  | Initial draft outline for v1 API.                                                  |
|             |             |            |             |                                                                                    |

## 1. Introduction

### 1.1 Purpose
    - To provide comprehensive documentation for the STAR Community App v1 API.
    - Intended audience: Frontend developers, mobile app developers, third-party integrators, backend developers.

### 1.2 API Overview
    - Brief description of the API's capabilities and what it enables.
    - Mention of the microservice architecture and how the API Gateway serves as the entry point.

### 1.3 Base URLs
    - Production Base URL: `https://api.starcommunity.app/api/v1` (Example)
    - Staging/Development Base URL: `https://staging-api.starcommunity.app/api/v1` (Example)
    - Note on using the correct base URL for the target environment.

### 1.4 Authentication
    - Authentication method: JWT (JSON Web Tokens).
    - How to obtain a token (e.g., via `/auth/login` endpoint).
    - How to send the token: `Authorization: Bearer <JWT_TOKEN>` header.
    - Token expiry and refresh mechanisms (if applicable).
    - Link to detailed authentication endpoints.

### 1.5 API Versioning
    - Current API Version: v1.
    - Versioning strategy: URL path based (e.g., `/api/v1/...`).
    - Policy on backward compatibility and deprecation.

### 1.6 Rate Limiting
    - General rate limiting policies (e.g., X requests per minute per IP/user).
    - Specific limits for sensitive endpoints if any.
    - HTTP status code for rate limit exceeded (e.g., `429 Too Many Requests`) and relevant headers (`Retry-After`).

### 1.7 General Conventions
    - Request/Response format: JSON (`application/json`).
    - Naming conventions for JSON fields (e.g., `camelCase`).
    - Date and time format (e.g., ISO 8601: `YYYY-MM-DDTHH:mm:ss.sssZ`).
    - Handling of `null` vs. omitted fields.

## 2. Common API Information

### 2.1 HTTP Status Codes
    - List of commonly used HTTP status codes and their meaning in the context of this API:
        - `200 OK`
        - `201 Created`
        - `204 No Content`
        - `400 Bad Request` (and general error response structure)
        - `401 Unauthorized`
        - `403 Forbidden`
        - `404 Not Found`
        - `422 Unprocessable Entity` (for semantic validation errors not caught by schema)
        - `429 Too Many Requests`
        - `500 Internal Server Error`

### 2.2 Standard Error Response Format
    - Detailed structure of the JSON error response (as shown in Backend Architecture Plan).
        ```json
        {
            "success": false,
            "error": {
                "code": "ERROR_CODE_STRING",
                "message": "User-friendly or developer-friendly error message.",
                "details": [ // Optional array for specific validation errors
                    { "field": "fieldName", "message": "Specific error for this field." }
                ]
            }
        }
        ```
    - Common error codes (`code` field) and their meanings (e.g., `VALIDATION_ERROR`, `AUTHENTICATION_FAILED`, `NOT_FOUND`, `INSUFFICIENT_PERMISSIONS`).

### 2.3 Pagination
    - Standard query parameters for pagination (e.g., `page` or `offset`, `pageSize` or `limit`).
    - Structure of paginated responses (e.g., including `currentPage`, `pageSize`, `totalPages`, `totalItems`, `data: []`).

### 2.4 Filtering and Sorting
    - General approach to filtering collections (e.g., `?status=active&category=X`).
    - General approach to sorting collections (e.g., `?sortBy=createdAt&order=desc`).

## 3. API Endpoints by Microservice / Resource Group

*(Repeat this section for each microservice or logical group of resources. Each microservice will have its own set of resources and endpoints.)*

### 3.X [Microservice Name / Resource Group Name]

    - Brief description of the resources and functionalities provided by this group of endpoints.

    *(For each endpoint within this group, provide the following details. This is where tools like Swagger UI shine by generating interactive documentation from an OpenAPI spec.)*

#### 3.X.Y [HTTP Method] [Path] (e.g., `POST /auth/register`)

*   **Summary:** Short, human-readable summary of what the endpoint does (e.g., "Register a new user").
*   **Description:** More detailed explanation if needed.
*   **Tags:** (For grouping in OpenAPI/Swagger, e.g., `Authentication`, `Users`)
*   **Parameters:**
    *   **Path Parameters:**
        | Name      | In   | Description                      | Required | Type   | Example         |
        |-----------|------|----------------------------------|----------|--------|-----------------|
        | `userId`  | path | ID of the user to retrieve       | true     | string (uuid) | `a1b2c3d4-...`  |
    *   **Query Parameters:**
        | Name        | In    | Description                      | Required | Type   | Example         |
        |-------------|-------|----------------------------------|----------|--------|-----------------|
        | `status`    | query | Filter by status                 | false    | string | `active`        |
        | `page`      | query | Page number for pagination       | false    | integer| `1`             |
    *   **Header Parameters:** (Usually common, like `Authorization`, `Content-Type`)
*   **Request Body:**
    *   **Description:** Description of the request payload.
    *   **Content Type:** e.g., `application/json`.
    *   **Schema:** Detailed definition of the request body JSON structure (can reference a global schema/model).
        ```json
        // Example for POST /auth/register
        {
            "fullName": "string (required)",
            "email": "string (email, required)",
            "password": "string (minLength: 8, required)",
            "role": "string (enum: [Resident, ServiceProvider, Agent], required)"
        }
        ```
    *   **Example Value:** A complete example of a valid request body.
*   **Responses:**
    *(List all possible responses with their HTTP status codes)*
    *   **`200 OK` / `201 Created` / `204 No Content` (Success Response):**
        *   **Description:** Description of the successful outcome.
        *   **Content Type:** e.g., `application/json` (or none for 204).
        *   **Schema:** Detailed definition of the response body JSON structure.
        *   **Example Value:** A complete example of a successful response body.
        *   **Headers:** Any specific headers returned on success (e.g., `Location` for 201).
    *   **`400 Bad Request` (Validation Error / Malformed Request):**
        *   **Description:** "Invalid input provided."
        *   **Schema:** Standard Error Response Format.
        *   **Example Value:** Example of a 400 error response.
    *   **`401 Unauthorized` (Authentication Failure):**
        *   **Description:** "Authentication token is missing or invalid."
        *   **Schema:** Standard Error Response Format.
    *   **`403 Forbidden` (Authorization Failure):**
        *   **Description:** "User does not have permission to perform this action."
        *   **Schema:** Standard Error Response Format.
    *   **`404 Not Found` (Resource Not Found):**
        *   **Description:** "The requested resource was not found."
        *   **Schema:** Standard Error Response Format.
    *   *(Other relevant error codes specific to the endpoint)*
*   **Security:**
    *   Authentication requirements (e.g., "JWT Bearer token required").
    *   Required roles/permissions if any (e.g., "Requires 'Admin' role").
*   **Notes / Business Logic:**
    *   Any specific business rules, side effects, or important considerations for this endpoint.

## 4. Data Models / Schemas (Reusable Objects)

    - This section defines common data structures (JSON objects) that are reused across multiple API endpoints in request or response bodies.
    - For each model:
        - Model Name (e.g., `User`, `Booking`, `Wallet`)
        - Properties:
            | Field Name  | Data Type              | Required | Description                                     | Example Value    |
            |-------------|------------------------|----------|-------------------------------------------------|------------------|
            | `userId`    | string (uuid)          | true     | Unique identifier for the user.                 | `a1b2c3d4-...`   |
            | `fullName`  | string                 | true     | User's full name.                               | "John Doe"       |
            | `email`     | string (format: email) | true     | User's email address.                           | "user@example.com"|
            | `createdAt` | string (date-time)     | true     | Timestamp of creation in ISO 8601 format.       | "2023-10-27T10:30:00Z" |
            | ...         | ...                    | ...      | ...                                             | ...              |
        - Example of the full JSON object.

## 5. Appendix

### 5.1 Glossary of Terms
    - Definitions of API-specific or domain-specific terms.

### 5.2 Deprecated APIs (If any)
    - List of endpoints or API versions that are deprecated, with information on alternatives and sunset dates.
```
