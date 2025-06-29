# STAR Community App - Backend Architecture Plan (MVP Stage)

## 1. Executive Summary

The backend architecture for the STAR Community App MVP is designed to be robust, scalable, and secure, leveraging a microservices pattern to ensure modularity and maintainability. The core technology stack will be .NET (ASP.NET Core) for API and service development, with PostgreSQL serving as the primary relational database.

Deployment for the MVP stage will be on a single Linux VPS (4 cores, 4GB RAM, 100GB storage), utilizing Docker Compose to containerize and manage all backend components. This includes the individual microservices, the PostgreSQL database, and an Apache web server acting as a reverse proxy and handling SSL termination. An API Gateway, potentially using a .NET-based solution like Ocelot, will manage and route client requests to the appropriate microservices, providing a unified entry point to the system.

This approach, even on a single server, allows for clear separation of concerns, independent development and deployment of services (in principle, preparing for future scaling), and efficient resource utilization within the given constraints. The architecture prioritizes MVP functionality while laying a foundation for future growth and adherence to enterprise-grade security standards, especially for financial transactions and data protection.

## 2. System Architecture Diagram

```mermaid
graph TD
    subgraph "Client Tier"
        MobileApp[Mobile App (.NET MAUI/React Native/Flutter)]
        WebApp[Web App (ASP.NET Blazor/React/Angular)]
    end

    subgraph "API Gateway Tier (Single VPS)"
        APIGateway[API Gateway (Ocelot on .NET)]
    end

    subgraph "Service Tier (Microservices on Single VPS via Docker Compose)"
        UserService[User Management Service (.NET)]
        ServiceService[Service & Booking Service (.NET)]
        TokenService[Token & Wallet Service (.NET)]
        PaymentService[Payment Integration Service (.NET)]
        CommunityService[Community (Projects & Causes) Service (.NET)]
        NotificationService[Notification Service (.NET)]
    end

    subgraph "Data Tier (Single VPS via Docker Compose)"
        PostgreSQL[PostgreSQL Database]
    end

    subgraph "External Services"
        PaymentGateway[Payment Gateway (e.g., PayFast)]
        EmailService[Email Service (e.g., SendGrid)]
        SMSService[SMS Service (e.g., Twilio)]
        PushNotificationService[Push Notification Service (FCM)]
    end

    MobileApp --> APIGateway
    WebApp --> APIGateway

    APIGateway --> UserService
    APIGateway --> ServiceService
    APIGateway --> TokenService
    APIGateway --> PaymentService
    APIGateway --> CommunityService
    APIGateway --> NotificationService

    UserService --> PostgreSQL
    ServiceService --> PostgreSQL
    TokenService --> PostgreSQL
    PaymentService --> PostgreSQL
    CommunityService --> PostgreSQL
    NotificationService --> PostgreSQL

    PaymentService --> PaymentGateway
    NotificationService --> EmailService
    NotificationService --> SMSService
    NotificationService --> PushNotificationService

    %% Styling (optional, for better readability if rendered)
    classDef client fill:#lightblue,stroke:#333,stroke-width:2px;
    classDef gateway fill:#lightgreen,stroke:#333,stroke-width:2px;
    classDef service fill:#lightyellow,stroke:#333,stroke-width:2px;
    classDef data fill:#orange,stroke:#333,stroke-width:2px;
    classDef external fill:#lightgrey,stroke:#333,stroke-width:2px;

    class MobileApp,WebApp client;
    class APIGateway gateway;
    class UserService,ServiceService,TokenService,PaymentService,CommunityService,NotificationService service;
    class PostgreSQL data;
    class PaymentGateway,EmailService,SMSService,PushNotificationService external;
```

## 3. Microservices Breakdown

The STAR Community App's backend will be structured as a set of microservices. This approach promotes modularity, independent scalability (future) and maintainability. For the MVP deployed on a single VPS, these services will run as distinct containers managed by Docker Compose.

**Identified Microservices:**

1.  **User Management Service (.NET)**
    *   **Responsibilities:** User registration, authentication (JWT), profile management, verification processes, RBAC info provider, admin user moderation functions.
    *   **Boundaries:** All aspects of user identity, authentication, authorization, and core profile data.
    *   **Key Entities:** `Users`, `Roles`, `UserRoles`, `VerificationDocuments`, `Addresses`.

