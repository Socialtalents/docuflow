# Update Template Guide

Complete guide to generating migration templates with DocuFlow.

## Table of Contents

- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Available Templates](#available-templates)
- [Template Details](#template-details)
- [Examples](#examples)
- [Best Practices](#best-practices)

## Overview

The `update-template` command generates pre-configured migration file templates for common data transformation scenarios. These templates can be customized and then applied using the `import` command.

### Why Use Templates?

- **Quick Start**: Generate migration files instantly
- **Best Practices**: Templates follow MongoDB aggregation pipeline patterns
- **Reduce Errors**: Pre-validated JSON structure
- **Common Patterns**: Cover typical migration scenarios

## Basic Usage

Generate a template for a specific collection:

```bash
docuflow update-template <template-type> -c <collection>
```

This creates a file named `{ticks}.update.{collection}.json` that can be edited and imported.

### Command Options

| Option | Short | Description |
|--------|-------|-------------|
| `--collection` | `-c` | Collection name (required for auto-naming) |
| `--output` | `-o` | Custom output filename (optional) |

## Available Templates

| Template | Description | Use Case |
|----------|-------------|----------|
| **unset-field** | Remove a field from documents | Cleanup unused fields |
| **backfill-field** | Add missing field with default value | Add new required fields |
| **convert-field** | Convert field type using mapping | String to number conversion |
| **rename-field** | Rename a field in documents | Refactor field names |
| **combine-fields** | Combine multiple fields into one | Denormalize data |
| **multi-operation** | Multiple migration steps in sequence | Complex migrations |

## Template Details

### 1. unset-field

Remove a field from all documents where it exists.

**Generate:**
```bash
docuflow update-template unset-field -c users
```

**Generated template:**
```json
{
  "description": "Remove field from documents",
  "filter": {
    "fieldToRemove": {"$exists": true}
  },
  "update": [
    {
      "$unset": "fieldToRemove"
    }
  ]
}
```

**Example customization:**
```json
{
  "description": "Remove deprecated 'middleName' field",
  "filter": {
    "middleName": {"$exists": true}
  },
  "update": [
    {
      "$unset": "middleName"
    }
  ]
}
```

**Use cases:**
- Remove deprecated fields
- Clean up temporary fields
- Remove fields after data migration

---

### 2. backfill-field

Add a missing field to documents with a default value.

**Generate:**
```bash
docuflow update-template backfill-field -c users
```

**Generated template:**
```json
{
  "description": "Add missing field with default value",
  "filter": {
    "fieldName": {"$exists": false}
  },
  "update": [
    {
      "$set": {
        "fieldName": "defaultValue",
        "backfilledAt": "$$NOW"
      }
    }
  ]
}
```

**Example customization:**
```json
{
  "description": "Add default role for users without one",
  "filter": {
    "role": {"$exists": false}
  },
  "update": [
    {
      "$set": {
        "role": "user",
        "roleAssignedAt": "$$NOW"
      }
    }
  ]
}
```

**Use cases:**
- Add new required fields to existing documents
- Set default values for null fields
- Initialize fields with current timestamp

---

### 3. convert-field

Convert field types using conditional mapping.

**Generate:**
```bash
docuflow update-template convert-field -c orders
```

**Generated template:**
```json
{
  "description": "Convert field type using mapping",
  "operations": [
    {
      "description": "Convert string status to numeric code",
      "filter": {
        "status": {"$type": "string"}
      },
      "update": [
        {
          "$set": {
            "statusCode": {
              "$switch": {
                "branches": [
                  {"case": {"$eq": ["$status", "pending"]}, "then": 1},
                  {"case": {"$eq": ["$status", "active"]}, "then": 2},
                  {"case": {"$eq": ["$status", "completed"]}, "then": 3},
                  {"case": {"$eq": ["$status", "cancelled"]}, "then": 4}
                ],
                "default": 0
              }
            },
            "convertedAt": "$$NOW"
          }
        }
      ]
    }
  ]
}
```

**Example customization:**
```json
{
  "description": "Convert priority from text to number",
  "operations": [
    {
      "description": "Map priority strings to numbers",
      "filter": {
        "priority": {"$type": "string"}
      },
      "update": [
        {
          "$set": {
            "priorityLevel": {
              "$switch": {
                "branches": [
                  {"case": {"$eq": ["$priority", "low"]}, "then": 1},
                  {"case": {"$eq": ["$priority", "medium"]}, "then": 2},
                  {"case": {"$eq": ["$priority", "high"]}, "then": 3},
                  {"case": {"$eq": ["$priority", "urgent"]}, "then": 4}
                ],
                "default": 2
              }
            }
          }
        }
      ]
    }
  ]
}
```

**Use cases:**
- Convert enums from strings to integers
- Normalize data formats
- Map legacy values to new schema

---

### 4. rename-field

Rename a field while preserving its value.

**Generate:**
```bash
docuflow update-template rename-field -c products
```

**Generated template:**
```json
{
  "description": "Rename a field in documents",
  "filter": {
    "oldFieldName": {"$exists": true},
    "newFieldName": {"$exists": false}
  },
  "update": [
    {
      "$set": {
        "newFieldName": "$oldFieldName"
      }
    },
    {
      "$unset": "oldFieldName"
    }
  ]
}
```

**Example customization:**
```json
{
  "description": "Rename 'userName' to 'username'",
  "filter": {
    "userName": {"$exists": true},
    "username": {"$exists": false}
  },
  "update": [
    {
      "$set": {
        "username": "$userName"
      }
    },
    {
      "$unset": "userName"
    }
  ]
}
```

**Use cases:**
- Refactor field naming conventions
- Fix typos in field names
- Align with API contract changes

---

### 5. combine-fields

Merge multiple fields into a single field.

**Generate:**
```bash
docuflow update-template combine-fields -c users
```

**Generated template:**
```json
{
  "description": "Combine multiple fields into one",
  "filter": {
    "firstName": {"$exists": true},
    "lastName": {"$exists": true}
  },
  "update": [
    {
      "$set": {
        "fullName": {
          "$concat": ["$firstName", " ", "$lastName"]
        }
      }
    }
  ]
}
```

**Example customization:**
```json
{
  "description": "Create full address from parts",
  "filter": {
    "street": {"$exists": true},
    "city": {"$exists": true},
    "zipCode": {"$exists": true}
  },
  "update": [
    {
      "$set": {
        "fullAddress": {
          "$concat": [
            "$street", ", ",
            "$city", " ",
            {"$toString": "$zipCode"}
          ]
        }
      }
    }
  ]
}
```

**Use cases:**
- Denormalize data for faster queries
- Create computed fields
- Generate display names

---

### 6. multi-operation

Execute multiple migration steps in sequence.

**Generate:**
```bash
docuflow update-template multi-operation -c products
```

**Generated template:**
```json
{
  "description": "Multiple migration operations",
  "operations": [
    {
      "description": "First migration step",
      "filter": {"condition1": true},
      "update": [
        {
          "$set": {
            "newField": "value"
          }
        }
      ]
    },
    {
      "description": "Second migration step",
      "filter": {"condition2": {"$exists": false}},
      "update": [
        {
          "$set": {
            "anotherField": 123
          }
        }
      ]
    }
  ]
}
```

**Example customization:**
```json
{
  "description": "User data normalization",
  "operations": [
    {
      "description": "Lowercase all email addresses",
      "filter": {"email": {"$exists": true}},
      "update": [
        {
          "$set": {
            "email": {"$toLower": "$email"}
          }
        }
      ]
    },
    {
      "description": "Add full name if missing",
      "filter": {
        "fullName": {"$exists": false},
        "firstName": {"$exists": true},
        "lastName": {"$exists": true}
      },
      "update": [
        {
          "$set": {
            "fullName": {
              "$concat": ["$firstName", " ", "$lastName"]
            }
          }
        }
      ]
    },
    {
      "description": "Set default status",
      "filter": {"status": {"$exists": false}},
      "update": [
        {
          "$set": {
            "status": "active"
          }
        }
      ]
    }
  ]
}
```

**Use cases:**
- Complex migrations with multiple steps
- Sequential data transformations
- Atomic multi-step updates

## Examples

### Example 1: Remove Deprecated Field

```bash
# Generate template
docuflow update-template unset-field -c users

# Edit the generated file (e.g., 638730123456789.update.users.json)
# Change "fieldToRemove" to "legacyId"

# Apply migration
docuflow import mongodb://localhost:27017/mydb
```

### Example 2: Add Default Role

```bash
# Generate template
docuflow update-template backfill-field -c users

# Edit file to set role field
# Apply migration
docuflow import mongodb://localhost:27017/mydb
```

### Example 3: Convert Status Field

```bash
# Generate template
docuflow update-template convert-field -c orders

# Customize status mappings
# Apply migration
docuflow import mongodb://localhost:27017/mydb
```

### Example 4: Custom Output Filename

```bash
# Generate with custom filename
docuflow update-template rename-field -c products -o product-migration-2024.json

# Edit and apply
docuflow import mongodb://localhost:27017/mydb
```

### Example 5: Multi-Step Migration

```bash
# Generate multi-operation template
docuflow update-template multi-operation -c users

# Add multiple migration steps in the operations array
# Apply all steps in sequence
docuflow import mongodb://localhost:27017/mydb
```

## Complete Workflow Example

### Scenario: Refactor User Schema

**Step 1: Generate templates**
```bash
# Combine first and last name
docuflow update-template combine-fields -c users -o 001-add-fullname.json

# Normalize email addresses
docuflow update-template multi-operation -c users -o 002-normalize-emails.json

# Add default role
docuflow update-template backfill-field -c users -o 003-add-role.json
```

**Step 2: Customize templates**

Edit `001-add-fullname.json`:
```json
{
  "description": "Add fullName field from firstName and lastName",
  "filter": {
    "firstName": {"$exists": true},
    "lastName": {"$exists": true},
    "fullName": {"$exists": false}
  },
  "update": [
    {
      "$set": {
        "fullName": {
          "$concat": ["$firstName", " ", "$lastName"]
        }
      }
    }
  ]
}
```

Edit `002-normalize-emails.json`:
```json
{
  "description": "Normalize email addresses",
  "operations": [
    {
      "description": "Convert emails to lowercase",
      "filter": {"email": {"$exists": true}},
      "update": [
        {
          "$set": {
            "email": {"$toLower": "$email"}
          }
        }
      ]
    }
  ]
}
```

Edit `003-add-role.json`:
```json
{
  "description": "Add default user role",
  "filter": {
    "role": {"$exists": false}
  },
  "update": [
    {
      "$set": {
        "role": "user",
        "roleAssignedAt": "$$NOW"
      }
    }
  ]
}
```

**Step 3: Test on dev environment**
```bash
# Apply migrations to dev
docuflow import mongodb://dev:27017/mydb -c users -v
```

**Step 4: Apply to production**
```bash
# Backup first
mongodump --uri="mongodb://prod:27017/mydb" --collection=users

# Apply migrations
docuflow import mongodb://prod:27017/mydb -c users
```

**Step 5: Verify results**
```bash
# Check migration status
mongo mongodb://prod:27017/mydb
> db._syncstate.find({fileName: /users/}).pretty()

# Verify data
> db.users.findOne()
```

## Best Practices

### 1. Always Test First

Test migrations on a development environment before production:

```bash
# Dev
docuflow import mongodb://dev:27017/mydb

# Verify results
mongo mongodb://dev:27017/mydb
> db.users.find().limit(5)

# If good, proceed to production
```

### 2. Use Descriptive Filenames

Name files with sequence numbers and descriptions:

```bash
docuflow update-template backfill-field -c users -o 001-add-user-role.json
docuflow update-template rename-field -c users -o 002-rename-username.json
docuflow update-template unset-field -c users -o 003-remove-legacy-fields.json
```

### 3. Add Detailed Descriptions

Edit the generated templates to include clear descriptions:

```json
{
  "description": "Migration #12: Add default status to all orders without one. Required for v2.0 release.",
  "filter": {"status": {"$exists": false}},
  "update": [{"$set": {"status": "pending"}}]
}
```

### 4. Use Specific Filters

Avoid updating more documents than necessary:

```json
{
  "description": "Only update users created before 2024",
  "filter": {
    "createdAt": {"$lt": "2024-01-01T00:00:00Z"},
    "role": {"$exists": false}
  },
  "update": [{"$set": {"role": "legacy"}}]
}
```

### 5. Check Document Count First

Before running migration, check how many documents will be affected:

```bash
mongo mongodb://localhost:27017/mydb
> db.users.countDocuments({"role": {"$exists": false}})
```

### 6. Version Control Migrations

Commit migration files to Git:

```bash
git add *.update.*.json
git commit -m "Add user schema migration"
git push
```

### 7. Document Migration Dependencies

If migrations must run in order, document this:

```json
{
  "description": "Migration #3: Requires migration #1 and #2 to be completed first",
  "filter": {...},
  "update": [...]
}
```

### 8. Keep Migrations Idempotent

Ensure migrations can be safely re-run:

```json
{
  "description": "Safe to run multiple times",
  "filter": {
    "newField": {"$exists": false},
    "sourceField": {"$exists": true}
  },
  "update": [{"$set": {"newField": "$sourceField"}}]
}
```

## Troubleshooting

### Template Not Found

```
Error: Unknown template 'xyz'
```

**Solution:** Check available templates:
```bash
docuflow update-template --help
```

### Missing Collection Name

```
Error: --collection or --output must be specified
```

**Solution:** Specify collection or custom output:
```bash
docuflow update-template backfill-field -c users
# or
docuflow update-template backfill-field -o my-migration.json
```

### File Already Exists

If the generated filename already exists, either:
- Wait a moment and regenerate (filenames use ticks)
- Use `-o` to specify a custom name

## MongoDB Aggregation Pipeline Reference

Templates use MongoDB aggregation pipeline syntax. Common operators:

### Set/Update Operators
- `$set`: Add or update fields
- `$unset`: Remove fields
- `$rename`: Rename fields (use template pattern instead)

### Conditional Operators
- `$cond`: Ternary conditional
- `$switch`: Multi-case conditional
- `$ifNull`: Handle null values

### String Operators
- `$concat`: Combine strings
- `$toLower`: Convert to lowercase
- `$toUpper`: Convert to uppercase
- `$trim`: Remove whitespace

### Type Conversion
- `$toString`: Convert to string
- `$toInt`: Convert to integer
- `$toDouble`: Convert to double
- `$toDate`: Convert to date

### Date Operators
- `$$NOW`: Current timestamp
- `$dateToString`: Format date
- `$dateToParts`: Extract date components

## Next Steps

- Learn about [Import Command](import.md) to apply migrations
- Learn about [Export Command](export.md) to backup data first
- Check out [Examples](../examples/) for more migration patterns
