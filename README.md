# SQL Best Practices Repository

Welcome to the SQL Best Practices repository! This repository contains comprehensive documentation on SQL best practices, with a focus on the WJO Enterprise system.

## Contents

### 📚 Documentation

1. **[11 Best Practices for Writing SQL Queries](./11_Best_Practices_SQL_Readability.md)**
   - Comprehensive guide to writing readable and maintainable SQL queries
   - Covers formatting, naming conventions, and code organization
   - Includes practical before/after examples

2. **[WJO Enterprise Architecture Overview](./WJO_Enterprise_Architecture.md)**
   - Complete architectural documentation for the WJO Enterprise system
   - Database schema and table structures
   - Security considerations and best practices
   - Integration points and API structure
   - Performance optimization guidelines

## About WJO Enterprise

The **WJO (Workforce & Jobs Organization) Enterprise** is a comprehensive user and identity management system designed to:
- Manage user profiles and personal information
- Track identity documents (Civil IDs, Passports)
- Monitor document expiration dates
- Maintain secure access to user data

## Quick Start

### For Developers

1. Review the [SQL Best Practices](./11_Best_Practices_SQL_Readability.md) document before writing any queries
2. Familiarize yourself with the [WJO Enterprise Architecture](./WJO_Enterprise_Architecture.md)
3. Follow the naming conventions and query patterns outlined in the documentation

### Key Principles

- **Consistency**: Use consistent formatting and naming conventions
- **Readability**: Write queries that are easy to understand and maintain
- **Security**: Follow security best practices for data access and protection
- **Performance**: Optimize queries with proper indexing and query patterns

## Examples

### Well-Formatted Query Example
```sql
-- Selecting active users with expiring documents
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
    AND (
        u.CivilIDExpiryDate <= DATEADD(DAY, 30, GETDATE())
        OR u.PassPortExpiryDate <= DATEADD(DAY, 30, GETDATE())
    )
ORDER BY 
    u.LastName,
    u.FirstName;
```

## Contributing

When contributing to this repository:
1. Follow the established SQL best practices
2. Update documentation when making architectural changes
3. Include examples for new patterns or practices
4. Ensure all queries are properly formatted and tested

## Resources

- **SQL Server Documentation**: [Microsoft SQL Server Docs](https://docs.microsoft.com/en-us/sql/)
- **SQL Formatting Tools**: Use IDE features or online formatters
- **Best Practices**: Continuously learn and adapt best practices

## Author

**MoAbdulFattah**

## License

This repository is maintained for educational and reference purposes.

---

*Last Updated: 2026-01-15*