2.  **Service & Booking Management Service (.NET)**
    *   **Responsibilities:** SP profile management, service listings, service discovery, booking creation/management, service request/bidding, rating/review system, Service ID (SI) management.
    *   **Boundaries:** Service definitions, SP profiles (service-related), booking lifecycle, reviews. Does not handle token transactions directly.
    *   **Key Entities:** `ServiceProviders`, `ServicesOffered`, `ServiceCategories`, `Bookings`, `ServiceRequests`, `Bids`, `Reviews`, `AvailabilitySlots`.

3.  **Token & Wallet Service (.NET)**
    *   **Responsibilities:** Digital wallet management, token balance operations, token purchase/redemption processing (initiates with Payment Service), subscription management, transaction history, escrow functionality.
    *   **Boundaries:** Manages STAR token lifecycle and ledger. Relies on Payment Service for monetary transactions.
    *   **Key Entities:** `Wallets`, `TokenTransactions`, `Subscriptions`, `EscrowAccounts`.

4.  **Payment Integration Service (.NET)**
    *   **Responsibilities:** Integration with external payment gateways, handling payment initiation for purchases, processing incoming webhooks, initiating payouts for redemptions, secure management of gateway configurations.
    *   **Boundaries:** Anti-corruption layer for external financial institutions. Does not store token balances.
    *   **Key Entities:** `PaymentRecords`, `GatewayConfigurations`.

5.  **Community Management Service (.NET)**
    *   **Responsibilities:** STAR Projects (creation, funding, participation, SP selection, QA), STAR Causes (application, approval, donations, tracking), Cause Champion role management.
    *   **Boundaries:** Manages collaborative community initiatives. Relies on Token Service for financial aspects.
    *   **Key Entities:** `Projects`, `ProjectMembers`, `ProjectContributions`, `Causes`, `CauseDonations`, `CauseApplications`, `CauseChampions`.

6.  **Notification Service (.NET)**
    *   **Responsibilities:** Sending emails, SMS, and push notifications. Managing notification templates. User notification preferences (future).
    *   **Boundaries:** Centralized service for all outgoing communications.
    *   **Key Entities:** `NotificationTemplates`, `NotificationLogs`.

**Inter-service Communication Patterns:**

*   **Synchronous (REST APIs via API Gateway):** Primary method for client-to-service and some service-to-service communication.
*   **Asynchronous (Optional for MVP):** Message queues (e.g., RabbitMQ) or .NET background tasks for decoupled operations. Deferred for MVP if resource-heavy, favoring direct calls or simple background tasks initially.

**API Gateway Configuration:**

*   **Ocelot (.NET-based):** Recommended. Configured via JSON for routing, authentication, rate limiting. Runs as a Docker container.

**Service Discovery and Load Balancing:**

*   **Service Discovery (Docker Compose):** Internal DNS allows containers to reach each other by service name.
*   **Load Balancing (MVP):** Not applicable with single instances. API Gateway routes to the single instance. Architecture prepares for future load balancing.

## 4. Database Architecture

PostgreSQL is selected, running as a container on the single VPS.

**Database Design Strategy:**

*   **MVP Approach: Single PostgreSQL Instance with Schema-per-Service.** A single PostgreSQL instance hosts multiple schemas (e.g., `user_service_schema`, `booking_service_schema`). This provides logical separation and access control while being resource-efficient for the VPS. Fallback to table prefixing if schemas add too much ORM complexity for MVP.

### 4.1 High-Level Entity-Relationship Diagram (ERD - Mermaid Syntax)

This ERD provides a conceptual overview of the main entities within each microservice domain and their primary relationships. For detailed table structures and all relationships, refer to the dedicated Database Schema Document (see Section 18).

