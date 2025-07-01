# Service & Booking Management Service - API Reference Example

This document provides a detailed example of API documentation for key endpoints within the **Service & Booking Management Service**. It illustrates how to apply the structure defined in the main `API-Reference-Outline.md`.

## Service & Booking Management API

Provides endpoints for managing service provider profiles, service listings, service discovery, bookings, and reviews. Base path for these endpoints: `/api/v1`

---

### 1. Create Service Offering

*   **Endpoint:** `POST /services/me/offerings`
*   **Summary:** Allows an authenticated Service Provider to create a new service they offer.
*   **Description:** Creates a new service listing associated with the currently authenticated Service Provider. The service is linked to a category and includes details like name, description, price, and duration.
*   **Tags:** `Services`, `ServiceProviders`
*   **Parameters:** None (Path or Query).
*   **Request Body:**
    *   **Content Type:** `application/json`
    *   **Schema:** `ServiceOfferingCreateRequest` (see Data Models section at the end)
    *   **Example Value:**
        ```json
        {
            "categoryId": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
            "name": "Deep Tissue Massage (60 min)",
            "description": "A 60-minute session focusing on deep muscle layers to relieve chronic tension.",
            "price": 750.00,
            "durationMinutes": 60,
            "isActive": true
        }
        ```
*   **Responses:**
    *   **`201 Created`:**
        *   **Description:** Service offering created successfully.
        *   **Content Type:** `application/json`
        *   **Schema:** `ServiceOfferingResponse` (see Data Models section at the end)
        *   **Headers:**
            *   `Location`: URL to the newly created service offering (e.g., `/api/v1/services/offerings/{serviceId}`).
        *   **Example Value:**
            ```json
            {
                "success": true,
                "data": {
                    "serviceId": "s1o2f3f4-e5r6-7890-1234-567890abcdef",
                    "spUserId": "sp_user_guid_here",
                    "categoryId": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
                    "name": "Deep Tissue Massage (60 min)",
                    "description": "A 60-minute session focusing on deep muscle layers to relieve chronic tension.",
                    "price": 750.00,
                    "durationMinutes": 60,
                    "isActive": true,
                    "createdAt": "2023-10-28T10:00:00Z",
                    "updatedAt": "2023-10-28T10:00:00Z"
                },
                "message": "Service offering created successfully."
            }
            ```
    *   **`400 Bad Request` (Validation Error):**
        *   **Description:** Invalid input provided (e.g., missing required fields, invalid `categoryId`, negative price).
        *   **Content Type:** `application/json`
        *   **Schema:** Standard Error Response Format (see main API Reference Outline)
        *   **Example Value:**
            ```json
            {
                "success": false,
                "error": {
                    "code": "VALIDATION_ERROR",
                    "message": "One or more validation errors occurred.",
                    "details": [
                        { "field": "name", "message": "Service name is required." },
                        { "field": "price", "message": "Price must be a non-negative value." }
                    ]
                }
            }
            ```
    *   **`401 Unauthorized`:**
        *   **Description:** Authentication token is missing or invalid.
        *   **Schema:** Standard Error Response Format.
    *   **`403 Forbidden`:**
        *   **Description:** Authenticated user is not a Service Provider or does not have permission to create offerings.
        *   **Schema:** Standard Error Response Format.
*   **Security:**
    *   Authentication: JWT Bearer token required.
    *   Authorization: Requires `ServiceProvider` role.
*   **Notes / Business Logic:**
    *   The `spUserId` for the offering is derived from the authenticated user's JWT.
    *   `categoryId` must refer to an existing, valid `ServiceCategory`.

---

### 2. Get Service Offering Details

*   **Endpoint:** `GET /services/offerings/{serviceId}`
*   **Summary:** Retrieves details of a specific service offering.
*   **Description:** Fetches comprehensive information about a service offering by its unique ID. This can be used by anyone to view service details.
*   **Tags:** `Services`
*   **Parameters:**
    *   **Path Parameters:**
        | Name        | In   | Description                             | Required | Type          | Example                                  |
        |-------------|------|-----------------------------------------|----------|---------------|------------------------------------------|
        | `serviceId` | path | ID of the service offering to retrieve. | true     | string (uuid) | `s1o2f3f4-e5r6-7890-1234-567890abcdef` |
