# WJO Enterprise Architecture Overview

**Author**: MoAbdulFattah  
**Date**: 2026-01-15  
**Version**: 1.0

## Executive Summary

The WJO (Workforce & Jobs Organization) Enterprise is a comprehensive user and identity management system designed to track employee and citizen information, including identity document management and expiration tracking. This document provides an architectural overview of the WJO system, including its database schema, data flow, security considerations, and integration patterns.

## System Overview

### Purpose
The WJO Enterprise system is designed to:
- Manage user profiles and personal information
- Track identity documents (Civil IDs, Passports)
- Monitor document expiration dates
- Maintain user activity status
- Provide secure access to user data

### Key Features
- User profile management
- Identity document tracking
- Expiration date monitoring and alerts
- Active/inactive user status management
- Secure data access controls

## Architecture Components

### 1. Database Layer

#### Core Tables

##### WJOUsers Table
The primary table for storing user information:

```sql
CREATE TABLE WJOUsers (
    UserID INT PRIMARY KEY IDENTITY(1,1),
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    Email NVARCHAR(255) UNIQUE,
    PhoneNumber NVARCHAR(20),
    CivilIDNumber NVARCHAR(50),
    CivilIDExpiryDate DATE,
    PassportNumber NVARCHAR(50),
    PassPortExpiryDate DATE,
    Active BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE(),
    ModifiedDate DATETIME DEFAULT GETDATE(),
    CreatedBy INT,
    ModifiedBy INT
);
```

**Field Descriptions:**
- `UserID`: Unique identifier for each user
- `FirstName`: User's first name
- `LastName`: User's family name
- `Email`: User's email address (unique constraint)
- `PhoneNumber`: Contact phone number
- `CivilIDNumber`: Civil identification number
- `CivilIDExpiryDate`: Expiration date of civil ID
- `PassportNumber`: Passport identification number
- `PassPortExpiryDate`: Expiration date of passport
- `Active`: Status flag (1 = Active, 0 = Inactive)
- `CreatedDate`: Timestamp when record was created
- `ModifiedDate`: Timestamp when record was last modified
- `CreatedBy`: UserID of the creator
- `ModifiedBy`: UserID of the last modifier

##### WJODocumentTypes Table
Reference table for document types:

```sql
CREATE TABLE WJODocumentTypes (
    DocumentTypeID INT PRIMARY KEY IDENTITY(1,1),
    DocumentTypeName NVARCHAR(100) NOT NULL,
    Description NVARCHAR(500),
    IsActive BIT DEFAULT 1
);
```

##### WJOAuditLog Table
Tracks all changes to user records:

```sql
CREATE TABLE WJOAuditLog (
    AuditID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL,
    ActionType NVARCHAR(50) NOT NULL,
    ActionDate DATETIME DEFAULT GETDATE(),
    ActionBy INT NOT NULL,
    OldValue NVARCHAR(MAX),
    NewValue NVARCHAR(MAX),
    TableName NVARCHAR(100),
    FOREIGN KEY (UserID) REFERENCES WJOUsers(UserID),
    FOREIGN KEY (ActionBy) REFERENCES WJOUsers(UserID)
);
```

### 2. Application Layer

#### Business Logic Components

1. **User Management Service**
   - CRUD operations for user profiles
   - Validation of user data
   - Business rule enforcement

2. **Document Management Service**
   - Document expiration tracking
   - Alert generation for expiring documents
   - Document renewal workflows

3. **Security Service**
   - Authentication and authorization
   - Role-based access control (RBAC)
   - Audit logging

### 3. Data Access Layer

#### Stored Procedures

##### Get Active Users
```sql
CREATE PROCEDURE sp_GetActiveUsers
AS
BEGIN
    SELECT 
        UserID,
        FirstName,
        LastName,
        Email,
        CivilIDExpiryDate,
        PassPortExpiryDate
    FROM WJOUsers
    WHERE Active = 1
    ORDER BY LastName, FirstName;
END
```

##### Get Users with Expiring Documents
```sql
CREATE PROCEDURE sp_GetUsersWithExpiringDocuments
    @DaysThreshold INT = 30
AS
BEGIN
    SELECT 
        UserID,
        FirstName AS UserFirstName,
        LastName AS UserLastName,
        Email,
        CivilIDExpiryDate,
        PassPortExpiryDate,
        CASE 
            WHEN CivilIDExpiryDate <= DATEADD(DAY, @DaysThreshold, GETDATE()) 
            THEN 'Civil ID Expiring Soon'
            WHEN PassPortExpiryDate <= DATEADD(DAY, @DaysThreshold, GETDATE()) 
            THEN 'Passport Expiring Soon'
            ELSE 'No Immediate Expiration'
        END AS ExpirationStatus
    FROM WJOUsers
    WHERE 
        Active = 1 
        AND (
            CivilIDExpiryDate <= DATEADD(DAY, @DaysThreshold, GETDATE())
            OR PassPortExpiryDate <= DATEADD(DAY, @DaysThreshold, GETDATE())
        )
    ORDER BY 
        CASE 
            WHEN CivilIDExpiryDate < PassPortExpiryDate THEN CivilIDExpiryDate
            ELSE PassPortExpiryDate
        END;
END
```

## Data Flow

### User Creation Flow
1. User submits registration form
2. Application validates input data
3. Data Access Layer inserts record into WJOUsers table
4. Audit log entry created in WJOAuditLog
5. Confirmation sent to user