```mermaid
erDiagram
    USER_SERVICE {
        Users ||--o{ Addresses : has
        Users ||--o{ VerificationDocuments : has
        Users ||--|{ Roles : assigned_via_UserRoles
        UserRoles }|--|| Roles : maps
        UserRoles }|--|| Users : maps
    }

    SERVICE_BOOKING_SERVICE {
        Users ||--o{ ServiceProviders : "is_a (profile)"
        ServiceProviders ||--o{ ServicesOffered : offers
        ServicesOffered ||--|{ ServiceCategories : belongs_to
        Users ||--o{ Bookings : creates_as_resident
        ServiceProviders ||--o{ Bookings : assigned_to_as_sp
        Bookings ||--|{ ServicesOffered : for_service
        Bookings ||--o{ Reviews : has
    }

    TOKEN_WALLET_SERVICE {
        Users ||--|| Wallets : owns
        Wallets ||--o{ TokenTransactions : has_many
        Users ||--o{ Subscriptions : has
        Bookings ||--o{ TokenTransactions : "related_to (escrow)"
        Projects ||--o{ TokenTransactions : "related_to (funding)"
        Causes ||--o{ TokenTransactions : "related_to (donation)"
    }

    PAYMENT_SERVICE {
        TokenTransactions ||--o{ PaymentRecords : "logs_gateway_interaction_for"
    }

    COMMUNITY_SERVICE {
        Users ||--o{ Projects : creates
        Users ||--o{ ProjectMembers : joins
        ProjectMembers }|--|| Projects : member_of
        Users ||--o{ CauseApplications : submits
        Users ||--o{ CauseDonations : donates_to
        CauseDonations }|--|| Causes : for_cause
        Causes ||--|{ Users : "managed_by (Champion)"
    }

    Users {
        string UserID PK
        string FullName
        string Email
        string PhoneNumber
        string Role
    }
    ServiceProviders {
        string SP_UserID PK FK
        string BusinessName
    }
    Bookings {
        string BookingID PK
        string ResidentUserID FK
        string SP_UserID FK
        datetime BookingTime
        string Status
    }
    Wallets {
        string WalletID PK
        string UserID FK
        decimal TokenBalance
    }
    TokenTransactions {
        string TransactionID PK
        string WalletID FK
        string Type
        decimal Amount
    }
    Projects {
        string ProjectID PK
        string CreatorUserID FK
        string Title
        decimal TokenTargetAmount
    }
    Causes {
        string CauseID PK
        string ApplicantUserID FK
        string Title
        decimal FundingTarget
    }

    %% Relationships between service contexts (conceptual)
    USER_SERVICE ||--|| SERVICE_BOOKING_SERVICE : "User_acts_as_SP_or_Resident"
    USER_SERVICE ||--|| TOKEN_WALLET_SERVICE : "User_owns_Wallet"
    USER_SERVICE ||--|| COMMUNITY_SERVICE : "User_participates_in_Community"
    TOKEN_WALLET_SERVICE ||--|| PAYMENT_SERVICE : "TokenTransaction_triggers_Payment"

```

### 4.2 Key Table Structures (Illustrative Examples)

The following are simplified examples of key tables. Comprehensive schemas, including all columns, data types, constraints, and detailed relationships, should be maintained in a separate Database Schema Document.

**User Management Service:**

*   **`Users`**
    *   `UserID (PK, UUID/GUID)`
    *   `FullName (VARCHAR(255), NOT NULL)`
    *   `PhoneNumber (VARCHAR(20), UNIQUE, NOT NULL)`
    *   `Email (VARCHAR(255), UNIQUE, NOT NULL)`
    *   `PasswordHash (VARCHAR(512), NOT NULL)`
    *   `Role (VARCHAR(50), NOT NULL)` (e.g., 'Resident', 'ServiceProvider', 'Agent')
    *   `IsVerified (BOOLEAN, DEFAULT FALSE)`
    *   `CreatedAt (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)`
    *   `UpdatedAt (TIMESTAMP)`
*   **`Addresses`**
    *   `AddressID (PK, UUID/GUID)`
    *   `UserID (FK, References Users.UserID, NOT NULL)`
    *   `Street (VARCHAR(255))`
    *   `City (VARCHAR(100))`
    *   `PostalCode (VARCHAR(10))`
    *   `LocationCoordinates (POINT)` (Using PostGIS extension if available, or two DECIMAL fields for lat/lon)

**Service & Booking Management Service:**

*   **`ServiceProviders`** (Profile extension of Users)
    *   `SP_UserID (PK, FK, References Users.UserID, NOT NULL)`
    *   `BusinessName (VARCHAR(255), NOT NULL)`
    *   `BusinessDescription (TEXT)`
    *   `OperatingHours (JSONB)`
*   **`ServicesOffered`**
    *   `ServiceID (PK, UUID/GUID)`
    *   `SP_UserID (FK, References ServiceProviders.SP_UserID, NOT NULL)`
    *   `CategoryID (FK, References ServiceCategories.CategoryID, NOT NULL)`
    *   `Name (VARCHAR(255), NOT NULL)`
    *   `Price (DECIMAL(10,2))`
