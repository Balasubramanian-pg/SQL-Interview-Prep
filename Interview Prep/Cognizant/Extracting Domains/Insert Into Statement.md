Here's the complete `INSERT INTO` query to populate the `employees` table for testing email domain and username extraction:

```sql
-- Create employees table if it doesn't exist
CREATE TABLE IF NOT EXISTS employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Clear existing data (if any)
TRUNCATE TABLE employees;

-- Insert sample data with various email formats:
-- 1. Standard corporate emails
-- 2. Different domain types (.com, .org, .net, etc.)
-- 3. Subdomains
-- 4. Edge cases (multiple @ symbols, invalid formats)
-- 5. International domains
INSERT INTO employees (id, name, email) VALUES
-- Standard corporate emails
(1, 'John Smith', 'john.smith@company.com'),
(2, 'Sarah Johnson', 'sarah.johnson@company.com'),
(3, 'Michael Brown', 'michael.brown@company.org'),

-- Different domain types
(4, 'Emily Davis', 'emily.davis@business.net'),
(5, 'Robert Wilson', 'robert.wilson@organization.edu'),
(6, 'Jennifer Lee', 'jennifer.lee@startup.io'),

-- Subdomains
(7, 'David Miller', 'david.miller@us.branch.company.com'),
(8, 'Jessica Taylor', 'jessica.taylor@uk.department.org'),

-- Edge cases
(9, 'Daniel Anderson', 'daniel.anderson@company@backup.com'),  -- Multiple @ symbols
(10, 'Lisa Martinez', 'lisa.martinez@company'),                -- Missing TLD
(11, 'Kevin Robinson', 'kevin.robinson@'),                     -- Empty domain
(12, 'Amanda White', 'amanda.white'),                          -- No email
(13, 'James Harris', 'james.harris@sub.domain.co.uk'),         -- Multi-part domain

-- International domains
(14, 'Maria Garcia', 'maria.garcia@empresa.es'),               -- Spanish
(15, 'Thomas Martin', 'thomas.martin@entreprise.fr'),          -- French
(16, 'Sophia Clark', 'sophia.clark@firma.de');                 -- German
```

To test the email domain extraction:

```sql
-- Standard domain extraction (works for most cases)
SELECT
    id,
    name,
    email,
    SUBSTRING(email FROM POSITION('@' IN email) + 1) AS domain,
    CASE 
        WHEN email LIKE '%@%' THEN SUBSTRING(email FROM 1 FOR POSITION('@' IN email) - 1)
        ELSE NULL
    END AS username
FROM
    employees
ORDER BY
    id;
```

For PostgreSQL's SPLIT_PART (more robust handling):

```sql
SELECT
    id,
    name,
    email,
    CASE 
        WHEN email LIKE '%@%' THEN SPLIT_PART(email, '@', 2)
        ELSE NULL
    END AS domain,
    CASE
        WHEN email LIKE '%@%' THEN SPLIT_PART(email, '@', 1)
        ELSE NULL
    END AS username
FROM
    employees
ORDER BY
    id;
```

For SQL Server (using RIGHT/LEFT):

```sql
SELECT
    id,
    name,
    email,
    CASE
        WHEN CHARINDEX('@', email) > 0 THEN RIGHT(email, LEN(email) - CHARINDEX('@', email))
        ELSE NULL
    END AS domain,
    CASE
        WHEN CHARINDEX('@', email) > 0 THEN LEFT(email, CHARINDEX('@', email) - 1)
        ELSE NULL
    END AS username
FROM
    employees
ORDER BY
    id;
```

Key features of this test data:
1. **Standard email formats** (most common cases)
2. **Various domain types** (.com, .org, .net, .edu, .io)
3. **Subdomains** (multi-part domains)
4. **Edge cases**:
   - Multiple @ symbols
   - Missing top-level domain
   - Empty domain
   - Missing email entirely
5. **International domains** (non-English TLDs)

The solutions should:
- Correctly handle standard email formats
- Gracefully manage edge cases (return NULL when invalid)
- Properly extract both usernames and domains
- Work across different database systems

The CASE statements ensure NULL is returned for invalid email formats rather than errors. For production use, you might want to add additional validation or cleaning steps.
