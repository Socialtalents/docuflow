# DocuFlow

**Docuflow for MongoDB - State Management Tool** - Export, import, and migrate MongoDB collections with ease.

## What is DocuFlow?

DocuFlow is a powerful command-line tool for managing MongoDB database state through file-based workflows. It enables:

- **Version Control** your database schemas and seed data
- **Environment Synchronization** between development, staging, and production
- **Data Migrations** using aggregation pipelines
- **Partial Exports** with custom filters
- **Index Management** with ensure or replace modes

Perfect for:
- Database versioning in Git
- Automated deployments
- Data migrations and transformations
- Seeding test environments
- Backing up specific collections or filtered data

## Installation

### Download Pre-built Binaries

Download the latest release for your platform from our download page:

[Downloads](https://www.socialtalents.com/docuflow/0.9.4/download.html)

### Request full license

You may use it freely for localhost database. In case you are ready to use it in production please [request license here](https://forms.office.com/r/uwB9ZH47UU)

### Quick Setup

**Linux/macOS:**
```bash
# Download the binary
wget https://github.com/yourusername/docuflow/raw/main/bin/docuflow-linux-amd64

# Make it executable
chmod +x docuflow-linux-amd64

# Move to a directory in your PATH (optional)
sudo mv docuflow-linux-amd64 /usr/local/bin/docuflow
```

**Windows:**
```powershell
# Download docuflow-windows-amd64.exe
# Add the directory to your PATH or run directly
```

## Quick Start

### Check Connection Status

Verify your MongoDB connection:

```bash
docuflow status mongodb://localhost:27017/mydb
```

### Export a Collection

Export all documents from your database:

```bash
docuflow export mongodb://localhost:27017/mydb -c users
```

This creates files like:
- `638730123456789.upsert.users-0.json`

### Export with Filter

Export only active users:

```bash
docuflow export mongodb://localhost:27017/mydb -c users -q '{"status":"active"}'
```

### Export with Interactive Filter

Prompt for filter at runtime:

```bash
docuflow export mongodb://localhost:27017/mydb -c users -qp
```

You'll be prompted:
```
Enter filter query (JSON format, or press Enter for no filter):
{"createdAt": {"$gte": "2024-01-01"}}
```

### Export Collection Indexes

Export all indexes documents dor your colleciton:

```bash
docuflow export mongodb://localhost:27017/mydb -c users --indexes
```

This creates files like:
- `638730123456789.ensure-indexes.users.json`

### Import Data

Import all files from the current directory:

```bash
docuflow import mongodb://localhost:27017/mydb
```

DocuFlow automatically:
- Detects file types (`.upsert.`, `.update.`, `.ensure-indexes.`, etc.)
- Tracks imported files in `_syncstate` collection
- Skips already-imported files
- Handles indexes and data migrations

## File Formats

DocuFlow uses a timestamp-based naming convention:

### Data Files
```
{ticks}.upsert.{collection}-{partition}.json
{ticks}.import.{collection}-{partition}.json
{ticks}.update.{collection}.json
```

### Index Files
```
{ticks}.ensure-indexes.{collection}.json
{ticks}.replace-indexes.{collection}.json
```

## Common Workflows

### Seed Development Database

```bash
# Export production data with filter
docuflow export mongodb://prod:27017/app -c users -q '{"role":"admin"}'

# Import to local database
docuflow import mongodb://localhost:27017/app-dev
```

### Version Control Workflow

```bash
# Export current state
docuflow export mongodb://localhost:27017/mydb -c config

# Commit to git
git add *.json
git commit -m "Update config data"

# Deploy to staging
git pull
docuflow import mongodb://staging:27017/mydb -c config
```

### Data Migration

```bash
# Create migration template
docuflow update-template backfill-field -c users

# Edit the generated file
# Then apply migration
docuflow import mongodb://localhost:27017/mydb
```

## Documentation

- [Export Guide](docs/export.md) - Complete export options and examples
- [Import Guide](docs/import.md) - Import modes and data migrations
- [Update Migrations](docs/migrations.md) - Data transformation using aggregation pipelines
- [Examples](examples/) - Ready-to-use example files

## Licensing

DocuFlow requires a license for non-localhost MongoDB hosts. Place your license file (e.g., `db.example.com-a3f9.license`) in the same directory as the executable.

**Localhost connections are always free** - no license required for development/testing!

Contact us for licensing information.

## Support

- **Issues**: [GitHub Issues](https://github.com/socialtalents/docuflow/issues)
- **Documentation**: [docs/](docs/)
- **Examples**: [examples/](examples/)

## Features

- Multiple export modes - full collection, filtered, or partitioned
- Flexible imports - insert, upsert, or update operations
- Index management - ensure or replace modes
- Query filters - inline, from file, or interactive prompt
- State tracking - prevents duplicate imports
- Data migrations - aggregation pipeline transformations
- Batch processing - handles large collections efficiently
- Cross-platform - Linux, Windows, macOS (Intel & ARM)

## Version

Current version: 1.0.0

---

Built for MongoDB developers who love version control.