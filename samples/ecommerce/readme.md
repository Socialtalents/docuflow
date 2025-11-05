# E-Commerce Database Seed Example

This example demonstrates how to use DocuFlow for MongoDB database seeding, schema versioning, and index management in an e-commerce application. Perfect for understanding how to version control your MongoDB data and indexes for development, testing, and CI/CD pipelines.

## Overview

This sample shows:
- Exporting seed data for development environments
- Managing MongoDB indexes with version control
- Setting up initial data for users, categories, and warehouses
- Text search and geospatial index configuration
- import all of those changes at once to new database instance

## Building File based database state

### 1. Seed Admin User

Export the initial admin user for your application:

```bash
./docuflow export mongodb://127.0.0.1:27017/ecommerce -c users
# Output:
# 1761942040045113700.import.users-0.json
```

### 2. Export User Collection Indexes

Export indexes including the login field index for authentication:

```bash
./docuflow export mongodb://127.0.0.1:27017/ecommerce -c users --indexes
# Output:
# 1761942148908424400.ensure-indexes.users.json
```

### 3. Export Initial Categories

Export the initial product categories for your e-commerce catalog:

```bash
./docuflow export mongodb://127.0.0.1:27017/ecommerce -c categories
# Output:
# 1761942148908744400.import.categories-0.json
```

### 4. Export Warehouse Locations

Export warehouse data including location information:

```bash
./docuflow export mongodb://127.0.0.1:27017/ecommerce -c warehouses
# Output:
# 1761942148908744401.import.warehouses-0.json
```

## Export Advanced Indexes

### 5. Text Search Index for Categories

Export indexes for product cateogries, incluides text indexe to enable full-text search:

```bash
./docuflow export mongodb://127.0.0.1:27017/ecommerce -c categories --indexes
# Output:
# 1761942957881993000.ensure-indexes.categories.json
```

### 6. Geospatial Index for Warehouses

Export indexes for warehouses, includes geospatial indexe for location-based queries:

```bash
./docuflow export mongodb://127.0.0.1:27017/ecommerce -c warehouses --indexes
# Output:
# 1761944629540342600.ensure-indexes.warehouses.json
```

# Importing to New Environment

To seed a new development or testing database with this data:

```bash
# Navigate to the samples/ecommerce directory
cd samples/ecommerce

# Import all seed data and indexes
docuflow import mongodb://localhost:27017/ecommerce-dev
```

DocuFlow will automatically:
- Create collections with initial seed data
- Apply all indexes (login fields, text search, geospatial)
- Track imported files to prevent duplicates

## Use Cases

This example is perfect for:
- **Development environment setup** - Quickly seed local databases
- **Testing pipelines** - Consistent test data across CI/CD runs
- **Schema migration** - Version control your database structure
- **Team collaboration** - Share database state through Git

## Next Steps

- Add to your Git repository for version control
- Integrate into your CI/CD pipeline for automated database setup
- Explore data transformation with update migrations (see [docs/update-template.md](../../docs/update-template.md))
