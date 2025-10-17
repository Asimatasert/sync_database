# Example Scripts for sync_database

This document contains example scripts and crontab entries for using sync_database and sync_database_runner.

---

## Table of Contents
- [sync_database vs sync_database_runner](#sync_database-vs-sync_database_runner)
- [Basic Usage Examples (sync_database)](#basic-usage-examples-sync_database)
- [Runner Usage Examples (sync_database_runner)](#runner-usage-examples-sync_database_runner)
- [Crontab Examples](#crontab-examples)
- [Advanced Examples](#advanced-examples)

---

## sync_database vs sync_database_runner

### sync_database
- Single database sync script
- All parameters passed via command line
- Good for one-time syncs or simple cron jobs
- Usage: `bash sync_database [OPTIONS]`

### sync_database_runner
- Batch runner for multiple databases
- Configuration managed via JSON file
- Supports multiple databases with different settings
- Sends Telegram notifications with sync status
- Provides summary reports with disk usage
- Usage: `bash sync_database_runner [config_file]`

**When to use which?**
- Use `sync_database` for single, simple syncs
- Use `sync_database_runner` for managing multiple databases with centralized configuration

---

## Basic Usage Examples (sync_database)

### Example 1: Simple Dump (No Restore)
Dump a database from remote server without restoring locally:

```bash
bash sync_database \
  --database 'mydb' \
  --source-host 'localhost' \
  --source-port 5432 \
  --source-user 'postgres' \
  --source-password 'mypassword' \
  --remote-user 'sshuser' \
  --remote-ip '192.168.1.100' \
  --remote-port 22
```

### Example 2: Dump and Restore
Dump from remote server and restore to local database:

```bash
bash sync_database \
  --database 'mydb' \
  --source-host 'localhost' \
  --source-port 5432 \
  --source-user 'postgres' \
  --source-password 'remote_password' \
  --remote-user 'sshuser' \
  --remote-ip '192.168.1.100' \
  --remote-port 22 \
  --restore \
  --dest-database 'mydb' \
  --dest-host 'localhost' \
  --dest-port 5432 \
  --dest-user 'postgres' \
  --dest-password 'local_password'
```

### Example 3: With Excluded Tables
Dump and restore while excluding specific tables:

```bash
bash sync_database \
  --database 'mydb' \
  --source-host 'localhost' \
  --source-port 5432 \
  --source-user 'postgres' \
  --source-password 'remote_password' \
  --remote-user 'sshuser' \
  --remote-ip '192.168.1.100' \
  --remote-port 22 \
  --restore \
  --dest-database 'mydb' \
  --dest-host 'localhost' \
  --dest-port 5432 \
  --dest-user 'postgres' \
  --dest-password 'local_password' \
  --exclude "logs,sessions,temp_data"
```

---

## Runner Usage Examples (sync_database_runner)

### Example Config File (sync_database.json)
```json
{
  "global_options": {
    "keep_dumps": 5,
    "log_file": "./data/dumps/sync.log",
    "telegram": {
      "enabled": true,
      "bot_token": "123456789:ABCdefGHIjklMNOpqrsTUVwxyz",
      "chat_id": "-1001234567890"
    }
  },
  "databases": [
    {
      "name": "Production DB",
      "enabled": true,
      "source": {
        "database": "proddb",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "SecurePass123!"
      },
      "remote": {
        "ssh_user": "sshuser",
        "ssh_host": "prod.example.com",
        "ssh_port": 22
      },
      "destination": {
        "database": "proddb",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "LocalPass456",
        "restore": true
      },
      "options": {
        "exclude_tables": ["audit.logged_actions", "public.sessions"],
        "exclude_schemas": [],
        "compression": {
          "enabled": true,
          "type": "gzip",
          "level": 6,
          "pg_dump_level": 6
        }
      }
    },
    {
      "name": "Staging DB",
      "enabled": false,
      "source": {
        "database": "stagingdb",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "StagePass789"
      },
      "remote": {
        "ssh_user": "sshuser",
        "ssh_host": "staging.example.com",
        "ssh_port": 22
      },
      "destination": {
        "database": "stagingdb",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "LocalPass456",
        "restore": true
      },
      "options": {
        "exclude_tables": [],
        "exclude_schemas": [],
        "compression": {
          "enabled": true
        }
      }
    }
  ]
}
```

### Run with Default Config
```bash
bash sync_database_runner
```
This will look for `sync_database.json` in the same directory as the script.

### Run with Custom Config
```bash
bash sync_database_runner /path/to/custom_config.json
```

### What You'll Get
- Individual notifications for each database sync (with start/end time, duration, and dump size)
- Summary notification with:
  - Total databases processed
  - Success/Failed/Skipped counts
  - Disk usage information
  - Overall status
- Detailed logs in the specified log file
- Automatic cleanup of old dumps (if `keep_dumps` is set)

---

## Crontab Examples

### Using sync_database_runner (Recommended for Multiple Databases)

#### Every 10 Minutes - Run All Enabled Databases
```bash
*/10 * * * * /bin/bash /home/dbuser/sync_database_runner >> /var/log/sync_database_runner.log 2>&1
```

#### Every Hour - Run All Enabled Databases
```bash
0 * * * * /bin/bash /home/dbuser/sync_database_runner >> /var/log/sync_database_runner.log 2>&1
```

#### Every Day at 2 AM - Run All Enabled Databases
```bash
0 2 * * * /bin/bash /home/dbuser/sync_database_runner >> /var/log/sync_database_runner.log 2>&1
```

#### With Custom Config File
```bash
*/10 * * * * /bin/bash /home/dbuser/sync_database_runner /etc/db_sync/custom_config.json >> /var/log/sync_database_runner.log 2>&1
```

---

### Using sync_database (For Single Database)

#### Every 10 Minutes - Production Database
```bash
*/10 * * * * /bin/bash /home/dbuser/sync_database --database 'proddb' --source-host 'localhost' --source-port 5432 --source-user 'postgres' --source-password 'SecurePass123!' --remote-user 'sshuser' --remote-ip '10.0.1.100' --remote-port 22 --restore --dest-database 'proddb' --dest-host 'localhost' --dest-port 5432 --dest-user 'postgres' --dest-password 'LocalPass456' --exclude "audit.logged_actions,public.sessions,public.logs,public.temp_data" --compression gzip --compression-level 6 --pg-compression 6 >> /var/log/sync_database_prod.log 2>&1
```

#### Every 10 Minutes - Staging Database
```bash
*/10 * * * * /bin/bash /home/dbuser/sync_database --database 'stagingdb' --source-host 'localhost' --source-port 5432 --source-user 'postgres' --source-password 'StagePass789' --remote-user 'sshuser' --remote-ip 'staging.example.com' --remote-port 22 --restore --dest-database 'stagingdb' --dest-host 'localhost' --dest-port 5432 --dest-user 'postgres' --dest-password 'LocalPass456' --exclude "audit.logged_actions" --compression gzip --compression-level 6 --pg-compression 6 >> /var/log/sync_database_staging.log 2>&1
```

#### Every Hour at Minute 0
```bash
0 * * * * /bin/bash /home/dbuser/sync_database --database 'mydb' --source-password 'MySecurePass' --remote-ip '192.168.1.100' >> /var/log/sync_database.log 2>&1
```

#### Every Day at 2 AM
```bash
0 2 * * * /bin/bash /home/dbuser/sync_database --database 'mydb' --source-password 'MySecurePass' --remote-ip '192.168.1.100' >> /var/log/sync_database.log 2>&1
```

#### Every 6 Hours
```bash
0 */6 * * * /bin/bash /home/dbuser/sync_database --database 'mydb' --source-password 'MySecurePass' --remote-ip '192.168.1.100' >> /var/log/sync_database.log 2>&1
```

#### Every Sunday at 3 AM
```bash
0 3 * * 0 /bin/bash /home/dbuser/sync_database --database 'mydb' --source-password 'MySecurePass' --remote-ip '192.168.1.100' >> /var/log/sync_database.log 2>&1
```

---

## Advanced Examples

### With Keep Dumps (Keep Last 5 Dumps)
```bash
bash sync_database \
  --database 'mydb' \
  --source-password 'password' \
  --remote-ip '192.168.1.100' \
  --keep-dumps 5
```

### Without Compression
```bash
bash sync_database \
  --database 'mydb' \
  --source-password 'password' \
  --remote-ip '192.168.1.100' \
  --compression none
```

### With XZ Compression (Best Compression)
```bash
bash sync_database \
  --database 'mydb' \
  --source-password 'password' \
  --remote-ip '192.168.1.100' \
  --compression xz \
  --compression-level 9 \
  --pg-compression 9
```

### Excluding Schemas
```bash
bash sync_database \
  --database 'mydb' \
  --source-password 'password' \
  --remote-ip '192.168.1.100' \
  --exclude-schema "temp,logs,audit"
```

### Multiple Exclusions (Tables and Schemas)
```bash
bash sync_database \
  --database 'mydb' \
  --source-password 'password' \
  --remote-ip '192.168.1.100' \
  --exclude "logs,sessions" \
  --exclude-schema "temp,audit"
```

---

## Crontab Timing Reference

```
* * * * *
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

### Common Crontab Patterns
- `*/5 * * * *` - Every 5 minutes
- `*/10 * * * *` - Every 10 minutes
- `*/15 * * * *` - Every 15 minutes
- `*/30 * * * *` - Every 30 minutes
- `0 * * * *` - Every hour
- `0 */2 * * *` - Every 2 hours
- `0 0 * * *` - Every day at midnight
- `0 2 * * *` - Every day at 2 AM
- `0 0 * * 0` - Every Sunday at midnight
- `0 0 1 * *` - First day of every month at midnight

---

## Tips

1. **Special Characters in Passwords**: If your password contains special characters like `%`, `$`, `*`, `!`, wrap them in single quotes and consider escaping them in crontab.

2. **Log Files**: Always redirect output to log files in crontab:
   ```bash
   >> /var/log/sync_database.log 2>&1
   ```

3. **Testing**: Test your command manually before adding to crontab:
   ```bash
   bash /home/ustek/sync_database --database 'test' ...
   ```

4. **View Crontab**:
   ```bash
   crontab -l
   ```

5. **Edit Crontab**:
   ```bash
   crontab -e
   ```

6. **View Logs**:
   ```bash
   tail -f /var/log/sync_database.log
   ```
