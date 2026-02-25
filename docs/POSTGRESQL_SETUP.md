# PostgreSQL Setup Guide for StudFolio

This document explains how PostgreSQL is configured for the StudFolio backend project.

It includes:
- What we did
- Why we did it
- Step-by-step setup
- Security reasoning
- Troubleshooting guide
- Future improvements

This ensures every teammate can configure their local database correctly and understand the architecture decisions.

---

# 1. Purpose

To create a secure, isolated, and production-ready database setup for StudFolio.

Instead of using the default `postgres` superuser directly inside our application, we created:

- A dedicated database (`studfolio_db`)
- A dedicated user (`studfolio_user`)

## Why?

Using separate database and user ensures:

- Project isolation
- Better security
- Controlled permissions
- Easier maintenance
- Production-ready architecture
- Avoid using superuser in application

This is a standard backend best practice.

---

# 2. Versions Used

For consistency across teammates, the following versions are recommended:

- PostgreSQL: 15+ (or latest stable)
- Spring Boot: 3.5.11
- Java: 17
- pgAdmin: Latest version

Using similar versions avoids unexpected compatibility issues.

---

# 3. Detailed Step-by-Step Setup (Using pgAdmin)

---

## Prerequisites

Before starting, ensure:

- PostgreSQL is installed
- pgAdmin is installed
- PostgreSQL service is running
- You know your `postgres` superuser password

### Why?

The `postgres` account is the administrative account used to create databases and users.

---

## Step 1: Install PostgreSQL

Download from:

https://www.postgresql.org/download/

During installation:

- Set a password for `postgres`
- Remember it

### Why?

This password is required to:
- Create databases
- Create users
- Manage privileges

---

## Step 2: Open pgAdmin

1. Open pgAdmin
2. Expand:

Servers → PostgreSQL → Databases

### Why?

pgAdmin provides a graphical interface to manage:
- Databases
- Roles
- Permissions
- Schemas

---

## Step 3: Create Database

1. Right-click **Databases**
2. Click **Create → Database**
3. Enter:

- Database Name: `studfolio_db`
- Owner: `postgres`

Click **Save**

### Why?

Each project must have its own database to:

- Prevent mixing data between projects
- Maintain clean separation
- Allow independent deployment

---

## Step 4: Create Dedicated User

1. Expand **Login/Group Roles**
2. Right-click → **Create → Login/Group Role**

### General Tab
- Name: `studfolio_user`

### Definition Tab
- Password: `studfolio123`
- Confirm password

### Privileges Tab
Enable:
- Can login → YES
- Can create database → YES

Click **Save**

### Why?

Applications should never use superuser accounts.

A dedicated user:
- Limits access scope
- Improves security
- Makes revoking access easy
- Follows production standards

---

## Step 5: Assign Database Privileges

1. Right-click `studfolio_db`
2. Click **Properties**
3. Open **Security tab**
4. Click **Add**
5. Select `studfolio_user`
6. Enable ALL privileges
7. Click **Save**

### Why?

Without privileges, the user cannot:
- Create tables
- Insert data
- Update records
- Delete records

Granting privileges ensures the application can operate normally.

---

## Step 6: Grant Schema Permissions

1. Expand:

Databases → studfolio_db → Schemas → public

2. Right-click → **Properties**
3. Open **Security tab**
4. Add `studfolio_user`
5. Enable ALL permissions
6. Click **Save**

### Why?

PostgreSQL stores tables inside schemas.

If schema permissions are not granted:
- Hibernate cannot create tables
- Application will fail during startup

This ensures `ddl-auto: update` works correctly.

---

# 4. Spring Boot Configuration

Update `application.yml`:

```yaml
spring:
  application:
    name: studfolio

  datasource:
    url: jdbc:postgresql://localhost:5432/studfolio_db
    username: studfolio_user
    password: studfolio123
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

## Why?

- `url` → Connects to correct database
- `username` → Uses dedicated project user
- `password` → Authentication
- `ddl-auto: update` → Auto creates tables
- `show-sql` → Helps debugging

---

# 5. Environment Variable Configuration (Recommended for Production)

Hardcoding passwords is acceptable for local development but not for production.

Instead, use:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Then set environment variables in your system.

## Why?

- Prevents exposing credentials in GitHub
- Improves security
- Production-ready setup

---

# 6. How To Verify It Works

1. Run Spring Boot
2. Ensure no authentication or permission errors appear
3. Open pgAdmin
4. Check:

Databases → studfolio_db → Schemas → public → Tables

Tables should be auto-created.

If tables appear, configuration is correct.

---

# 7. Common Errors & Troubleshooting

### Error: Password authentication failed
Cause: Wrong username/password  
Fix: Verify credentials in pgAdmin and application.yml

---

### Error: Permission denied for schema public
Cause: Schema privileges not granted  
Fix: Enable ALL permissions in public schema

---

### Error: Database does not exist
Cause: Wrong database name in application.yml  
Fix: Create `studfolio_db`

---

### Error: Hibernate cannot determine dialect
Cause: Database connection failed  
Fix: Verify PostgreSQL is running

---

# 8. Database Naming Conventions

To maintain consistency:

- Database → snake_case
- Tables → snake_case
- Columns → snake_case
- Avoid camelCase in database

Example:

```
studfolio_db
user_profile
created_at
```

## Why?

- Improves readability
- Standard SQL practice
- Prevents confusion

---

# 9. Security Best Practices

- Never use `postgres` superuser in applications
- Never commit production passwords to GitHub
- Use different passwords for production
- Rotate credentials periodically
- Use environment variables in deployment

---

# 10. Future Improvements

In future, we will:

- Add Flyway or Liquibase for migration versioning
- Move all credentials to environment variables
- Restrict database privileges in production
- Add Docker-based database setup

---

# 11. Instructions for Teammates

Every teammate must:

1. Install PostgreSQL
2. Create `studfolio_db`
3. Create `studfolio_user`
4. Set password to `studfolio123`
5. Assign database privileges
6. Grant schema permissions
7. Update `application.yml`

Backend will then run at:

http://localhost:8080

---

PostgreSQL setup completed successfully.

This configuration is secure, organized, and production-ready.
