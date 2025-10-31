# Import Guide

Complete guide to importing MongoDB collections with DocuFlow.

## Table of Contents

- [Basic Usage](#basic-usage)
- [Import Options](#import-options)
- [File Operations](#file-operations)
- [Import Modes](#import-modes)
- [Sync State Tracking](#sync-state-tracking)
- [Examples](#examples)
- [Best Practices](#best-practices)

## Basic Usage

Import all JSON files from the current directory:

```bash
docuflow import mongodb://localhost:27017/mydb
```

Import filtered by specific collection:

```bash
docuflow import mongodb://localhost:27017/mydb -c users
```

DocuFlow automatically:
- Detects JSON files matching the expected format
- Determines the operation type (import, upsert, update, indexes)
- Tracks import status to avoid duplicates
- Skips already imported files

## Import Options

### Required Arguments

| Argument | Description |
|----------|-------------|
| `<connection-string>` | MongoDB connection string |

### Optional Options

| Option | Short | Description |
|--------|-------|-------------|
| `--collection` | `-c` | Specific collection to import (imports all if omitted) |
| `--timeout` | `-t` | Timeout in seconds (default: 30) |
| `--verbose` | `-v` | Enable verbose output |

## File Operations

DocuFlow supports multiple operation types based on filename:

### Operation Types

| Operation | Filename Pattern | Description |
|-----------|-----------------|-------------|
| **import** | `{ticks}.import.{collection}-{partition}.json` | Insert documents (fails on duplicates) |
| **upsert** | `{ticks}.upsert.{collection}-{partition}.json` | Replace existing or insert new documents |
| **update** | `{ticks}.update.{collection}.json` | Update documents using aggregation pipeline |
| **ensure-indexes** | `{ticks}.ensure-indexes.{collection}.json` | Create indexes (keeps existing) |
| **replace-indexes** | `{ticks}.replace-indexes.{collection}.json` | Drop unlisted indexes, create/keep listed ones |

## Import Modes

### 1. Import Mode (Insert Only)

Inserts new documents. Fails if document with same `_id` already exists.

**File format:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com"
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "name": "Jane Smith",
    "email": "jane@example.com"
  }
]
```

**Use case:** Initial data load when collection is empty.

### 2. Upsert Mode (Replace or Insert)

Replaces existing documents or inserts new ones based on `_id`.

**File format:** Same as import mode

**Behavior:**
- If document with `_id` exists: replaces entire document
- If document doesn't exist: inserts new document
- All documents must have `_id` field

**Use case:** Syncing data, ensuring specific state of documents.

### 3. Update Mode (Aggregation Pipeline)

Updates documents using MongoDB aggregation pipeline.

**Single operation format:**
```json
{
  "description": "Add status field to all users without one",
  "filter": {"status": {"$exists": false}},
  "update": [
    {"$set": {"status": "active", "updatedAt": "$$NOW"}}
  ],
  "options": {}
}
```

**Multiple operations format:**
```json
{
  "description": "User data migration",
  "operations": [
    {
      "description": "Normalize email addresses",
      "filter": {"email": {"$exists": true}},
      "update": [
        {"$set": {"email": {"$toLower": "$email"}}}
      ]
    },
    {
      "description": "Add default role",
      "filter": {"role": {"$exists": false}},
      "update": [
        {"$set": {"role": "user"}}
      ]
    }
  ]
}
```

**Use case:** Data migrations, schema updates, computed field additions.

### 4. Ensure Indexes Mode

Creates indexes without removing existing ones.

**File format:**
```json
[
  {
    "key": {"email": 1},
    "name": "email_1",
    "unique": true
  },
  {
    "key": {"status": 1, "createdAt": -1},
    "name": "status_1_createdAt_-1"
  },
  {
    "key": {"createdAt": 1},
    "name": "createdAt_1",
    "expireAfterSeconds": 86400
  }
]
```

**Behavior:**
- Creates indexes if they don't exist
- Keeps all existing indexes
- Safe for adding new indexes

**Use case:** Adding new indexes without affecting existing ones.

### 5. Replace Indexes Mode

Ensures only specified indexes exist (removes others).

**File format:** Same as ensure-indexes mode

**Behavior:**
- Creates indexes from file if they don't exist
- Removes indexes NOT in the file
- Always preserves `_id` index
- Useful for strict index management

**Use case:** Removing old indexes, ensuring exact index configuration.

## Sync State Tracking

DocuFlow tracks imported files in the `_syncstate` collection to prevent duplicate imports.

### _syncstate Collection Structure

```javascript
{
  _id: "638730123456789.upsert.users-0.json",
  fileName: "638730123456789.upsert.users-0.json",
  successful: true,
  lastImport: ISODate("2024-01-15T10:30:00Z"),
  error: ""  // Empty if successful
}
```

### Behavior

- **Already imported files are skipped** automatically
- **Failed imports are recorded** with error message
- **Files can be re-imported** by removing the `_syncstate` entry

## Examples

### Import All Files

Import all JSON files in the current directory:

```bash
docuflow import mongodb://localhost:27017/mydb
```

Output:
```
Importing data to: mongodb://localhost:27017/mydb
✓ Processed 638730123456789.upsert.users-0.json (1000 documents, operation: upsert)
✓ Processed 638730123456789.upsert.users-1.json (850 documents, operation: upsert)
✓ Processed 638730123456789.ensure-indexes.users.json (2 indexes created, operation: ensure-indexes)

Total items processed: 1850
✓ Import completed successfully
```

### Import Specific Collection

```bash
docuflow import mongodb://localhost:27017/mydb -c users
```

Only processes files matching pattern `*.users*.json`.

### Import with Verbose Output

```bash
docuflow import mongodb://localhost:27017/mydb --verbose
```

Verbose output includes:
```
[VERBOSE] Connecting to MongoDB...
[VERBOSE] Connected successfully
[VERBOSE] Found 3 file(s) to import
[VERBOSE] Processing 638730123456789.upsert.users-0.json (operation: upsert)...
[VERBOSE] File size: 2.45 MB
[VERBOSE] Upserting 1000 documents into users
[VERBOSE] Upserted: 1000, Modified: 0, Matched: 0
✓ Processed 638730123456789.upsert.users-0.json (1000 documents, operation: upsert)
```

### Import from Remote Database

```bash
docuflow import mongodb://user:pass@prod.example.com:27017/mydb -t 120
```

### Skip Already Imported Files

DocuFlow automatically skips files already in `_syncstate`:

```bash
docuflow import mongodb://localhost:27017/mydb
```

Output:
```
Importing data to: mongodb://localhost:27017/mydb
✓ Processed 638730123456789.upsert.users-0.json (1000 documents, operation: upsert)
✓ Import completed successfully
```

Run again:
```
Importing data to: mongodb://localhost:27017/mydb
No import files found in current directory
✓ Import completed successfully
```

With verbose mode, you'll see:
```
[VERBOSE] Skipping 638730123456789.upsert.users-0.json (already imported)
```

## Error Handling

### Duplicate Key Error (Import Mode)

```
Import failed: failed to import 638730123456789.import.users-0.json:
failed to insert documents: E11000 duplicate key error
```

**Solution:** Use upsert mode instead of import mode.

### Missing _id Field (Upsert Mode)

```
Import failed: document at index 5 missing required _id field
```

**Solution:** Ensure all documents have `_id` field when using upsert mode.

### Invalid Update Pipeline

```
Import failed: failed to execute update operation 0: invalid pipeline stage
```

**Solution:** Check update pipeline syntax in the JSON file.

### Connection Timeout

```
Import failed: failed to connect to MongoDB: context deadline exceeded
```

**Solution:** Increase timeout with `-t` flag:
```bash
docuflow import mongodb://remote:27017/mydb -t 120
```

## Troubleshooting

### Files Not Being Imported

Check verbose mode to see why:

```bash
docuflow import mongodb://localhost:27017/mydb -v
```

Common reasons:
- File already imported (check `_syncstate`)
- Filename doesn't match pattern
- Operation type not recognized
- Collection filter doesn't match

### Import Takes Too Long

For large imports:

1. **Increase timeout**:
   ```bash
   docuflow import mongodb://localhost:27017/mydb -t 300
   ```

2. **Import specific collection**:
   ```bash
   docuflow import mongodb://localhost:27017/mydb -c users
   ```

3. **Check MongoDB performance**:
   - Indexes might be slowing inserts
   - Server might be under load

### Memory Issues

For very large files:
- Export with smaller batch size
- Import files in multiple runs
- Increase server memory

## Next Steps

- Learn about [Export Options](export.md)
- Learn about [Status Command](status.md)
- Check out [Examples](../examples/)
