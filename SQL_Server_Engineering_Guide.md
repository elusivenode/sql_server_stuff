
# SQL Server Engineering Guide

## T-SQL Essentials

### Core Concepts

- **Joins**:
  - `INNER JOIN`: Returns only the rows that have matching values in both tables.
  - `LEFT OUTER JOIN`: Returns all rows from the left table, and matched rows from the right. NULLs for unmatched.
  - `RIGHT OUTER JOIN`: Returns all rows from the right table, and matched rows from the left. NULLs for unmatched.
  - `FULL OUTER JOIN`: Combines results of LEFT and RIGHT JOINs.
  - `CROSS JOIN`: Returns the Cartesian product of both tables.
  - `SELF JOIN`: A table is joined to itself using aliases.

- **CTEs (Common Table Expressions)**:
  - Temporary result set defined with `WITH` keyword.
  - Useful for recursion, better readability, breaking down complex queries.
  - Example:
```sql
WITH SalesCTE AS (
    SELECT SalesPersonID,
           SUM(SalesAmount) AS TotalSales
    FROM Sales
    GROUP BY SalesPersonID
)

SELECT *
FROM SalesCTE
WHERE TotalSales > 10000;
```

- **Subqueries**:
  - A query nested inside another query (`SELECT`, `FROM`, or `WHERE`).
  - Correlated subqueries reference the outer query and are executed row-by-row.
  - Inline subqueries useful for calculated columns or filtering.
  - Example:
```sql
SELECT Name,  Salary FROM Employees e WHERE Salary > ( SELECT AVG(Salary) FROM
Employees WHERE DepartmentID = e.DepartmentID );
```

- **APPLY (CROSS APPLY and OUTER APPLY)**:
  - Enables joining each row to a table-valued function or derived table.
  - `CROSS APPLY` behaves like `INNER JOIN`.
  - `OUTER APPLY` behaves like `LEFT JOIN`.
  - Best used when:
    - Using table-valued functions
    - Joining on derived tables that depend on outer rows
  - Example:
```sql
SELECT e.EmployeeID,  e.Name,  t.MostRecentTrainingDate FROM Employees e OUTER
APPLY ( SELECT TOP 1 TrainingDate AS MostRecentTrainingDate FROM Training t
WHERE t.EmployeeID = e.EmployeeID ORDER BY TrainingDate DESC ) t;
```

### Comparison

| Feature            | CTE                    | Subquery                     | APPLY                              |
|--------------------|------------------------|-------------------------------|-------------------------------------|
| Readability        | High                   | Medium                        | Medium                              |
| Reusability        | Moderate (referencable) | Low                           | High (w/ table-valued functions)    |
| Recursion support  | Yes                    | No                            | No                                  |
| Performance        | Similar to derived tables | Inline fast, Correlated slower | Efficient with TVFs, sometimes better |

### Use Cases

- Use **CTEs** when:
  - Breaking a complex query into parts
  - Performing recursive operations (e.g., org charts)

- Use **Subqueries** when:
  - You need a quick filter or inline computation
  - The subquery is scalar or tightly bound to the main query

- Use **APPLY** when:
  - You need to invoke a table-valued function per row
  - Joining each row in the outer query to dynamic subsets

- **MERGE statements**:
  - Allows `INSERT`, `UPDATE`, and `DELETE` in one operation by comparing target and source.
  - Example:
```sql
MERGE INTO TargetTable AS T USING SourceTable AS S ON T.ID = S.ID WHEN MATCHED
THEN UPDATE SET T.Value = S.Value WHEN NOT MATCHED BY TARGET     THEN INSERT
(ID,  Value) VALUES (S.ID,  S.Value) WHEN NOT MATCHED BY SOURCE     THEN DELETE;
```
  - Use cases:
    - Synchronizing tables (ETL)
    - Handling slow-changing dimensions
    - Auditing via `OUTPUT` clause
  - When to use:
    - You want atomic sync logic
    - You need row-level audit logging
    - Logic isn’t overly conditional

  - Performance Comparison:

    | Factor                  | MERGE                              | UPDATE + INSERT                  |
    |-------------------------|------------------------------------|----------------------------------|
    | Simplicity              | One statement                      | Separate logic paths             |
    | Readability             | Complex if many conditions         | Easier for basic logic           |
    | Small Set Performance   | Comparable                         | Slightly faster sometimes        |
    | Large Set Performance   | May degrade with complexity        | Often more efficient             |
    | OUTPUT Support          | Supported                          | Limited to `INSERT`/`UPDATE`     |
    | Error Handling          | All-or-nothing                     | Finer control                    |


## Indexing & Performance Tuning