*   **`Bookings`**
    *   `BookingID (PK, UUID/GUID)`
    *   `ResidentUserID (FK, References Users.UserID, NOT NULL)`
    *   `SP_UserID (FK, References ServiceProviders.SP_UserID, NOT NULL)`
    *   `ServiceID (FK, References ServicesOffered.ServiceID, NOT NULL)`
    *   `BookingTime (TIMESTAMP, NOT NULL)`
    *   `Status (VARCHAR(50), NOT NULL)` (e.g., 'Pending', 'Confirmed', 'Completed', 'Cancelled')
    *   `ServiceID_Code (VARCHAR(10), UNIQUE)`

**Token & Wallet Service:**

*   **`Wallets`**
    *   `WalletID (PK, UUID/GUID)`
    *   `UserID (FK, References Users.UserID, UNIQUE, NOT NULL)`
    *   `TokenBalance (DECIMAL(18,4), DEFAULT 0.00, NOT NULL)`
    *   `UpdatedAt (TIMESTAMP)`
*   **`TokenTransactions`**
    *   `TransactionID (PK, UUID/GUID)`
    *   `WalletID (FK, References Wallets.WalletID, NOT NULL)`
    *   `Type (VARCHAR(50), NOT NULL)` (e.g., 'Purchase', 'Redemption', 'Escrow', 'Release')
    *   `Amount (DECIMAL(18,4), NOT NULL)`
    *   `RelatedEntityID (UUID/GUID, NULLABLE)` (e.g., BookingID, ProjectID)
    *   `Timestamp (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)`

**Data Consistency and Transaction Management:**

*   **Local ACID Transactions:** Within each service for its own data.
*   **Distributed Transactions (SAGA Pattern - Conceptual for MVP):** For operations spanning multiple services. For MVP, critical financial operations might use carefully orchestrated synchronous calls if sharing the same DB instance, or rely on eventual consistency for less critical flows.

**Database Connection Pooling:**

*   Handled by .NET Core's `Npgsql` driver and Entity Framework Core defaults.

**Indexing Strategies for Performance:**

*   Indexes on Primary Keys, Foreign Keys, and common query fields (e.g., user emails, booking statuses, entity IDs for lookups). Specific indexing strategies will be detailed in the separate Database Schema Document.

## 5. Security & Data Protection

**Authentication and Authorization:**

*   **JWT (JSON Web Tokens):** For stateless authentication. Issued by User Management Service.
*   **RBAC (Role-Based Access Control):** Roles included in JWT claims, enforced by services.

**Data Encryption:**

*   **In Transit: SSL/TLS** for all external communication (HTTPS). Apache handles SSL termination.
*   **At Rest:** Column-level encryption (e.g., `pgcrypto` AES-256) for highly sensitive data like bank account details. Secure storage of secrets (API keys, JWT secret) via environment variables.

**API Security Best Practices:**

*   HTTPS only, input validation, output encoding, parameterized queries (via EF Core), principle of least privilege, generic error messages.

**GDPR/POPIA Compliance Considerations (MVP Focus):**

*   Data minimization, consent management (via T&C for MVP), documented process for data access/erasure requests, clear privacy policy.

**Security Headers and CORS:**

*   Standard HTTP security headers (HSTS, X-Content-Type-Options, X-Frame-Options) via Apache/Ocelot.
*   Proper CORS configuration to allow requests only from authorized frontend domains.

**Rate Limiting and DDoS Protection:**

*   Basic rate limiting at API Gateway (Ocelot) or ASP.NET Core middleware.
*   For DDoS on single VPS: `ufw` firewall rules. **Highly recommend using Cloudflare (free tier acceptable for MVP) for significant DDoS mitigation and WAF capabilities.**

## 6. Financial Transaction Security

Focus on offloading sensitive payment card data handling to PCI DSS compliant payment gateways.

**PCI DSS Compliance:**

*   **No Direct Handling of Card Data:** Backend will not store, process, or transmit raw card data.
*   Achieved via gateway-hosted pages or client-side tokenization with the gateway.

**Payment Gateway Integration Architecture (e.g., PayFast):**

