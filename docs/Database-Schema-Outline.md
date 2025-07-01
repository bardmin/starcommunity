# Detailed Database Schema Document - STAR Community App

## 0. Version History

| Version | Date       | Author(s)   | Summary of Changes                                  |
|---------|------------|-------------|-----------------------------------------------------|
| 1.0     | YYYY-MM-DD | Jules (AI)  | Initial draft outline.                              |
|         |            |             |                                                     |

## 1. Introduction

### 1.1 Purpose of this Document
    - Describe the document's goal: to provide a comprehensive definition of the database structure for the STAR Community App backend.
    - Target audience (developers, database administrators, architects).

### 1.2 Scope
    - Which parts of the system does this schema cover (e.g., all microservices for MVP).
    - Relationship to the overall Backend Architecture document.

### 1.3 Database Overview
    - Database technology used (PostgreSQL, specific version).
    - Brief description of the overall data model strategy (e.g., single instance with schema-per-service for MVP).
    - High-level diagram illustrating how different service schemas relate (if applicable).

### 1.4 Conventions Used in this Document
    - Naming conventions for tables, columns, indexes, constraints.
    - Data type notation.
    - Diagram notation (e.g., Crow's Foot for ERDs).
    - Primary Key (PK), Foreign Key (FK), Unique Key (UK) indicators.

## 2. General Database Conventions & Standards

### 2.1 Naming Conventions
    - Tables: e.g., `PascalCase`, `snake_case` (specify and be consistent, e.g., `Users`, `ServiceBookings`).
    - Columns: e.g., `camelCase`, `PascalCase` (specify, e.g., `userId`, `BookingTime`).
    - Primary Keys: e.g., `ID` or `TableNameID` (e.g., `UserID`).
    - Foreign Keys: e.g., `RelatedTableNameID` (e.g., `UserID` in `Bookings` table referencing `Users.UserID`).
    - Indexes: e.g., `IX_TableName_ColumnName`.
    - Constraints: e.g., `CK_TableName_ColumnName_Rule`, `UQ_TableName_ColumnName`.

### 2.2 Data Types
    - Standard data types to be used (e.g., `UUID` for PKs, `VARCHAR(N)` for strings, `TEXT` for long strings, `TIMESTAMP WITH TIME ZONE` for dates, `DECIMAL(P,S)` for currency, `BOOLEAN`, `JSONB` for flexible data).
    - Justification for common data type choices.

### 2.3 Referential Integrity
    - Policy on foreign key constraints (e.g., `ON DELETE RESTRICT`, `ON DELETE SET NULL`, `ON DELETE CASCADE` - specify usage).
    - Handling of orphaned records.

### 2.4 Nullability
    - General approach to `NULL` values (e.g., avoid where possible, use `NOT NULL` extensively).
    - Default values for columns.

### 2.5 Character Sets & Collations
    - Default character set (e.g., UTF-8).
    - Default collation.

## 3. Detailed Schema per Microservice

*(Repeat this section for each microservice: User Management, Service & Booking Management, Token & Wallet, Payment Integration, Community Management, Notification Service)*

### 3.X [Microservice Name] Service Schema

#### 3.X.1 Introduction & Overview
    - Brief description of the domain and data managed by this service.
    - Schema name if using schema-per-service (e.g., `user_service_schema`).

#### 3.X.2 Entity-Relationship Diagram (ERD)
    - Detailed ERD for this microservice's schema (e.g., using Mermaid syntax or embedded image).
    - Clearly show all tables, columns (at least PKs/FKs), and relationships with cardinalities.

#### 3.X.3 Table Definitions
    *(Repeat this subsection for each table within the microservice's schema)*

##### 3.X.3.Y [TableName]

*   **Description:** Purpose of the table.
*   **Columns:**
    | Column Name     | Data Type         | Constraints                                  | Nullable | Default Value | Description / Notes                               |
    |-----------------|-------------------|----------------------------------------------|----------|---------------|---------------------------------------------------|
    | `UserID`        | `UUID`            | `PRIMARY KEY`                                | No       | `gen_random_uuid()` | Unique identifier for the user.                   |
    | `FullName`      | `VARCHAR(255)`    | `NOT NULL`                                   | No       |               | User's full name.                                 |
    | `Email`         | `VARCHAR(255)`    | `NOT NULL`, `UNIQUE`                         | No       |               | User's email address.                             |
    | `PasswordHash`  | `VARCHAR(512)`    | `NOT NULL`                                   | No       |               | Hashed password.                                  |
    | `Role`          | `VARCHAR(50)`     | `NOT NULL`, `CHECK (Role IN ('Resident', ...))` | No       |               | User's role in the system.                        |
    | `CreatedAt`     | `TIMESTAMPTZ`     | `NOT NULL`                                   | No       | `NOW()`       | Timestamp of record creation.                     |
    | `UpdatedBy`     | `UUID`            | `FOREIGN KEY (Users.UserID)`                 | Yes      |               | User who last updated the record.                 |
    | ...             | ...               | ...                                          | ...      | ...           | ...                                               |
*   **Indexes:**
    | Index Name              | Columns Involved        | Type (e.g., B-tree, GIN) | Unique | Description                                       |
    |-------------------------|-------------------------|--------------------------|--------|---------------------------------------------------|
    | `IX_Users_Email`        | `Email`                 | B-tree                   | Yes    | For fast lookup by email.                         |
    | `IX_Users_Role`         | `Role`                  | B-tree                   | No     | For filtering users by role.                      |
    | ...                     | ...                     | ...                      | ...    | ...                                               |
*   **Constraints (other than column-level):**
    | Constraint Name         | Type (e.g., FOREIGN KEY, CHECK) | Details                                              |
    |-------------------------|---------------------------------|------------------------------------------------------|
    | `FK_Addresses_UserID`   | `FOREIGN KEY`                   | `Addresses(UserID)` REFERENCES `Users(UserID)` ON DELETE CASCADE |
    | ...                     | ...                             | ...                                                  |
*   **Notes / Business Rules:** Any specific rules related to this table not covered elsewhere.

## 4. Data Dictionary (Optional - if not fully covered in table definitions)

    - Alphabetical listing of all significant data elements (columns) across all tables.
    - For each element: Name, Table(s) it appears in, Data Type, Description, Allowed Values/Format.
    - Useful for a quick lookup of a specific field's meaning.

## 5. Views (If any)

    - Definition and purpose of any database views created for reporting or simplifying complex queries.
    - SQL code for creating the view.

## 6. Stored Procedures & Functions (If any)

    - Definition, purpose, parameters, and return values for any stored procedures or functions.
    - SQL code or PL/pgSQL code.
    - (Generally, business logic is preferred in the microservice code, but some utility functions might reside in the DB).

## 7. Appendix

### 7.1 Glossary of Terms
    - Definitions of any domain-specific or technical terms used in the document.

### 7.2 References
    - Links to related documents (e.g., Backend Architecture Plan, requirements documents).
```