*   **Request Body:** None.
*   **Responses:**
    *   **`200 OK`:**
        *   **Description:** Service offering details retrieved successfully.
        *   **Content Type:** `application/json`
        *   **Schema:** `ServiceOfferingResponse` (see Data Models section at the end)
        *   **Example Value:**
            ```json
            {
                "success": true,
                "data": {
                    "serviceId": "s1o2f3f4-e5r6-7890-1234-567890abcdef",
                    "spUserId": "sp_user_guid_here",
                    "categoryId": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
                    "name": "Deep Tissue Massage (60 min)",
                    "description": "A 60-minute session focusing on deep muscle layers to relieve chronic tension.",
                    "price": 750.00,
                    "durationMinutes": 60,
                    "isActive": true,
                    "createdAt": "2023-10-28T10:00:00Z",
                    "updatedAt": "2023-10-28T10:00:00Z",
                    "providerInfo": {
                        "businessName": "Wellness Pro",
                        "averageRating": 4.8,
                        "totalReviews": 25
                    },
                    "categoryInfo": {
                        "categoryName": "Massage Therapy"
                    }
                }
            }
            ```
    *   **`404 Not Found`:**
        *   **Description:** Service offering with the specified ID not found.
        *   **Content Type:** `application/json`
        *   **Schema:** Standard Error Response Format.
        *   **Example Value:**
            ```json
            {
                "success": false,
                "error": {
                    "code": "RESOURCE_NOT_FOUND",
                    "message": "Service offering with ID 's1o2f3f4-e5r6-7890-1234-567890abcdef' not found."
                }
            }
            ```
*   **Security:**
    *   Authentication: Optional. Publicly accessible to allow browsing of services.
*   **Notes / Business Logic:**
    *   The response should ideally include some basic, non-sensitive information about the service provider (e.g., business name, overall rating) and the service category name for better user experience.

---

### 3. Create Booking

*   **Endpoint:** `POST /bookings`
*   **Summary:** Allows an authenticated Resident to create a new booking for a service.
*   **Description:** Creates a booking request for a specific service offering from a service provider at a chosen date and time.
*   **Tags:** `Bookings`, `Residents`
*   **Parameters:** None (Path or Query).
*   **Request Body:**
    *   **Content Type:** `application/json`
    *   **Schema:** `BookingCreateRequest` (see Data Models section at the end)
    *   **Example Value:**
        ```json
        {
            "serviceId": "s1o2f3f4-e5r6-7890-1234-567890abcdef",
            "spUserId": "sp_user_guid_here",
            "bookingTime": "2023-11-15T14:00:00Z",
            "specialRequirements": "Please ensure to use hypoallergenic products if possible.",
            "serviceLocation": {
                "street": "123 Resident St",
                "city": "Johannesburg",
                "postalCode": "2000",
                "notes": "Apartment 5, near the park."
            }
        }
        ```
*   **Responses:**
    *   **`201 Created`:**
        *   **Description:** Booking created successfully (status likely 'Pending' awaiting SP confirmation).
        *   **Content Type:** `application/json`
        *   **Schema:** `BookingResponse` (see Data Models section at the end)
        *   **Headers:**
            *   `Location`: URL to the newly created booking (e.g., `/api/v1/bookings/{bookingId}`).
        *   **Example Value:**
            ```json
            {
                "success": true,
                "data": {
                    "bookingId": "b1k2n3g4-e5f6-7890-1234-567890abcdef",
                    "residentUserId": "resident_user_guid_here",
                    "spUserId": "sp_user_guid_here",
                    "serviceId": "s1o2f3f4-e5r6-7890-1234-567890abcdef",
                    "bookingTime": "2023-11-15T14:00:00Z",
                    "status": "Pending",
                    "agreedPrice": 750.00,
                    "specialRequirements": "Please ensure to use hypoallergenic products if possible.",
                    "serviceLocation": {
                        "street": "123 Resident St",
                        "city": "Johannesburg",
                        "postalCode": "2000",
                        "notes": "Apartment 5, near the park."
                    },
                    "serviceCompletionIdCode": null,
                    "createdAt": "2023-10-28T11:00:00Z",
                    "updatedAt": "2023-10-28T11:00:00Z"
                },
                "message": "Booking created successfully. Awaiting provider confirmation."
            }
            ```
    *   **`400 Bad Request` (Validation Error):**
        *   **Description:** Invalid input (e.g., `serviceId` not found, `bookingTime` in the past, missing required location fields).
        *   **Schema:** Standard Error Response Format.
    *   **`401 Unauthorized`:**
        *   **Description:** Authentication token is missing or invalid.
        *   **Schema:** Standard Error Response Format.
    *   **`403 Forbidden`:**
        *   **Description:** Authenticated user is not a Resident.
        *   **Schema:** Standard Error Response Format.
    *   **`422 Unprocessable Entity` (Business Logic Error):**
        *   **Description:** Valid request format, but business logic prevents booking (e.g., insufficient tokens for escrow if applicable, SP not available at that time, service inactive).
        *   **Schema:** Standard Error Response Format.
        *   **Example Value:**
            ```json
            {
                "success": false,
                "error": {
                    "code": "BOOKING_SLOT_UNAVAILABLE",
                    "message": "The selected service provider is not available at the requested time."
                }
            }
            ```
