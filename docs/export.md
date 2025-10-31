# Export Guide

Complete guide to exporting MongoDB collections with DocuFlow.

## Table of Contents

- [Basic Usage](#basic-usage)
- [Export Options](#export-options)
- [Query Filters](#query-filters)
- [Index Export](#index-export)
- [Batch Size](#batch-size)
- [Examples](#examples)

## Basic Usage

Export a collection:

```bash
docuflow export mongodb://localhost:27017/mydb -c users
```

This creates two types of files:

1. **Data files**: `{ticks}.upsert.{collection}-{partition}.json`
2. **Index files**: `{ticks}.ensure-indexes.{collection}.json`

## Export Options

### Required Options

| Option | Short | Description |
|--------|-------|-------------|
| `--collection` | `-c` | Collection name to export |

### Optional Options

| Option | Short | Description |
|--------|-------|-------------|
| `--query` | `-q` | Filter query in JSON format |
| `--query-file` | `-qf` | Read filter query from file |
| `--query-prompt` | `-qp` | Prompt for filter query interactively |
| `--batch-size` | `-b` | Items per file (default: 10000, max: 100000) |
| `--indexes` | | Export indexes in ensure mode |
| `--replace-indexes` | | Export indexes in replace mode |
| `--timeout` | `-t` | Timeout in seconds (default: 30) |
| `--verbose` | `-v` | Enable verbose output |

## Query Filters

### Inline Query

Pass a JSON query directly on the command line:

```bash
docuflow export mongodb://localhost:27017/mydb -c users -q '{"status":"active"}'
```

```bash
docuflow export mongodb://localhost:27017/mydb -c orders -q '{"total":{"$gte":100}}'
```

### Query from File

Store complex queries in a file:

**query.json:**
```json
{
  "createdAt": {
    "$gte": "2024-01-01T00:00:00Z",
    "$lt": "2024-12-31T23:59:59Z"
  },
  "status": {"$in": ["active", "pending"]}
}
```

Then export:

```bash
docuflow export mongodb://localhost:27017/mydb -c users -qf query.json
```

### Interactive Query Prompt

Prompt for query at runtime:

```bash
docuflow export mongodb://localhost:27017/mydb -c users -qp
```

You'll be prompted:
```
Enter filter query (JSON format, or press Enter for no filter):
```

Enter your query (or press Enter for no filter):
```json
{"role": "admin", "active": true}
```

## Index Export

### Ensure Mode (Default)

Export indexes without removing existing ones:

```bash
docuflow export mongodb://localhost:27017/mydb -c users --indexes
```

Creates: `{ticks}.ensure-indexes.users.json`

When imported, these indexes will be created if they don't exist. Existing indexes are preserved.

### Replace Mode

Export indexes and remove extras on import:

```bash
docuflow export mongodb://localhost:27017/mydb -c users --replace-indexes
```

Creates: `{ticks}.replace-indexes.users.json`

When imported:
- Indexes in the file are created/kept
- Indexes NOT in the file are removed
- The `_id` index is always preserved

## Batch Size

Control how many documents per file:

```bash
# 50,000 documents per file
docuflow export mongodb://localhost:27017/mydb -c users -b 50000
```

**Default**: 10,000 documents per file
**Maximum**: 100,000 documents per file

Large collections are automatically split:
- `{ticks}.upsert.users-0.json` (first 10,000)
- `{ticks}.upsert.users-1.json` (next 10,000)
- `{ticks}.upsert.users-2.json` (next 10,000)
- etc.

## Examples

### Export Active Users Only

```bash
docuflow export mongodb://localhost:27017/mydb -c users -q '{"status":"active"}'
```

### Export Orders from 2024

```bash
docuflow export mongodb://localhost:27017/mydb -c orders \
  -q '{"createdAt":{"$gte":"2024-01-01","$lt":"2025-01-01"}}'
```

### Export with Custom Batch Size

```bash
docuflow export mongodb://localhost:27017/mydb -c logs -b 50000
```

### Export Products with Replace Indexes

```bash
docuflow export mongodb://localhost:27017/mydb -c products --replace-indexes
```

### Export Config Collection (Small Data)

```bash
docuflow export mongodb://localhost:27017/mydb -c config -b 100
```

### Complex Filter from File

**admin-users-query.json:**
```json
{
  "role": "admin",
  "lastLogin": {"$gte": "2024-01-01"},
  "$or": [
    {"department": "IT"},
    {"permissions": {"$in": ["super_admin", "system_admin"]}}
  ]
}
```

```bash
docuflow export mongodb://localhost:27017/mydb -c users -qf admin-users-query.json
```

### Export with Verbose Output

```bash
docuflow export mongodb://localhost:27017/mydb -c users --verbose
```

Output includes:
- Connection details
- Query information
- Document counts
- File operations
- Performance metrics

### Export from Remote Database

```bash
docuflow export mongodb://user:pass@prod.example.com:27017/mydb -c users \
  -q '{"role":"tester"}' \
  -b 5000
```

### Export Multiple Collections

```bash
# Export users
docuflow export mongodb://localhost:27017/mydb -c users

# Export orders
docuflow export mongodb://localhost:27017/mydb -c orders

# Export products
docuflow export mongodb://localhost:27017/mydb -c products --replace-indexes
```

## File Output

### Data Files

Format: `{ticks}.upsert.{collection}-{partition}.json`

Example:
```
638730123456789.upsert.users-0.json
638730123456789.upsert.users-1.json
```

Content:
```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com",
    "status": "active"
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "name": "Jane Smith",
    "email": "jane@example.com",
    "status": "active"
  }
]
```

### Index Files

Format: `{ticks}.ensure-indexes.{collection}.json` or `{ticks}.replace-indexes.{collection}.json`

Example:
```
638730123456789.ensure-indexes.users.json
```

Content:
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
  }
]
```

## Best Practices

1. **Use filters for large collections** - Export only what you need
2. **Adjust batch size** - Smaller batches for better Git diffs
3. **Export indexes separately** - Use `--replace-indexes` for strict control
4. **Store queries in files** - Reuse complex filters
5. **Use verbose mode** - Debug export issues
6. **Version control exports** - Commit to Git for history

## Troubleshooting

### Connection Timeout

Increase timeout for slow connections:

```bash
docuflow export mongodb://remote:27017/mydb -c large_collection -t 120
```

### Large Collections

Use smaller batch sizes and filters:

```bash
docuflow export mongodb://localhost:27017/mydb -c logs \
  -q '{"level":"error"}' \
  -b 5000
```

### Memory Issues

Export in batches with filters:

```bash
# Export January data
docuflow export mongodb://localhost:27017/mydb -c events \
  -q '{"month":1}' -b 10000

# Export February data
docuflow export mongodb://localhost:27017/mydb -c events \
  -q '{"month":2}' -b 10000
```

## Next Steps

- Learn about [Import Options](import.md)
- Explore [Data Migrations](migrations.md)
- Check out [Examples](../examples/)