Core Concepts:
- Clustered vs Non-clustered indexes
- INCLUDE columns, filtered indexes
- Columnstore indexes, execution plans
- DMVs: sys.dm_exec_requests, sys.dm_exec_query_stats
- Index fragmentation and statistics
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
------------------------
--------------------
-----------------
-----------------------------
Indexing Options
Full
Full
No manual partition
switching
Execution Plan Control
Full
Full
Limited query hints
Query Store
Optional
Optional
Always enabled


Backup, Recovery & High Availability
Core Concepts:
- FULL, DIFFERENTIAL, LOG backups
- Recovery models: Simple, Full, Bulk-Logged
- AlwaysOn Availability Groups
- Log shipping and clustering
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
----------------------------
--------------------
-----------------
-----------------------------
Backup Storage
Local/Disk/Cloud
Azure Disks
Azure Blob, Auto-backups
HA Features
Full
Custom setup
Built-in zone redundancy
Point-in-Time Restore
Manual
Manual
Built-in, configurable


Security & Role Management
Core Concepts:
- Users, logins, roles, and permissions
- Row-Level Security (RLS)
- Dynamic Data Masking (DDM)
- SQL Auditing
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
------------------------
--------------------
-----------------
-----------------------------
Authentication
Windows, SQL
AD, SQL
Azure AD, Managed Identity
DDM and RLS
Supported
Supported
Supported
Auditing
Manual or via 3rd
Custom or Azure
Built-in auditing options


Azure SQL Managed Instance & SQL Server 2022 Features
Core Concepts:
- Ledger tables for tamper-evidence
- Synapse Link integration
- Parameter Sensitive Plan Optimization (PSP)
- Contained Availability Groups
- System-versioned temporal tables
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
-------------------------------
--------------------
-----------------
-----------------------------
Ledger Tables
Available
Available
Partial Support
Synapse Link
Not native
Not native
Native integration
Contained AGs
New in SQL 2022
Available
Not available


Deployment & Infrastructure Differences
Core Concepts:
- OS-level access and patching
- Networking: ExpressRoute, VNet, Private Link
- Monitoring: XEvents, DMVs, Azure Monitor
- Scale-up/down options
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
--------------------
--------------------
---------------------
-----------------------------
OS Access
Full
Full
None
Patching
Manual
Semi-auto
Fully managed
Network Control
Full
Full (VNet)
VNet integration only


Mock Interview Questions
Sample Questions:
1. What is the difference between MERGE and UPSERT in SQL Server?
2. How would you handle fragmentation in a large index?
3. Describe how point-in-time recovery works in SQL Server.
4. Explain the difference between a login and a user.
5. What are the advantages of using Azure SQL Managed Instance?
Tip: Be ready to discuss real-world use cases you've handled.

## Backup, Recovery & High Availability

Core Concepts:
- FULL, DIFFERENTIAL, LOG backups
- Recovery models: Simple, Full, Bulk-Logged
- AlwaysOn Availability Groups
- Log shipping and clustering
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
----------------------------
--------------------
-----------------
-----------------------------
Backup Storage
Local/Disk/Cloud
Azure Disks
Azure Blob, Auto-backups
HA Features
Full
Custom setup
Built-in zone redundancy
Point-in-Time Restore
Manual
Manual
Built-in, configurable


Security & Role Management
Core Concepts:
- Users, logins, roles, and permissions
- Row-Level Security (RLS)
- Dynamic Data Masking (DDM)
- SQL Auditing
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
------------------------
--------------------
-----------------
-----------------------------
Authentication
Windows, SQL
AD, SQL
Azure AD, Managed Identity
DDM and RLS
Supported
Supported
Supported
Auditing
Manual or via 3rd
Custom or Azure
Built-in auditing options


Azure SQL Managed Instance & SQL Server 2022 Features
Core Concepts:
- Ledger tables for tamper-evidence
- Synapse Link integration
- Parameter Sensitive Plan Optimization (PSP)
- Contained Availability Groups
- System-versioned temporal tables
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
-------------------------------
--------------------
-----------------
-----------------------------
Ledger Tables
Available
Available
Partial Support
Synapse Link
Not native
Not native
Native integration
Contained AGs
New in SQL 2022
Available
Not available


Deployment & Infrastructure Differences
Core Concepts:
- OS-level access and patching
- Networking: ExpressRoute, VNet, Private Link
- Monitoring: XEvents, DMVs, Azure Monitor
- Scale-up/down options
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
--------------------
--------------------
---------------------
-----------------------------
OS Access
Full
Full
None
Patching
Manual
Semi-auto
Fully managed
Network Control
Full
Full (VNet)
VNet integration only


