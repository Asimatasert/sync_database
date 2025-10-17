# PostgreSQL Database Sync & Clone Tool

A powerful bash-based tool for syncing PostgreSQL databases between remote and local servers via SSH, or cloning databases locally. Supports multiple databases, schema/table exclusions, automated restore operations, compression, automatic cleanup, and Telegram notifications.

## SSH Access (For Remote Sync Only)

**Note**: SSH configuration is only needed for remote database sync. For local database cloning, you can skip this section!

For remote sync, you need to configure SSH key-based authentication to your remote server. This script requires passwordless SSH access to the remote server where your source database is located.

### Quick Setup

Run the following command to copy your SSH key to the remote server:

```bash
ssh-copy-id username@remote-ip-address
```

**Example:**
```bash
ssh-copy-id myuser@192.168.1.100
```

After running this command:
1. You will be asked for the remote user's password (one time only)
2. Your SSH public key will be copied to the remote server
3. Future connections will not require a password

### Verify SSH Access

Test that you can connect without a password:

```bash
ssh username@remote-ip-address
```

If you can login without entering a password, you're ready to use this tool!

### Generate SSH Key (if you don't have one)

If you don't have an SSH key yet, generate one first:

```bash
# Generate RSA key (press Enter for all prompts to use defaults)
ssh-keygen -t rsa -b 4096

# Copy the key to remote server
ssh-copy-id username@remote-ip-address
```

**Note**: Without SSH key authentication configured, this script will NOT work!

---

## 🚀 Features

### Core Features
- ✅ **SSH-based remote database dumps** - Connect to remote servers via SSH (RSA key authentication)
- ✅ **Local database cloning** - Clone databases on localhost without SSH
- ✅ **Automated restore** - Optionally restore dumps to local databases automatically
- ✅ **Multiple database support** - Sync multiple databases with a single command
- ✅ **Table exclusions** - Exclude specific tables from dumps
- ✅ **Schema exclusions** - Exclude entire schemas from dumps
- ✅ **JSON configuration** - Easy configuration via JSON files
- ✅ **Logging** - Comprehensive logging for tracking operations
- ✅ **Cron-ready** - Perfect for scheduled backups
- ✅ **Flexible naming** - Source and destination databases can have different names
- ✅ **Custom ports** - Support for non-standard PostgreSQL ports
- ✅ **Compression support** - Multiple compression options (gzip, xz, bzip2) with configurable levels

### Advanced Features (v2.2.0+)
- ✅ **Health Checks** - Database health monitoring (connection, disk space, replication lag, long queries, lock waits, table bloat)
- ✅ **Retention Policies** - Grandfather-Father-Son (GFS) backup rotation strategy
- ✅ **Alerting System** - Multi-channel alerts (Telegram, Webhook, Email) with severity levels
- ✅ **Streaming Dumps** - Zero-disk streaming for large databases (remote → local without intermediate files)
- ✅ **Data Masking** - Mask sensitive data (emails, phones, credit cards) for development environments
- ✅ **Multi-Source Sync** - Sync from multiple remote servers to single local database
- ✅ **Point-in-Time Recovery (PITR)** - WAL-based recovery to specific timestamps
- ✅ **Automatic dump cleanup** - Keep only last N dumps, automatically delete older ones
- ✅ **Telegram notifications** - Get instant notifications on sync status with emoji-rich messages

## 📋 Requirements

- Bash 4.0+
- PostgreSQL client tools (`pg_dump`, `pg_restore`, `psql`, `createdb`, `dropdb`)
- SSH access to remote server (RSA key authentication configured)
- `jq` (for JSON parsing in runner script)
- `curl` (for Telegram notifications, optional)

### Installing Dependencies

#### macOS

Using Homebrew:
```bash
# Install PostgreSQL client tools
brew install postgresql

# Install jq for JSON parsing
brew install jq

# Verify installation
psql --version
pg_dump --version
jq --version
```

#### Ubuntu/Debian