*   **Security:**
    *   Authentication: JWT Bearer token required.
    *   Authorization: Requires `Resident` role.
*   **Notes / Business Rules:**
    *   The `residentUserId` is derived from the authenticated user's JWT.
    *   The `agreedPrice` is fetched from the `ServicesOffered` table at the time of booking.
    *   This action may trigger a token escrow action in the Token & Wallet Service.
    *   A notification should be sent to the Service Provider.

---
## Data Models / Schemas (for Service & Booking Management API Examples)

*(These would typically be part of a global "Data Models / Schemas" section in a full API reference, or defined inline in an OpenAPI specification.)*

#### `ServiceOfferingCreateRequest`
*   **Description:** Payload for creating a new service offering.
*   **Properties:**
    | Field Name        | Data Type     | Required | Description                                     | Example                                  |
    |-------------------|---------------|----------|-------------------------------------------------|------------------------------------------|
    | `categoryId`      | string (uuid) | true     | ID of the service category.                     | `c1b2a3d4-e5f6-7890-1234-567890abcdef` |
    | `name`            | string        | true     | Name of the service offering.                   | "Deep Tissue Massage (60 min)"           |
    | `description`     | string        | false    | Detailed description of the service.            | "Focuses on deep muscle layers..."       |
    | `price`           | number (decimal, format: `#.00`)| true     | Price of the service.                           | `750.00`                                 |
    | `durationMinutes` | integer       | false    | Optional: Estimated duration in minutes.        | `60`                                     |
    | `isActive`        | boolean       | false    | Whether the service is initially active. Default: `true`. | `true`                                   |

#### `ServiceOfferingResponse`
*   **Description:** Represents a service offering.
*   **Properties:**
    | Field Name        | Data Type     | Description                                     | Example                                  |
    |-------------------|---------------|-------------------------------------------------|------------------------------------------|
    | `serviceId`       | string (uuid) | Unique ID of the service offering.              | `s1o2f3f4-e5r6-7890-1234-567890abcdef` |
    | `spUserId`        | string (uuid) | ID of the Service Provider.                     | `sp_user_guid_here`                      |
    | `categoryId`      | string (uuid) | ID of the service category.                     | `c1b2a3d4-e5f6-7890-1234-567890abcdef` |
    | `name`            | string        | Name of the service offering.                   | "Deep Tissue Massage (60 min)"           |
    | `description`     | string        | Detailed description of the service.            | "Focuses on deep muscle layers..."       |
    | `price`           | number (decimal)| Price of the service.                           | `750.00`                                 |
    | `durationMinutes` | integer       | Optional: Estimated duration in minutes.        | `60`                                     |
    | `isActive`        | boolean       | Whether the service is currently active.        | `true`                                   |
    | `createdAt`       | string (date-time)| Timestamp of creation.                        | "2023-10-28T10:00:00Z"                   |
    | `updatedAt`       | string (date-time)| Timestamp of last update.                     | "2023-10-28T10:00:00Z"                   |
    | `providerInfo`    | object        | (Optional) Brief info about the provider.        | `{ "businessName": "Wellness Pro", "averageRating": 4.8, "totalReviews": 25 }` |
    | `categoryInfo`    | object        | (Optional) Brief info about the category.       | `{ "categoryName": "Massage Therapy" }`  |