1.  Client initiates purchase in-app.
2.  STAR backend prepares transaction details (amount, reference).
3.  Client redirects to/uses gateway's secure form (user enters card details directly to gateway).
4.  Gateway processes payment.
5.  Gateway sends asynchronous webhook (ITN) to STAR backend (Payment Service) with status.
6.  STAR backend validates ITN and, if successful, Payment Service instructs Token Service to issue tokens.

**Secure Payment Processing Workflows:**

*   **Token Purchase:** As above.
*   **Token Redemption:** User requests redemption, Token Service validates, user provides bank details (stored encrypted), Payment Service initiates payout (via gateway API if available, or admin triggers manual transfer). Consider OTP for large redemptemons.

**Transaction Logging and Audit Trails:**

*   Comprehensive and immutable logging of all token movements (TokenService) and gateway interactions (PaymentService).

**Fraud Detection and Prevention (MVP Focus):**

*   Basic checks: Velocity checks (transaction frequency), unusual redemption patterns, additional verification for large transactions.

**Financial Data Encryption and Tokenization:**

*   Bank account details encrypted at rest (AES-256).
*   STAR tokens are digital ledger entries; security relies on overall system integrity.

## 7. Performance & Scalability (MVP Focus)

Strategies focus on efficient resource use on the single VPS.

**Caching Strategies:**

*   **In-Memory Caching (.NET `IMemoryCache`):** Within microservices for frequently accessed, rarely changing data (e.g., service categories). Limited by VPS RAM.
*   **Client-Side Caching:** Via HTTP cache headers.
*   **Redis (Conditional for MVP):** Introduce only if clear bottleneck and VPS RAM allows. Start with in-memory.

**Database Query Optimization:**

*   Efficient EF Core: Projections, avoid N+1, async operations.
*   Proper indexing.

**Connection Pooling:**

*   Default `Npgsql` capabilities.

**Resource Allocation within VPS Constraints:**

*   Docker Compose container resource limits (memory).
*   Continuous monitoring (htop, docker stats).
*   Vertical scaling (upgrading VPS) as the first response to resource exhaustion.

**Monitoring and Performance Metrics (MVP):**

*   Application logs: Request times, error rates.
*   Basic health checks (`/health` endpoints).
*   System-level monitoring (Linux tools, `docker stats`).

## 8. Data Management

**Database Backup and Recovery:**

*   **Tool:** `pg_dump` for logical backups.
*   **Schedule:** Daily full backups via cron job.
*   **Storage:** Local (separate partition) + **Highly Recommended: Regular transfer to secure off-server storage** (e.g., cloud storage).
*   **Retention (MVP):** 7 daily, 4 weekly, 3-6 monthly.
*   **Recovery:** Documented `pg_restore` process, tested periodically.

**Data Migration and Versioning:**

*   **Entity Framework Core Migrations:** For schema changes, version-controlled with code. Applied on deployment.

**Data Archiving Policies (MVP):**

*   No active archiving for MVP. Future consideration based on data growth.

**Disaster Recovery Planning (DRP - Basic for MVP):**

*   **Focus:** Data recovery from off-server backups. Infrastructure recreation via Docker Compose files and documented server setup steps.
*   **RPO/RTO:** RPO up to 24hrs (daily backups). RTO manual, potentially several hours to a day.

## 9. Communication Services

Handled by a dedicated Notification Service integrating with third-party providers.

**Email Service Integration:**

*   **Purpose:** Transactional emails.
*   **Providers:** SendGrid, Mailgun (free tiers).
*   **Method:** API integration via Notification Service.

**SMS/Notification Services:**

*   **Purpose:** Critical alerts (Service IDs, OTPs).
*   **Providers:** Twilio, Vonage, or local SA providers (e.g., Clickatell).
*   **Method:** API integration via Notification Service.

**Push Notification Services:**

*   **Purpose:** Real-time mobile app updates.
*   **Provider:** Firebase Cloud Messaging (FCM).
*   **Method:** Notification Service acts as app server to FCM.

**Real-time Communication (In-app Chat):**

*   **Technology:** SignalR (ASP.NET Core).
*   **Implementation:** SignalR Hub within a relevant service (e.g., Notification Service or dedicated Chat Service for MVP). Message persistence in DB.

**Message Queuing for Asynchronous Processing (Conditional for MVP):**

*   **Purpose:** Decouple services for non-immediate tasks.
*   **Providers (if used):** RabbitMQ (containerized).
*   **MVP Approach:** Defer if possible to save VPS resources. Use .NET Background Tasks (`IHostedService`) for simpler async needs.