```bash
# Update package list
sudo apt-get update

# Install PostgreSQL client tools
sudo apt-get install -y postgresql-client

# Install jq and curl
sudo apt-get install -y jq curl

# Verify installation
psql --version
pg_dump --version
jq --version
```

#### CentOS/RHEL 7

```bash
# Install PostgreSQL repository
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL client tools
sudo yum install -y postgresql15

# Install jq and curl
sudo yum install -y jq curl

# Verify installation
psql --version
pg_dump --version
jq --version
```

#### CentOS/RHEL 8/9 & Fedora

```bash
# Install PostgreSQL repository
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL client tools
sudo dnf install -y postgresql15

# Install jq and curl
sudo dnf install -y jq curl

# Verify installation
psql --version
pg_dump --version
jq --version
```

#### Arch Linux

```bash
# Install PostgreSQL, jq and curl
sudo pacman -S postgresql jq curl

# Verify installation
psql --version
pg_dump --version
jq --version
```

#### Alpine Linux

```bash
# Install PostgreSQL client, jq and curl
apk add postgresql-client jq curl

# Verify installation
psql --version
pg_dump --version
jq --version
```

#### FreeBSD

```bash
# Install PostgreSQL client, jq and curl
pkg install postgresql15-client jq curl

# Verify installation
psql --version
pg_dump --version
jq --version
```

## 🔧 Installation

1. Clone the repository:
```bash
git clone https://github.com/Asimatasert/sync_database.git
cd sync_database
```

2. Make scripts executable:
```bash
chmod +x sync_database sync_database_runner
```

3. Configure SSH key authentication to your remote server:
```bash
ssh-copy-id user@remote-server
```

4. Copy and configure your JSON file:
```bash
cp example_sync_database.json sync_database.json
# Edit sync_database.json with your configuration
```

## 📖 Usage

### Local Database Cloning (NEW!)

Clone a database on localhost without SSH:

```bash
# Basic clone (same cluster - credentials auto-inherited)
bash sync_database --local-clone \
  --database mydb \
  --source-password 'pass' \
  --dest-database mydb_clone

# Clone with table exclusions (same cluster)
bash sync_database --local-clone \
  --database production \
  --source-password 'prod_pass' \
  --dest-database test_env \
  --exclude 'audit.logged_actions,sessions,cache'

# Clone with schema exclusions (same cluster)
bash sync_database --local-clone \
  --database mydb \
  --source-password 'pass' \
  --dest-database mydb_backup \
  --exclude-schema 'test_schema,temp_schema'

# Clone between different clusters (different ports)
bash sync_database --local-clone \
  --database mydb --source-port 5432 --source-password 'pass1' \
  --dest-database mydb_copy --dest-port 5433 --dest-password 'pass2'
```

**Important**: When source and destination have the same host and port (same PostgreSQL cluster), destination credentials are automatically inherited from source. You only need to specify different credentials when cloning between different clusters (different ports).

### Remote Database Sync

Use `sync_database` for remote sync operations:

```bash
bash sync_database \
  --database mydb \
  --source-password 'source_pass' \
  --remote-ip 192.168.1.100 \
  --restore \
  --dest-password 'dest_pass' \
  --keep-dumps 5
```

### Multiple Databases Sync

Use `sync_database_runner` with a JSON configuration file:

```bash
bash sync_database_runner [config_file]
```

If no config file is specified, it uses `sync_database.json` by default.

## 🔑 Configuration

### JSON Configuration Structure