#### `ServiceLocationInput`
*   **Description:** Object representing service location details for input.
*   **Properties:**
    | Field Name   | Data Type | Required | Description      | Example          |
    |--------------|-----------|----------|------------------|------------------|
    | `street`     | string    | true     | Street address.  | "123 Resident St"|
    | `city`       | string    | true     | City.            | "Johannesburg"   |
    | `postalCode` | string    | false    | Postal code.     | "2000"           |
    | `notes`      | string    | false    | Additional location notes (e.g., apartment number, directions). | "Unit 5, Blue Complex" |
    | `latitude`   | number (double)| false  | Optional: Latitude for precise location. | `-26.2041`       |
    | `longitude`  | number (double)| false  | Optional: Longitude for precise location.| `28.0473`        |

#### `BookingCreateRequest`
*   **Description:** Payload for creating a new booking.
*   **Properties:**
    | Field Name        | Data Type     | Required | Description                                     | Example                                  |
    |-------------------|---------------|----------|-------------------------------------------------|------------------------------------------|
    | `serviceId`       | string (uuid) | true     | ID of the service offering to book.             | `s1o2f3f4-e5r6-7890-1234-567890abcdef` |
    | `spUserId`        | string (uuid) | true     | ID of the Service Provider for the service.     | `sp_user_guid_here`                      |
    | `bookingTime`     | string (date-time)| true   | Desired date and time for the booking (ISO 8601 UTC).| "2023-11-15T14:00:00Z"                   |
    | `specialRequirements`| string     | false    | Any special requests from the resident.         | "Hypoallergenic products please."        |
    | `serviceLocation` | `ServiceLocationInput` | true     | Location where service is to be performed.      | (see `ServiceLocationInput` example)   |

#### `BookingResponse`
*   **Description:** Represents a booking.
*   **Properties:**
    | Field Name        | Data Type     | Description                                     | Example                                  |
    |-------------------|---------------|-------------------------------------------------|------------------------------------------|
    | `bookingId`       | string (uuid) | Unique ID of the booking.                       | `b1k2n3g4-e5f6-7890-1234-567890abcdef` |
    | `residentUserId`  | string (uuid) | ID of the Resident who made the booking.        | `resident_user_guid_here`                |
    | `spUserId`        | string (uuid) | ID of the Service Provider.                     | `sp_user_guid_here`                      |
    | `serviceId`       | string (uuid) | ID of the service offering booked.              | `s1o2f3f4-e5r6-7890-1234-567890abcdef` |
    | `bookingTime`     | string (date-time)| Scheduled date and time (ISO 8601 UTC).       | "2023-11-15T14:00:00Z"                   |
    | `status`          | string        | Current status of the booking.                  | "Pending"                                |
    | `agreedPrice`     | number (decimal)| Price agreed upon at the time of booking.       | `750.00`                                 |
    | `specialRequirements`| string     | Special requests from the resident.             | "Hypoallergenic products please."        |
    | `serviceLocation` | `ServiceLocationInput` | Location where service is to be performed.      | (see `ServiceLocationInput` structure)   |
    | `serviceCompletionIdCode`| string  | Unique code for service completion (null until set). | `null` or `AB12CD34`                   |
    | `createdAt`       | string (date-time)| Timestamp of booking creation.                | "2023-10-28T11:00:00Z"                   |
    | `updatedAt`       | string (date-time)| Timestamp of last booking update.             | "2023-10-28T11:05:00Z"                   |
    | `serviceDetails`  | object        | (Optional) Brief info about the service.        | `{ "serviceName": "Deep Tissue Massage", "durationMinutes": 60 }`|
    | `providerDetails` | object        | (Optional) Brief info about the provider.        | `{ "businessName": "Wellness Pro" }`     |
    | `residentDetails` | object        | (Optional) Brief info about the resident.        | `{ "fullName": "Jane Resident" }`        |

---
This example provides a template for how each endpoint and data model can be documented for the Service & Booking Management Service. A complete API reference would include many more endpoints (e.g., for updating/deleting services, listing services with filters, managing SP profiles, handling reviews, service requests, bids, availability, etc.).