Mock Interview Questions
Sample Questions:
1. What is the difference between MERGE and UPSERT in SQL Server?
2. How would you handle fragmentation in a large index?
3. Describe how point-in-time recovery works in SQL Server.
4. Explain the difference between a login and a user.
5. What are the advantages of using Azure SQL Managed Instance?
Tip: Be ready to discuss real-world use cases you've handled.

## Security & Role Management

Core Concepts:
- Users, logins, roles, and permissions
- Row-Level Security (RLS)
- Dynamic Data Masking (DDM)
- SQL Auditing
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
------------------------
--------------------
-----------------
-----------------------------
Authentication
Windows, SQL
AD, SQL
Azure AD, Managed Identity
DDM and RLS
Supported
Supported
Supported
Auditing
Manual or via 3rd
Custom or Azure
Built-in auditing options


Azure SQL Managed Instance & SQL Server 2022 Features
Core Concepts:
- Ledger tables for tamper-evidence
- Synapse Link integration
- Parameter Sensitive Plan Optimization (PSP)
- Contained Availability Groups
- System-versioned temporal tables
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
-------------------------------
--------------------
-----------------
-----------------------------
Ledger Tables
Available
Available
Partial Support
Synapse Link
Not native
Not native
Native integration
Contained AGs
New in SQL 2022
Available
Not available


Deployment & Infrastructure Differences
Core Concepts:
- OS-level access and patching
- Networking: ExpressRoute, VNet, Private Link
- Monitoring: XEvents, DMVs, Azure Monitor
- Scale-up/down options
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
--------------------
--------------------
---------------------
-----------------------------
OS Access
Full
Full
None
Patching
Manual
Semi-auto
Fully managed
Network Control
Full
Full (VNet)
VNet integration only


Mock Interview Questions
Sample Questions:
1. What is the difference between MERGE and UPSERT in SQL Server?
2. How would you handle fragmentation in a large index?
3. Describe how point-in-time recovery works in SQL Server.
4. Explain the difference between a login and a user.
5. What are the advantages of using Azure SQL Managed Instance?
Tip: Be ready to discuss real-world use cases you've handled.

## Azure SQL Managed Instance & SQL Server 2022 Features

Core Concepts:
- Ledger tables for tamper-evidence
- Synapse Link integration
- Parameter Sensitive Plan Optimization (PSP)
- Contained Availability Groups
- System-versioned temporal tables
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
-------------------------------
--------------------
-----------------
-----------------------------
Ledger Tables
Available
Available
Partial Support
Synapse Link
Not native
Not native
Native integration
Contained AGs
New in SQL 2022
Available
Not available


Deployment & Infrastructure Differences
Core Concepts:
- OS-level access and patching
- Networking: ExpressRoute, VNet, Private Link
- Monitoring: XEvents, DMVs, Azure Monitor
- Scale-up/down options
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
--------------------
--------------------
---------------------
-----------------------------
OS Access
Full
Full
None
Patching
Manual
Semi-auto
Fully managed
Network Control
Full
Full (VNet)
VNet integration only


Mock Interview Questions
Sample Questions:
1. What is the difference between MERGE and UPSERT in SQL Server?
2. How would you handle fragmentation in a large index?
3. Describe how point-in-time recovery works in SQL Server.
4. Explain the difference between a login and a user.
5. What are the advantages of using Azure SQL Managed Instance?
Tip: Be ready to discuss real-world use cases you've handled.

## Deployment & Infrastructure Differences

Core Concepts:
- OS-level access and patching
- Networking: ExpressRoute, VNet, Private Link
- Monitoring: XEvents, DMVs, Azure Monitor
- Scale-up/down options
Environment Differences:
Feature
On-Prem SQL Server
Azure IaaS (VM)
Azure SQL Managed
Instance
--------------------
--------------------
---------------------
-----------------------------
OS Access
Full
Full
None
Patching
Manual
Semi-auto
Fully managed
Network Control
Full
Full (VNet)
VNet integration only


Mock Interview Questions
Sample Questions:
1. What is the difference between MERGE and UPSERT in SQL Server?
2. How would you handle fragmentation in a large index?
3. Describe how point-in-time recovery works in SQL Server.
4. Explain the difference between a login and a user.
5. What are the advantages of using Azure SQL Managed Instance?
Tip: Be ready to discuss real-world use cases you've handled.

## Mock Interview Questions

Sample Questions:
1. What is the difference between MERGE and UPSERT in SQL Server?
2. How would you handle fragmentation in a large index?
3. Describe how point-in-time recovery works in SQL Server.
4. Explain the difference between a login and a user.
5. What are the advantages of using Azure SQL Managed Instance?
Tip: Be ready to discuss real-world use cases you've handled.