```json
{
  "databases": [
    {
      "name": "Production Database - Remote Sync",
      "enabled": true,
      "source": {
        "database": "prod_db",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "source_password"
      },
      "remote": {
        "ssh_user": "your_ssh_user",
        "ssh_host": "192.168.1.100",
        "ssh_port": 22
      },
      "destination": {
        "database": "prod_db",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "dest_password",
        "restore": true
      },
      "options": {
        "exclude_tables": ["audit.logged_actions", "sessions"],
        "exclude_schemas": ["test_schema"],
        "compression": {
          "enabled": true,
          "type": "gzip",
          "level": 6,
          "pg_dump_level": 6
        },
        "keep_dumps": 5
      }
    },
    {
      "name": "Local Clone - Same Cluster",
      "enabled": true,
      "mode": {
        "local_clone": true
      },
      "source": {
        "database": "prod_db",
        "host": "localhost",
        "port": 5432,
        "user": "postgres",
        "password": "postgres_pass"
      },
      "destination": {
        "database": "prod_db_backup"
      },
      "options": {
        "exclude_tables": ["logs", "sessions"],
        "keep_dumps": 5
      }
    },
    {
      "name": "Local Clone - Different Clusters",
      "enabled": true,
      "mode": {
        "local_clone": true
      },
      "source": {
        "database": "dev_db",
        "host": "localhost",
        "port": 5432,
        "user": "dev_admin",
        "password": "dev_pass"
      },
      "destination": {
        "database": "dev_db_test",
        "host": "localhost",
        "port": 5433,
        "user": "test_admin",
        "password": "test_pass"
      },
      "options": {
        "exclude_tables": ["cache"],
        "keep_dumps": 3
      }
    }
  ],
  "global_options": {
    "log_file": "./data/dumps/sync.log",
    "keep_dumps": 3,
    "telegram": {
      "enabled": true,
      "bot_token": "YOUR_TELEGRAM_BOT_TOKEN",
      "chat_id": "YOUR_TELEGRAM_CHAT_ID"
    }
  }
}
```

### Configuration Options

#### Database Entry

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Descriptive name for the database |
| `enabled` | boolean | Yes | Enable/disable this database sync |
| `mode.local_clone` | boolean | No | Enable local clone mode (default: false = remote sync) |
| `source.database` | string | Yes | Source database name |
| `source.host` | string | No | Source DB host (default: localhost) |
| `source.port` | number | No | Source DB port (default: 5432) |
| `source.user` | string | No | Source DB user (default: postgres) |
| `source.password` | string | Yes | Source DB password |
| `remote.ssh_user` | string | No | SSH username (default: your_ssh_user) |
| `remote.ssh_host` | string | Conditional* | SSH server IP/hostname (*not needed for local clone) |
| `remote.ssh_port` | number | No | SSH port (default: 22) |
| `destination.database` | string | No | Destination DB name (default: same as source) |
| `destination.host` | string | No | Destination DB host (default: localhost) |
| `destination.port` | number | No | Destination DB port (default: 5432) |
| `destination.user` | string | No | Destination DB user (auto-inherited if same cluster) |
| `destination.password` | string | Conditional | Required if restore is true or different cluster |
| `destination.restore` | boolean | Conditional* | Auto-restore after dump (*ignored in local clone) |
| `options.exclude_tables` | array | No | Tables to exclude (e.g., ["audit.logged_actions"]) |
| `options.exclude_schemas` | array | No | Schemas to exclude (e.g., ["test_schema"]) |
| `options.compression.enabled` | boolean | No | Enable compression (default: true, ignored in local clone) |
| `options.compression.type` | string | No | Compression type: gzip, xz, bzip2, none (default: gzip) |
| `options.compression.level` | number | No | Compression level 1-9 (default: 6) |
| `options.compression.pg_dump_level` | number | No | PostgreSQL dump compression 0-9 (default: 6) |
| `options.keep_dumps` | number | No | Keep dump policy: -1=keep all, 0=keep none, N=keep last N (overrides global setting) |

**Important Notes:**
- When `mode.local_clone` is `true`, the `remote` section is not needed
- When source and destination have the same `host:port` (same cluster), destination credentials are auto-inherited from source
- Only specify different credentials when cloning between different clusters (different ports)

