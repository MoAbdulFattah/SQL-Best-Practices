# 11 Best Practices for Writing SQL Queries in SQL Server with Better Readability  
**Author**: MoAbdulFattah  
**Date**: 2025-11-11  

## 1. Consistent Formatting and Indentation  
Consistency in formatting and indentation makes SQL queries easier to read and maintain. For example:

```sql
SELECT  
    FirstName,  
    LastName,  
    CivilIDExpiryDate  
FROM Users  
WHERE Active = 1;
```

## 2. Capitalize SQL Keywords  
Capitalizing keywords helps to distinguish SQL commands from other identifiers:

```sql
SELECT FirstName, LastName FROM Users WHERE Active = 1;
```
becomes
```sql
SELECT FirstName, LastName FROM Users WHERE Active = 1;
```

## 3. Break Complex Queries into Multiple Lines  
This practice enhances readability and makes it easier to debug:

```sql
SELECT FirstName, LastName
FROM Users
WHERE Active = 1 AND Age > 30;
```

## 4. Use Meaningful Aliases  
Meaningful aliases clarify the context of the fields:

```sql
SELECT u.FirstName AS UserFirstName, u.LastName AS UserLastName 
FROM Users u;
```

## 5. Add Comments and Documentation  
Commenting on complex queries provides context:

```sql
-- Fetch active users older than 30
SELECT FirstName, LastName
FROM Users
WHERE Active = 1 AND Age > 30;
```

## 6. Organize Clauses Logically  
Structure SQL statements logically for better flow:

```sql
SELECT FirstName, LastName
FROM Users
WHERE Active = 1
ORDER BY LastName;
```

## 7. Use Common Table Expressions (CTEs)  
CTEs simplify complex queries:

```sql
WITH ActiveUsers AS (SELECT * FROM Users WHERE Active = 1)
SELECT FirstName, LastName FROM ActiveUsers;
```

## 8. Format JOINs Clearly  
Clearly defining join conditions can improve understanding:

```sql
SELECT u.FirstName, o.OrderDate
FROM Users u
JOIN Orders o ON u.UserID = o.UserID;
```

## 9. Use White Space Effectively  
Using white spaces effectively enhances readability:

```sql
SELECT
    FirstName,
    LastName
FROM Users;
```

## 10. Follow a Consistent Style Guide  
Having a style guide ensures uniformity across the codebase.

## 11. Use Clear Column Naming Conventions (PascalCase)  
Consistent naming conventions enhance clarity:

```sql
SELECT UserID, FirstName, LastName
FROM Users;
```

## Before/After Example  
**Before:**  
```sql
SELECT FirstName, LastName, CivilIDExpiryDate, PassportExpiryDate FROM WJOUsers WHERE Active = 1;
```

**After:**  
```sql
-- Selecting user details with proper formatting
SELECT  
    FirstName AS UserFirstName,  
    LastName AS UserLastName,  
    CivilIDExpiryDate,  
    PassportExpiryDate  
FROM WJOUsers  
WHERE Active = 1;
```

## Appendix  
- Regularly refactor SQL queries for readability.  
- Use SQL formatting tools or IDE features to automate formatting.  
- Continue learning and adapting best practices to improve skills.