## 10. DevOps & Infrastructure

Focus on Docker Compose for deployment on the single Linux VPS.

**Docker Compose Configuration Structure:**

*   Single `docker-compose.yml` defining:
    *   PostgreSQL database (with persistent volume).
    *   .NET Microservices (UserService, ServiceService, etc.).
    *   API Gateway (Ocelot).
    *   Apache Web Server (reverse proxy).
*   Custom Docker bridge network for inter-service communication.

**Environment Variable Management:**

*   `.env` file (not in version control) for secrets and configurations.
*   `.env.example` file in Git.

**Logging and Monitoring Setup (MVP):**

*   Container logging to `stdout`/`stderr` (Docker default).
*   .NET services log to console (e.g., Serilog).
*   Log rotation for Docker's `json-file` driver.
*   Manual inspection via `docker-compose logs`. System tools (`htop`, `docker stats`).

**Health Checks and Service Restart Policies:**

*   `/health` endpoints in each .NET service.
*   Docker Compose health checks defined in `docker-compose.yml`.
*   `restart: unless-stopped` or `restart: always` policy for services.

**CI/CD Pipeline Considerations (MVP & Future):**

*   **MVP Deployment:** Manual/scripted (pull code, `docker-compose build`, `docker-compose up -d`).
*   **Future:** Automated CI/CD (Jenkins, GitLab CI, GitHub Actions) to build, test, push images to registry, and deploy.

## 11. Payment Integration

Secure integration with payment gateways, offloading sensitive card data.

**Recommended Payment Gateway Providers (South Africa):**

*   PayFast, PayGate. (Example: PayFast).

**Payment Processing Architecture (Token Purchase):**

1.  User initiates purchase in-app.
2.  STAR backend prepares transaction (amount, internal ID).
3.  Client redirects to/uses PayFast's secure payment page/form.
4.  User pays directly on PayFast.
5.  PayFast sends ITN (webhook) to STAR backend (`PaymentService`).
6.  `PaymentService` validates ITN, then instructs `TokenService` to issue tokens.

**Webhook Handling:**

*   Secure HTTPS endpoint in `PaymentService`.
*   Validate webhook integrity (e.g., signature check or callback to gateway).
*   Idempotent handlers. Respond quickly (200 OK), process logic asynchronously if needed.

**Refund and Chargeback Management:**

*   **Refunds (MVP):** Admin-initiated via payment gateway's merchant portal. Record manually in STAR system.
*   **Chargebacks:** Handled via payment gateway. STAR platform needs policy for associated accounts/tokens.

**Multi-currency Support:**

*   **MVP:** ZAR (South African Rand) focus.
*   **Future:** Requires gateway support and backend logic for currency management.

## 12. API Design Guidelines

Consistent and well-designed APIs are crucial for maintainability and ease of integration.

*   **RESTful Principles:** Standard HTTP methods (GET, POST, PUT, DELETE) and resource-based URIs.
*   **Versioning:** URL-based (e.g., `/api/v1/`).
*   **Request/Response:** JSON format, camelCase for properties. Consistent success/error structures.
*   **HTTP Status Codes:** Standard usage (200, 201, 400, 401, 403, 404, 500).
*   **Authentication:** JWT in `Authorization: Bearer <token>` header.
*   **Pagination:** For list endpoints (`?page=1&pageSize=20`).
*   **Filtering & Sorting:** Via query parameters.

### 12.1 API Endpoint Documentation Template/Example

This template should be used as a basis for documenting each API endpoint, ideally in a dedicated API Reference (e.g., OpenAPI/Swagger).

*   **Endpoint:** `POST /api/v1/users`
*   **Description:** Registers a new user (Resident, Service Provider, or Agent).
*   **Request Parameters:**
    *   **Path Parameters:** None
    *   **Query Parameters:** None
    *   **Body Parameters:**
        *   `fullName (string, required)`: User's full name.
        *   `email (string, required, format: email)`: User's email address.
        *   `phoneNumber (string, required)`: User's phone number.
        *   `password (string, required, minLength: 8)`: User's password.
        *   `role (string, required, enum: [Resident, ServiceProvider, Agent])`: Role of the user.
        *   `address (object, optional)`: User's address details.
            *   `street (string)`
            *   `city (string)`