#### Global Options

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `global_options.log_file` | string | No | Log file path (default: ./data/dumps/sync.log) |
| `global_options.keep_dumps` | number | No | Default keep dump policy: -1=keep all, 0=keep none, N=keep last N (default: -1) |
| `global_options.telegram.enabled` | boolean | No | Enable Telegram notifications (default: false) |
| `global_options.telegram.bot_token` | string | Conditional | Telegram bot token (required if enabled) |
| `global_options.telegram.chat_id` | string | Conditional | Telegram chat ID (required if enabled) |
| `global_options.health_checks.enabled` | boolean | No | Enable health checks (default: true) |
| `global_options.health_checks.pre_sync` | boolean | No | Run health checks before sync (default: true) |
| `global_options.health_checks.post_sync` | boolean | No | Run health checks after sync (default: false) |
| `global_options.retention_policy.enabled` | boolean | No | Enable GFS retention (default: false) |
| `global_options.retention_policy.strategy` | string | No | Retention strategy (default: gfs) |
| `global_options.retention_policy.daily` | number | No | Daily backups to keep (default: 7) |
| `global_options.retention_policy.weekly` | number | No | Weekly backups to keep (default: 4) |
| `global_options.retention_policy.monthly` | number | No | Monthly backups to keep (default: 12) |
| `global_options.retention_policy.yearly` | number | No | Yearly backups to keep (default: 3) |
| `global_options.alerting.webhook.url` | string | No | Webhook URL for alerts |
| `global_options.alerting.email.to` | string | No | Email address for alerts |
| `global_options.alerting.min_severity` | string | No | Minimum alert severity (default: warning) |
| `options.streaming` | boolean | No | Enable streaming dumps (default: false) |
| `options.data_masking.enabled` | boolean | No | Enable data masking (default: false) |
| `options.data_masking.rules_file` | string | Conditional | Masking rules JSON file path |

### Setting up Telegram Notifications

1. **Create a Telegram Bot:**
   - Open Telegram and search for `@BotFather`
   - Send `/newbot` and follow the instructions
   - Copy the bot token provided

2. **Get Your Chat ID:**
   - Send a message to your bot
   - Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
   - Find your `chat_id` in the response

3. **Configure in JSON:**
   ```json
   "telegram": {
     "enabled": true,
     "bot_token": "123456789:ABCdefGHIjklMNOpqrsTUVwxyz",
     "chat_id": "987654321"
   }
   ```

**Notification Features:**
- ✅ Success notifications for each database
- ❌ Failure notifications for each database
- 📊 Summary notification with overall statistics
- Rich HTML formatting with emojis

## 📚 Examples

### Example 1: Basic Sync with Auto-Cleanup

```bash
bash sync_database \
  --database mydb \
  --source-password 'secret123' \
  --remote-ip 192.168.1.100 \
  --restore \
  --dest-password 'local123' \
  --keep-dumps 5
```

### Example 2: Exclude Tables and Schema

```bash
bash sync_database \
  --database mydb \
  --source-password 'secret123' \
  --remote-ip 192.168.1.100 \
  --exclude 'audit.logged_actions,sessions,cache' \
  --exclude-schema 'test_schema' \
  --restore \
  --dest-password 'local123' \
  --keep-dumps 3
```

### Example 3: With Custom Compression

```bash
bash sync_database \
  --database mydb \
  --source-password 'secret123' \
  --remote-ip 192.168.1.100 \
  --compression xz \
  --compression-level 9 \
  --restore \
  --dest-password 'local123'
```

### Example 4: Different Database Names

```bash
bash sync_database \
  --database production \
  --source-password 'prod_pass' \
  --remote-ip 192.168.1.100 \
  --restore \
  --dest-database development \
  --dest-password 'dev_pass' \
  --keep-dumps 7
```

### Example 5: Dump Only (No Restore, No Compression)

```bash
bash sync_database \
  --database mydb \
  --source-password 'secret123' \
  --remote-ip 192.168.1.100 \
  --compression none
```

### Example 6: Multiple Databases via Config

Create `sync_database.json`:

