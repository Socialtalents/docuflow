# DocuFlow - MongoDB Schema Change Management & Migration Tool

**Version-controlled MongoDB database management** - Export, import, and migrate MongoDB collections and indexes in your CI/CD pipeline.

## What is DocuFlow?

DocuFlow is a powerful command-line tool for MongoDB schema change management and database state management through file-based workflows. It enables:

- **Schema Version Control** - Track database schemas, indexes, and seed data in Git
- **Schema Migration** - Apply controlled changes across environments
- **Environment Synchronization** - Keep development, staging, and production in sync
- **Data Migrations** - Transform data using MongoDB aggregation pipelines
- **Partial Exports** - Export filtered datasets with custom queries
- **Index Management** - Ensure or replace indexes across deployments

Perfect for:
- MongoDB schema versioning and GitOps workflows
- CI/CD and automated database deployments
- Database change management with code review practices
- Data migrations and transformations
- Seeding test environments
- Backing up specific collections or filtered data

## Use Cases

### Schema Change Management
Track and apply MongoDB schema changes across environments. DocuFlow enables database-as-code practices by storing schemas, indexes, and migrations in version control. Review database changes through pull requests just like application code.

### CI/CD Integration
Incorporate DocuFlow into your continuous integration and deployment pipeline for automated database updates. Perfect for GitOps workflows and infrastructure as code practices. Enable code review processes for database changes before deployment.

### MongoDB DevOps
Streamline MongoDB operations with file-based state management. Handle index changes, data migrations, and environment synchronization as part of your DevOps workflow.

## Installation

### Via NPM (Recommended)

Install globally using npm:

```bash
npm install -g @socialtalents/docuflow
```

Or use with npx without installing:

```bash
npx @socialtalents/docuflow status mongodb://localhost:27017/mydb
```

**Advantages:**
- ✅ Automatic platform detection (Linux, macOS, Windows)
- ✅ Automatic architecture detection (x64, ARM64)
- ✅ Easy updates with `npm update -g @socialtalents/docuflow`
- ✅ Works with Node.js version managers (nvm, fnm)
- ✅ No manual PATH configuration needed

### Download Pre-built Binaries

Download the latest release for your platform from our download page:

[Downloads](https://www.socialtalents.com/docuflow/latest/index.html)

**Manual Setup:**

**Linux/macOS:**
```bash
# Download the binary for your platform
# For Linux x64:
wget https://www.socialtalents.com/docuflow/docuflow-linux-amd64

# Make it executable
chmod +x docuflow-linux-amd64

# Move to a directory in your PATH (optional)
sudo mv docuflow-linux-amd64 /usr/local/bin/docuflow
```

**Windows:**
```powershell
# Download docuflow-windows-amd64.exe from the download page
# Add the directory to your PATH or run directly
```

### Request Full License

You may use it freely for localhost database. In case you are ready to use it in production please [request license here](https://forms.office.com/r/uwB9ZH47UU)

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

# Create pull request for review
git push origin feature/update-config

# After code review and merge, deploy to staging
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

- **Schema versioning** - Track MongoDB schemas in Git
- **Multiple export modes** - Full collection, filtered, or partitioned
- **Flexible imports** - Insert, upsert, or update operations
- **Index management** - Ensure or replace indexes with change tracking
- **Query filters** - Inline, from file, or interactive prompt
- **State tracking** - Prevents duplicate imports with sync state
- **Data migrations** - Aggregation pipeline transformations
- **Batch processing** - Handles large collections efficiently
- **CI/CD ready** - Integrate with automated deployment pipelines
- **Cross-platform** - Linux, Windows, macOS (Intel & ARM)

## Version

Current version: 1.0.0

---

Built for MongoDB developers who love version control.