*   **Request Body Example:**
    ```json
    {
        "fullName": "John Doe",
        "email": "john.doe@example.com",
        "phoneNumber": "+27820000000",
        "password": "Password123!",
        "role": "Resident",
        "address": {
            "street": "123 Main Rd",
            "city": "Cape Town"
        }
    }
    ```
*   **Successful Response (201 Created):**
    ```json
    {
        "success": true,
        "data": {
            "userId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
            "message": "User registered successfully. Please verify your email."
        },
        "message": "User registration initiated."
    }
    ```
*   **Error Response Examples:**
    *   **400 Bad Request (Validation Error):**
        ```json
        {
            "success": false,
            "error": {
                "code": "VALIDATION_ERROR",
                "message": "One or more validation errors occurred.",
                "details": [
                    { "field": "email", "message": "The email address is already in use." },
                    { "field": "password", "message": "Password must be at least 8 characters long." }
                ]
            }
        }
        ```
    *   **500 Internal Server Error:**
        ```json
        {
            "success": false,
            "error": {
                "code": "INTERNAL_SERVER_ERROR",
                "message": "An unexpected error occurred. Please try again later."
            }
        }
        ```
*   **Authentication:** Public endpoint (no JWT required for registration itself).
*   **Authorization:** N/A.

### 12.2 Primary Resource Endpoints per Microservice (Illustrative)

This is not an exhaustive list but illustrates the main resources managed by each service. Refer to the dedicated API Reference for complete details.

**User Management Service (`/api/v1`):**

*   `POST /auth/register`: Register a new user.
*   `POST /auth/login`: Authenticate user and issue JWT.
*   `POST /auth/refresh-token`: Refresh JWT.
*   `GET /users/me`: Get current authenticated user's profile.
*   `PUT /users/me`: Update current user's profile.
*   `GET /users/{userId}`: Get user profile by ID (Admin).
*   `PUT /users/{userId}/status`: Update user status (Admin - suspend, verify).
*   `POST /users/me/addresses`: Add address for current user.
*   `GET /users/me/addresses`: List addresses for current user.

**Service & Booking Management Service (`/api/v1`):**

*   `POST /service-providers/me/profile`: Create/Update current user's SP profile.
*   `GET /service-providers/{spUserId}/profile`: Get SP profile.
*   `POST /service-providers/me/services`: Add a new service offered by current SP.
*   `GET /services`: List/search all services (filtered).
*   `GET /services/{serviceId}`: Get specific service details.
*   `POST /bookings`: Create a new booking.
*   `GET /bookings/me`: List bookings for current user (resident or SP).
*   `GET /bookings/{bookingId}`: Get booking details.
*   `PUT /bookings/{bookingId}/status`: Update booking status (e.g., confirm, cancel, complete with SI).
*   `POST /bookings/{bookingId}/reviews`: Add a review for a completed booking.
*   `POST /service-requests`: Create a service request (resident).
*   `GET /service-requests`: List available service requests (SPs).
*   `POST /service-requests/{requestId}/bids`: Submit a bid for a service request (SP).

**Token & Wallet Service (`/api/v1`):**

*   `GET /wallet/me`: Get current user's wallet balance and history.
*   `POST /wallet/me/purchase/initiate`: Initiate token purchase.
*   `POST /wallet/me/redeem`: Request token redemption.
*   `GET /subscriptions/me`: Get current user's subscription status.
*   `POST /subscriptions/me`: Subscribe to a token package.

**Payment Integration Service (Primarily internal or specific callbacks):**

*   `POST /payments/webhook/payfast`: Endpoint for PayFast ITN. (Not directly called by clients).

**Community Management Service (`/api/v1`):**

*   `POST /projects`: Create a new STAR Project.
*   `GET /projects`: List/search STAR Projects.
*   `GET /projects/{projectId}`: Get project details.
*   `POST /projects/{projectId}/join`: Join a project (contribute tokens).
*   `POST /causes/apply`: Apply to list a STAR Cause.
*   `GET /causes`: List active STAR Causes.
*   `POST /causes/{causeId}/donate`: Donate to a cause.
*   Admin endpoints for approving causes, managing champions.

**Notification Service (Primarily internal, consumed by other services):**

*   (No direct public API endpoints for clients, other services trigger notifications).

## 13. Performance Optimization Summary