```json
{
  "databases": [
    {
      "name": "Hotel Database",
      "enabled": true,
      "source": {
        "database": "hotel_db",
        "password": "pass1"
      },
      "remote": {
        "ssh_host": "192.168.1.100"
      },
      "destination": {
        "password": "local_pass1",
        "restore": true
      },
      "options": {
        "exclude_tables": ["audit.logged_actions"],
        "exclude_schemas": [],
        "compression": {
          "enabled": true
        },
        "keep_dumps": 3
      }
    },
    {
      "name": "Resort Database",
      "enabled": true,
      "source": {
        "database": "resort_db",
        "password": "pass2"
      },
      "remote": {
        "ssh_host": "192.168.1.100"
      },
      "destination": {
        "password": "local_pass2",
        "restore": true
      },
      "options": {
        "exclude_tables": ["sessions", "cache"],
        "exclude_schemas": [],
        "compression": {
          "enabled": false
        },
        "keep_dumps": 5
      }
    }
  ],
  "global_options": {
    "log_file": "./data/dumps/sync.log",
    "keep_dumps": 3,
    "telegram": {
      "enabled": true,
      "bot_token": "YOUR_BOT_TOKEN",
      "chat_id": "YOUR_CHAT_ID"
    }
  }
}
```

Run:
```bash
bash sync_database_runner
```

## 🔬 Advanced Features

### Health Checks

Monitor database health before and after sync operations:

**Features:**
- Connection testing
- Disk space monitoring
- Replication lag detection
- Long-running query detection
- Lock wait monitoring
- Table bloat analysis

**Configuration (JSON):**
```json
"global_options": {
  "health_checks": {
    "enabled": true,
    "pre_sync": true,
    "post_sync": false,
    "connection_timeout": 10,
    "disk_space_threshold": 80,
    "replication_lag_threshold": 10,
    "long_query_threshold": 300,
    "lock_wait_threshold": 5,
    "bloat_threshold": 30
  }
}
```

**CLI Usage:**
```bash
bash sync_database --database mydb \
  --source-password 'pass' \
  --health-check-pre \
  --health-check-post \
  --remote-ip 192.168.1.100 \
  --restore
```

### Retention Policies (GFS)

Grandfather-Father-Son backup rotation strategy automatically manages backup lifecycle:

**Strategy:**
- **Daily**: Keep last 7 days
- **Weekly**: Keep last 4 weeks
- **Monthly**: Keep last 12 months
- **Yearly**: Keep last 3 years

**Configuration (JSON):**
```json
"global_options": {
  "retention_policy": {
    "enabled": true,
    "strategy": "gfs",
    "daily": 7,
    "weekly": 4,
    "monthly": 12,
    "yearly": 3
  }
}
```

**CLI Usage:**
```bash
bash sync_database --database mydb \
  --source-password 'pass' \
  --retention-policy gfs \
  --retention-daily 7 \
  --retention-weekly 4 \
  --retention-monthly 12 \
  --retention-yearly 3
```

### Alerting System

Multi-channel alerting with severity levels:

**Channels:**
- Telegram (with HTML formatting)
- Webhook (JSON POST)
- Email (SMTP)

**Severity Levels:** info, warning, error, critical

**Configuration (JSON):**
```json
"global_options": {
  "alerting": {
    "webhook": {
      "url": "https://your-webhook-url.com/alerts",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    },
    "email": {
      "enabled": true,
      "smtp_host": "smtp.gmail.com",
      "smtp_port": 587,
      "smtp_user": "your-email@gmail.com",
      "smtp_password": "your-app-password",
      "from": "alerts@yourdomain.com",
      "to": "admin@yourdomain.com"
    },
    "min_severity": "warning"
  }
}
```

**CLI Usage:**
```bash
bash sync_database --database mydb \
  --source-password 'pass' \
  --alert-webhook "https://your-webhook.com/alerts" \
  --alert-email "admin@example.com" \
  --alert-severity warning
```

### Streaming Dumps

Zero-disk streaming for large databases (no intermediate dump files):

**Benefits:**
- Reduced disk usage
- Faster transfers
- Real-time progress monitoring
- Direct remote → local pipeline

**How it works:**
```
SSH → pg_dump | gzip → local gunzip | pg_restore
```

**Configuration (JSON):**
```json
{
  "name": "Large Database - Streaming",
  "options": {
    "streaming": true
  }
}
```

**CLI Usage:**
```bash
bash sync_database --database large_db \
  --source-password 'pass' \
  --remote-ip 192.168.1.100 \
  --streaming \
  --restore
```