### Document Expiration Monitoring Flow
1. Scheduled job runs daily
2. Query executed to find documents expiring within threshold
3. Notifications generated for relevant users
4. Alert records created in notification system
5. Email/SMS sent to users and administrators

## Security Considerations

### Data Protection
- **Encryption at Rest**: Sensitive fields (CivilIDNumber, PassportNumber) should be encrypted
- **Encryption in Transit**: All API communications use TLS 1.2 or higher
- **Data Masking**: Personal identifiable information (PII) masked in logs

### Access Control
- **Role-Based Access Control (RBAC)**:
  - Admin: Full access to all records
  - Manager: Read/write access to team records
  - User: Read-only access to own record
  - Auditor: Read-only access to audit logs

### Compliance
- **GDPR Compliance**: Right to access, right to be forgotten
- **Data Retention**: Inactive records archived after 7 years
- **Audit Trail**: All data modifications logged

## Integration Points

### API Endpoints

#### REST API Structure
```
GET    /api/v1/users              - List all active users
GET    /api/v1/users/{id}         - Get specific user
POST   /api/v1/users              - Create new user
PUT    /api/v1/users/{id}         - Update user
DELETE /api/v1/users/{id}         - Deactivate user
GET    /api/v1/users/expiring     - Get users with expiring documents
```

### External Systems
- **Identity Provider**: Integration with OAuth 2.0 / OpenID Connect
- **Notification Service**: Email and SMS gateway
- **Document Verification Service**: Third-party document validation

## Performance Considerations

### Indexing Strategy
```sql
-- Primary key index on UserID (created automatically)
CREATE INDEX IX_WJOUsers_Active ON WJOUsers(Active);
CREATE INDEX IX_WJOUsers_Email ON WJOUsers(Email);
CREATE INDEX IX_WJOUsers_CivilIDExpiryDate ON WJOUsers(CivilIDExpiryDate) WHERE Active = 1;
CREATE INDEX IX_WJOUsers_PassPortExpiryDate ON WJOUsers(PassPortExpiryDate) WHERE Active = 1;
CREATE INDEX IX_WJOUsers_LastName_FirstName ON WJOUsers(LastName, FirstName);
```

### Query Optimization
- Use parameterized queries to prevent SQL injection
- Implement pagination for large result sets
- Use appropriate WHERE clause filtering
- Leverage indexed columns in JOIN conditions

## Best Practices

### SQL Query Patterns
Following the 11 Best Practices for SQL Readability (see 11_Best_Practices_SQL_Readability.md):

```sql
-- Good: Properly formatted query
SELECT 
    u.UserID,
    u.FirstName AS UserFirstName,
    u.LastName AS UserLastName,
    u.Email,
    u.CivilIDExpiryDate,
    u.PassPortExpiryDate
FROM WJOUsers u
WHERE 
    u.Active = 1
    AND u.CivilIDExpiryDate > GETDATE()
ORDER BY 
    u.LastName,
    u.FirstName;
```

### Naming Conventions
- **Tables**: PascalCase with WJO prefix (e.g., WJOUsers)
- **Columns**: PascalCase (e.g., FirstName, CivilIDExpiryDate)
- **Stored Procedures**: sp_VerbNoun format (e.g., sp_GetActiveUsers)
- **Indexes**: IX_TableName_ColumnName format

### Code Review Guidelines
- All queries must follow the readability guidelines
- Indexes required for frequently queried columns
- Stored procedures preferred over inline SQL
- All data modifications must be audited

## Deployment Architecture

### Environment Tiers
1. **Development**: For active development and testing
2. **Staging**: Pre-production environment for UAT
3. **Production**: Live environment with high availability

### High Availability
- Database replication (Primary-Secondary setup)
- Load balancing for API layer
- Automatic failover mechanisms
- Regular backup schedule (daily full, hourly differential)

## Monitoring and Maintenance

### Key Metrics
- Query response times
- Database connection pool usage
- Document expiration alert accuracy
- API endpoint performance
- Error rates and exceptions

### Maintenance Tasks
- Weekly index maintenance and statistics updates
- Monthly performance review and optimization
- Quarterly security audits
- Annual disaster recovery testing

## Future Enhancements

### Planned Features
1. **Multi-factor Authentication (MFA)**: Enhanced security for user access
2. **Mobile Application**: Native mobile apps for iOS and Android
3. **Document OCR**: Automatic extraction of document information
4. **Advanced Analytics**: Dashboard with user statistics and trends
5. **Integration Hub**: Middleware for third-party system integration

### Scalability Considerations
- Microservices architecture migration
- Cloud-native deployment (Azure/AWS)
- Event-driven architecture for real-time notifications
- GraphQL API layer for flexible data queries

## Appendix

### Related Documents
- [11 Best Practices for Writing SQL Queries](./11_Best_Practices_SQL_Readability.md)
- WJO API Documentation (separate document)
- WJO Security Policy (separate document)
- WJO Data Retention Policy (separate document)

### Glossary
- **WJO**: Workforce & Jobs Organization
- **Civil ID**: National civil identification document
- **PII**: Personally Identifiable Information
- **RBAC**: Role-Based Access Control
- **UAT**: User Acceptance Testing
- **OCR**: Optical Character Recognition

### Contact Information
For questions or concerns about the WJO Enterprise Architecture:
- **Technical Lead**: [Contact Information]
- **Database Administrator**: [Contact Information]
- **Security Team**: [Contact Information]

---
*Last Updated: 2026-01-15*