*   **Caching:** In-memory (`IMemoryCache`), client-side HTTP caching. Redis conditional.
*   **Database:** Efficient EF Core usage (projections, avoid N+1), indexing, connection pooling.
*   **Code:** Optimized .NET code, async operations, Release builds.
*   **Resource Limits:** Docker container limits.

## 14. Monitoring & Logging Summary

*   **Logging:** Structured console logging (Serilog), captured by Docker. Configurable levels.
*   **Metrics (MVP):** System (CPU, RAM, disk), Application (latency, error rates), Container resources.
*   **Health Checks:** `/health` endpoints used by Docker Compose.
*   **Log Management (MVP):** Docker log rotation, manual inspection.

## 15. Deployment Strategy Summary

*   **Environment:** Single Linux VPS.
*   **Containerization:** Docker for all components.
*   **Orchestration:** Docker Compose.
*   **Reverse Proxy:** Apache (Docker container) for SSL termination (Let's Encrypt) and routing to API Gateway.
*   **Process (MVP):** Manual/scripted (code pull, Docker build & up). EF Core migrations on startup/manual.

## 16. Risk Assessment Summary

*   **SPOF (Single VPS):** Mitigate with off-server backups, DRP.
*   **Resource Exhaustion:** Mitigate with monitoring, optimization, vertical scaling.
*   **Security Vulnerabilities:** Mitigate with best practices, patching, consider Cloudflare.
*   **Data Loss:** Mitigate with backups, tested recovery.
*   **Payment Gateway Issues:** Mitigate with error handling, manual processes.

## 17. Scalability Roadmap Summary

1.  **Vertical Scaling:** Upgrade VPS.
2.  **DB Optimization/Separation:** Dedicated DB server/managed service. Read replicas.
3.  **Horizontal Service Scaling:** Multiple instances, load balancer.
4.  **Distributed Cache:** Implement Redis.
5.  **CDN:** For static assets/API caching.
6.  **Message Queues:** RabbitMQ/Kafka for async.
7.  **Container Orchestration:** Kubernetes.
8.  **Specialized Services/DBs:** As needed (e.g., Elasticsearch).

## 18. Detailed Supporting Documentation (Recommendations)

While this Backend Architecture Plan provides a comprehensive overview of the system's design for the MVP stage, it is crucial for development, maintenance, and onboarding to create and maintain more detailed supporting documentation. It is strongly recommended that the following documents/specifications be developed and kept up-to-date alongside the evolving codebase:

*   **Detailed Database Schema Document:**
    *   **Purpose:** To provide a complete and unambiguous definition of the database structure.
    *   **Contents:**
        *   Full Entity-Relationship Diagrams (ERDs) for each microservice's database or schema.
        *   Complete table definitions for all tables, including all columns, data types (with precision/length), primary keys, foreign keys, unique constraints, NOT NULL constraints, default values, and check constraints.
        *   Detailed indexing strategies for all tables, explaining the rationale for each index.
        *   Descriptions of any views, stored procedures, or functions if used.
        *   Notes on data retention policies or partitioning strategies (as they evolve).

*   **Comprehensive API Reference (e.g., using OpenAPI/Swagger Specification):**
    *   **Purpose:** To serve as the single source of truth for all backend API functionalities, acting as a contract for frontend developers, mobile developers, and any third-party integrators.
    *   **Contents (for each endpoint):**
        *   Clear description of the endpoint's purpose and functionality.
        *   HTTP method (GET, POST, PUT, DELETE, etc.) and full request path.
        *   Detailed specification of all path parameters, query parameters, and request headers (including name, data type, required/optional, description).
        *   Schema definitions for request bodies (JSON), including all fields, data types, validations (e.g., min/max length, format, enums), and examples.
        *   Schema definitions for all possible response bodies (JSON) for different HTTP status codes (e.g., 200, 201, 400, 401, 403, 404, 500), including all fields, data types, and examples.
        *   List of all possible error codes specific to the endpoint, with explanations.
        *   Authentication and authorization requirements (e.g., required JWT scopes or roles).
        *   Example request and response payloads.
    *   **Tooling:** Using tools like Swagger Editor or Swashbuckle (for ASP.NET Core) can help generate and maintain this specification, often providing interactive API documentation.

Adopting these detailed documentation practices will significantly enhance team collaboration, reduce integration issues, simplify debugging, and ensure the long-term maintainability and evolvability of the STAR Community App.