**Note:** Streaming mode is ideal for:
- Very large databases (>100GB)
- Limited disk space scenarios
- Network transfers with low latency

### Data Masking

Mask sensitive data for development/testing environments:

**Supported Patterns:**
- Email addresses (user@example.com → u***@e***.com)
- Phone numbers (+1234567890 → +12****7890)
- Credit cards (1234567890123456 → 1234********3456)
- SSN patterns (XXX-XX-XXXX → ***-**-****)

**Default Masking (CLI):**
```bash
bash sync_database --database prod_db \
  --source-password 'pass' \
  --data-masking \
  --restore
```

**Custom Rules (JSON):**
```json
{
  "name": "Production - Masked for Dev",
  "options": {
    "data_masking": {
      "enabled": true,
      "rules_file": "./masking_rules.json"
    }
  }
}
```

**Example Masking Rules File:**
```json
{
  "rules": [
    {
      "table": "users",
      "column": "email",
      "method": "email_mask",
      "description": "Mask user email addresses"
    },
    {
      "table": "users",
      "column": "phone",
      "method": "phone_mask",
      "description": "Mask user phone numbers"
    },
    {
      "table": "customers",
      "column": "credit_card",
      "method": "credit_card_mask",
      "description": "Mask credit card numbers"
    },
    {
      "table": "employees",
      "column": "ssn",
      "method": "anonymize",
      "description": "Anonymize social security numbers"
    }
  ]
}
```

**Masking Methods:**
- `email_mask`: Partially masks email addresses
- `phone_mask`: Masks middle digits of phone numbers
- `credit_card_mask`: Shows first/last 4 digits only
- `anonymize`: Replaces with random data

### Multi-Source Sync

Sync from multiple remote servers to a single local database:

**Use Cases:**
- Consolidating data from multiple branches
- Merging regional databases
- Development environment setup

**Configuration (JSON):**
```json
{
  "name": "Multi-Source Consolidation",
  "mode": {
    "multi_source": true
  },
  "sources": [
    {
      "remote_host": "branch1.example.com",
      "database": "sales_db",
      "user": "postgres",
      "password": "pass1"
    },
    {
      "remote_host": "branch2.example.com",
      "database": "sales_db",
      "user": "postgres",
      "password": "pass2"
    }
  ],
  "destination": {
    "database": "consolidated_sales",
    "host": "localhost",
    "port": 5432
  },
  "options": {
    "strategy": "latest"
  }
}
```

**Strategies:**
- `latest`: Use most recent source
- `merge_all`: Merge all sources (requires conflict resolution)

### Point-in-Time Recovery (PITR)

WAL-based recovery to specific timestamps:

**Setup Steps:**

1. **Configure WAL Archiving:**
```bash
# Edit postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /path/to/wal_archives/%f && cp %p /path/to/wal_archives/%f'

# Restart PostgreSQL
sudo systemctl restart postgresql
```

2. **Create Base Backup:**
```bash
bash sync_database --database mydb \
  --source-password 'pass' \
  --pitr-base-backup
```

3. **Restore to Point in Time:**
```bash
bash sync_database --database mydb \
  --pitr-restore '2025-01-17 14:30:00'
```

**Configuration (JSON):**
```json
"global_options": {
  "pitr": {
    "enabled": true,
    "wal_archive_dir": "./data/wal_archives",
    "base_backup_dir": "./data/base_backups"
  }
}
```

**Note:** PITR requires:
- Continuous WAL archiving enabled on source database
- Regular base backups
- Sufficient storage for WAL archives

## ⏰ Scheduling with Cron

Add to crontab (`crontab -e`):

```bash
# Daily at 2 AM - Remote sync runner
0 2 * * * cd /path/to/sync_database && bash sync_database_runner >> ./data/dumps/cron.log 2>&1

# Every 6 hours - Remote sync
0 */6 * * * cd /path/to/sync_database && bash sync_database_runner

# Clone production to test database every Sunday at 3 AM
0 3 * * 0 cd /path/to/sync_database && bash sync_database --local-clone --database production --source-password 'pass' --dest-database test_env --dest-password 'pass'

# Weekdays at 1 AM - Remote sync
0 1 * * 1-5 cd /path/to/sync_database && bash sync_database_runner
```

