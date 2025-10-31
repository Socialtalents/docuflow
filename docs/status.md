# Status Guide

Complete guide to checking MongoDB database connectivity with DocuFlow.

## Table of Contents

- [Basic Usage](#basic-usage)
- [Command Options](#command-options)
- [Examples](#examples)
- [Error Messages](#error-messages)
- [Troubleshooting](#troubleshooting)

## Basic Usage

Check if a MongoDB database is accessible:

```bash
docuflow status mongodb://localhost:27017/mydb
```

The status command verifies:
- Connection to MongoDB server
- Database accessibility
- Database version information

## Command Options

### Required Arguments

| Argument | Description |
|----------|-------------|
| `<connection-string>` | MongoDB connection string |

### Optional Options

| Option | Short | Description |
|--------|-------|-------------|
| `--timeout` | `-t` | Timeout in seconds (default: 30) |
| `--verbose` | `-v` | Enable verbose output |

## Examples

### Check Local Database

```bash
docuflow status mongodb://localhost:27017/mydb
```

Output:
```
Checking status of database: mongodb://localhost:27017/mydb
✓ Database is accessible
✓ Database version: 7.0.4
```

### Check Remote Database

```bash
docuflow status mongodb://user:pass@prod.example.com:27017/mydb
```

### Check with Custom Timeout

For slow or remote connections, increase the timeout:

```bash
docuflow status mongodb://remote:27017/mydb -t 60
```

### Check with Verbose Output

Get detailed connection information:

```bash
docuflow status mongodb://localhost:27017/mydb --verbose
```

Verbose output includes:
```
Checking status of database: mongodb://localhost:27017/mydb
[VERBOSE] Verifying connection...
✓ Database is accessible
[VERBOSE] Retrieving buildInfo from 'admin'...
✓ Database version: 7.0.4
```

### Check MongoDB Atlas

```bash
docuflow status "mongodb+srv://user:pass@cluster.mongodb.net/mydb"
```

Note: Use quotes around connection strings with special characters.

### Check Replica Set

```bash
docuflow status "mongodb://host1:27017,host2:27017,host3:27017/mydb?replicaSet=rs0"
```

## Error Messages

The status command provides user-friendly error messages for common connection issues:

### Connection Refused

```
Connection failed:
Cannot connect to MongoDB - the server appears to be offline or unreachable.
Please verify:
  - MongoDB is running
  - The connection string is correct
  - Firewall rules allow the connection
```

**Common causes:**
- MongoDB service is not running
- Wrong port number
- Firewall blocking the connection

### Connection Timeout

```
Connection failed:
Connection timeout - unable to reach MongoDB server.
Please verify:
  - The server address and port are correct
  - The server is running and accepting connections
  - Network connectivity is working
  - Try increasing timeout with --timeout flag
```

**Common causes:**
- Server is unreachable over the network
- Network latency issues
- Server is under heavy load

### Authentication Failed

```
Connection failed:
Authentication failed - invalid credentials.
Please verify:
  - Username and password are correct
  - User has appropriate permissions
  - Authentication database is correct
```

**Common causes:**
- Wrong username or password
- User doesn't exist in the database
- Incorrect authentication database (add `?authSource=admin` if needed)

### DNS Resolution Error

```
Connection failed:
Cannot resolve hostname.
Please verify:
  - The hostname in the connection string is correct
  - DNS is configured properly
```

**Common causes:**
- Hostname typo
- DNS configuration issues
- Hostname doesn't exist

## Troubleshooting

### Verify MongoDB is Running

**On Linux/macOS:**
```bash
# Check if MongoDB is running
ps aux | grep mongod

# Or check systemd status
sudo systemctl status mongod
```

**On Windows:**
```powershell
# Check MongoDB service
Get-Service MongoDB

# Or check running processes
Get-Process mongod
```

### Test Basic Connectivity

Use `mongosh` (MongoDB Shell) to verify connection:

```bash
mongosh mongodb://localhost:27017/mydb
```

If `mongosh` can't connect, the issue is with MongoDB itself, not DocuFlow.

### Check Firewall Rules

**On Linux:**
```bash
# Check if port 27017 is open
sudo netstat -tlnp | grep 27017

# Or use ss
ss -tlnp | grep 27017
```

**On Windows:**
```powershell
# Check listening ports
netstat -ano | findstr :27017
```

### Test Network Connectivity

```bash
# Test if server is reachable
ping prod.example.com

# Test if port is accessible
telnet prod.example.com 27017

# Or use nc (netcat)
nc -zv prod.example.com 27017
```

### Connection String Format

MongoDB connection strings must follow this format:

```
mongodb://[username:password@]host[:port][/database][?options]
```

**Examples:**
```bash
# Local database
mongodb://localhost:27017/mydb

# With authentication
mongodb://admin:password123@localhost:27017/mydb

# With auth database specified
mongodb://admin:password123@localhost:27017/mydb?authSource=admin

# Replica set
mongodb://host1:27017,host2:27017/mydb?replicaSet=rs0

# MongoDB Atlas (SRV format)
mongodb+srv://user:pass@cluster.mongodb.net/mydb
```

### Common Connection String Issues

1. **Missing database name**: Always include `/database` in the connection string
   ```bash
   # Wrong
   mongodb://localhost:27017

   # Correct
   mongodb://localhost:27017/mydb
   ```

2. **Special characters in password**: URL-encode special characters
   ```bash
   # If password is "p@ss:word"
   mongodb://user:p%40ss%3Aword@localhost:27017/mydb
   ```

3. **Missing authentication source**:
   ```bash
   # If user is in admin database
   mongodb://admin:pass@localhost:27017/mydb?authSource=admin
   ```

## Verbose Mode Details

Enable verbose mode to see detailed connection flow:

```bash
docuflow status mongodb://localhost:27017/mydb -v
```

Verbose output shows:
- Connection attempt details
- Ping verification
- Database version query
- Raw error messages (if any)

This is useful for:
- Debugging connection issues
- Understanding connection flow
- Reporting issues

## Exit Codes

The status command uses standard exit codes:

| Exit Code | Meaning |
|-----------|---------|
| 0 | Success - database is accessible |
| 1 | Failure - connection error or database unreachable |

Use exit codes in scripts:

```bash
if docuflow status mongodb://localhost:27017/mydb; then
    echo "Database is up"
else
    echo "Database is down"
    exit 1
fi
```

## Best Practices

1. **Regular Health Checks**: Use status command in monitoring scripts
2. **Pre-deployment Verification**: Check database before running migrations
3. **Timeout Configuration**: Set appropriate timeouts for remote databases
4. **Connection Pooling**: Status command doesn't leave connections open
5. **Security**: Avoid hardcoding credentials in scripts

## Integration Examples

### Shell Script Health Check

```bash
#!/bin/bash

DB_URL="mongodb://localhost:27017/mydb"

if docuflow status "$DB_URL" > /dev/null 2>&1; then
    echo "✓ Database is healthy"
    exit 0
else
    echo "✗ Database is unhealthy"
    exit 1
fi
```

### Pre-deployment Check

```bash
#!/bin/bash

echo "Checking database connectivity..."
if ! docuflow status mongodb://prod:27017/mydb -t 10; then
    echo "Error: Cannot connect to production database"
    echo "Deployment aborted"
    exit 1
fi

echo "Database is accessible, proceeding with deployment..."
# Run migrations, imports, etc.
```

### Monitoring Script

```bash
#!/bin/bash

DATABASES=(
    "mongodb://db1:27017/app1"
    "mongodb://db2:27017/app2"
    "mongodb://db3:27017/app3"
)

for db in "${DATABASES[@]}"; do
    echo "Checking $db..."
    if docuflow status "$db" -t 5; then
        echo "✓ OK"
    else
        echo "✗ FAILED"
        # Send alert
    fi
    echo ""
done
```

## Next Steps

- Learn about [Export Command](export.md)
- Learn about [Import Command](import.md)
- Check out [Examples](../examples/)