## 📂 File Structure

```
.
├── sync_database                  # Main sync script
├── sync_database_runner           # Multi-database runner
├── sync_database.json             # Your configuration
├── sync_database_example.json     # Example configuration with all features
├── example_sync_database.json     # Basic example configurations
├── README.md                      # This file
│
├── Feature Modules (sourced by main script):
├── health_checks                  # Database health monitoring
├── retention_policies             # GFS backup rotation
├── alerting                       # Multi-channel alerting system
├── streaming                      # Zero-disk streaming dumps
├── data_masking                   # Sensitive data masking
├── multi_source                   # Multi-source sync (simplified)
├── pitr                           # Point-in-Time Recovery (simplified)
│
└── data/
    ├── dumps/                     # Dump files location
    │   ├── sync.log               # Log file
    │   └── dbname_*.dump          # Database dumps
    ├── wal_archives/              # WAL archives for PITR
    └── base_backups/              # Base backups for PITR
```

## 🔍 Command Line Options

### sync_database

```
Basic Options:
  --database NAME              Source database name
  --exclude TABLES             Comma-separated list of tables to exclude
  --exclude-schema SCHEMAS     Comma-separated list of schemas to exclude
  --local-clone                Enable local database cloning mode (no SSH)
  --help                       Show help message

Remote Server Options (for remote sync):
  --remote-user USER           SSH username
  --remote-ip IP               SSH server IP (required for remote sync)
  --remote-port PORT           SSH port (default: 22)

Source Database Options:
  --source-host HOST           Source DB host (default: localhost)
  --source-port PORT           Source DB port (default: 5432)
  --source-user USER           Source DB username (default: postgres)
  --source-password PASS       Source DB password (required)

Destination Database Options:
  --restore                    Automatically restore after sync
  --dest-database NAME         Destination DB name (default: same as source)
  --dest-host HOST             Destination DB host (default: localhost)
  --dest-port PORT             Destination DB port (default: 5432)
  --dest-user USER             Destination DB username (default: postgres)
  --dest-password PASS         Destination DB password (required for restore)

Compression Options:
  --compression TYPE           Compression type: gzip, xz, bzip2, none (default: gzip)
  --compression-level LEVEL    Compression level 1-9 (default: 6)
  --pg-compression LEVEL       PostgreSQL dump compression 0-9 (default: 6)

Dump Management:
  --keep-dumps N               Keep dump policy: -1=keep all, 0=keep none, N=keep last N (default: -1)

Health Check Options:
  --health-check               Enable health checks (default: enabled)
  --no-health-check            Disable health checks
  --health-check-pre           Run health checks before sync
  --health-check-post          Run health checks after sync

Retention Policy Options:
  --retention-policy STRATEGY  Enable retention policy (default: gfs)
  --retention-daily N          Keep N daily backups (default: 7)
  --retention-weekly N         Keep N weekly backups (default: 4)
  --retention-monthly N        Keep N monthly backups (default: 12)
  --retention-yearly N         Keep N yearly backups (default: 3)

Alerting Options:
  --alert-webhook URL          Webhook URL for alerts
  --alert-email EMAIL          Email address for alerts
  --alert-severity LEVEL       Minimum alert severity: info, warning, error, critical

Advanced Options:
  --streaming                  Enable streaming dumps (zero-disk mode)
  --no-progress                Disable progress monitoring in streaming mode
  --data-masking               Enable data masking for sensitive information
  --masking-rules FILE         JSON file with custom masking rules

PITR Options:
  --pitr-base-backup           Create a base backup for PITR
  --pitr-restore TIME          Restore to specific point in time
```

## 🔒 Security Considerations

1. **SSH Keys**: Always use SSH key authentication instead of passwords
2. **Passwords**: Consider using environment variables or secure vaults instead of plain text passwords in JSON
3. **File Permissions**: Protect your configuration files:
   ```bash
   chmod 600 sync_database.json
   ```
4. **Network**: Use VPN or secure networks when syncing databases
5. **Firewall**: Ensure PostgreSQL ports are properly firewalled
6. **Telegram Bot Token**: Keep your bot token secret, never commit it to public repositories

## 📊 Logging

Logs are written to `./data/dumps/sync.log` by default. Each entry includes:
- Timestamp
- Log level (INFO, ERROR, SUCCESS)
- Operation details

Example log entries:
```
[2025-01-16 14:35:00] [INFO] ===== Sync Runner Started =====
[2025-01-16 14:35:05] [INFO] Starting sync for: Production Database
[2025-01-16 14:36:20] [SUCCESS] Completed sync for: Production Database
[2025-01-16 14:36:20] [INFO] Telegram notification sent successfully
[2025-01-16 14:36:20] [INFO] ===== Sync Runner Completed =====
```

## 🐛 Troubleshooting

### "Password authentication failed"

**Problem**: PostgreSQL rejects password authentication.

**Solutions**:
1. Configure PostgreSQL to accept md5 authentication:
   ```bash
   sudo nano /etc/postgresql/*/main/pg_hba.conf
   # Change 'peer' to 'md5' for local connections
   host    all    all    127.0.0.1/32    md5
   sudo systemctl restart postgresql
   ```

2. Set password for postgres user:
   ```bash
   sudo -u postgres psql
   ALTER USER postgres WITH PASSWORD 'your_password';
   ```

### "jq: command not found"

**Solution**: Install jq:
```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# CentOS/RHEL
sudo yum install jq
```

### "Permission denied (publickey)"

**Problem**: SSH key authentication not configured.

**Solution**:
```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id user@remote-server
```

### Dump file is too small

**Problem**: Database might be empty or wrong credentials.

**Solutions**:
1. Verify database has data:
   ```bash
   ssh user@remote-server "psql -U postgres -d dbname -c '\dt'"
   ```

2. Check credentials are correct

3. Verify database name is correct

### Telegram notifications not working

**Problem**: Notifications not being sent.

**Solutions**:
1. Verify bot token and chat ID are correct
2. Check if curl is installed: `curl --version`
3. Test manually:
   ```bash
   curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
     -d "chat_id=<CHAT_ID>" \
     -d "text=Test message"
   ```
4. Check logs for error messages

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- PostgreSQL community
- Bash scripting community
- Telegram Bot API

## 📞 Support

For issues and questions:
- Open an issue on GitHub

## 🔄 Changelog

### Version 2.2.0 (2025-10-17)
- **NEW**: Health Checks - Database monitoring (connection, disk, replication, queries, locks, bloat)
- **NEW**: Retention Policies - GFS (Grandfather-Father-Son) backup rotation strategy
- **NEW**: Alerting System - Multi-channel alerts (Telegram, Webhook, Email) with severity levels
- **NEW**: Streaming Dumps - Zero-disk streaming for large databases
- **NEW**: Data Masking - Mask sensitive data (emails, phones, credit cards, SSN)
- **NEW**: Multi-Source Sync - Consolidate from multiple remote servers
- **NEW**: Point-in-Time Recovery (PITR) - WAL-based recovery to specific timestamps
- Enhanced JSON configuration with all features configurable
- Modular architecture with separate feature files
- Improved error handling and logging
- Comprehensive documentation for advanced features

### Version 2.1.0 (2025-01-17)
- **NEW**: Added local database cloning (`--local-clone` flag)
- Clone databases on localhost without SSH
- Added validation to prevent same source/destination names in local clone mode
- Updated documentation with local cloning examples

### Version 2.0.0 (2025-01-17)
- Added compression support (gzip, xz, bzip2)
- Added automatic dump cleanup (keep last N dumps)
- Added Telegram notification support
- Enhanced JSON configuration options
- Improved error handling
- Better logging

### Version 1.0.0 (2025-01-16)
- Initial release
- Basic sync functionality
- Multiple database support
- Table and schema exclusions
- JSON configuration
- Automated restore
- Logging support
