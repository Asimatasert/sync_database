# PostgreSQL Database Sync Tool

A powerful bash-based tool for syncing PostgreSQL databases between remote and local servers via SSH. Supports multiple databases, schema/table exclusions, automated restore operations, compression, automatic cleanup, and Telegram notifications.

## ⚠️ Important: SSH Access Required

**Before using this tool, you MUST configure SSH key-based authentication to your remote server.**

This script requires passwordless SSH access to the remote server where your source database is located. You need to set up RSA key authentication first.

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

- ✅ **SSH-based remote database dumps** - Connect to remote servers via SSH (RSA key authentication)
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
cd postgres-sync-tool
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

### Single Database Sync

Use `sync_database` for single database operations:

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
      "name": "Production Database",
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
| `source.database` | string | Yes | Source database name |
| `source.host` | string | No | Source DB host (default: localhost) |
| `source.port` | number | No | Source DB port (default: 5432) |
| `source.user` | string | No | Source DB user (default: postgres) |
| `source.password` | string | Yes | Source DB password |
| `remote.ssh_user` | string | No | SSH username (default: your_ssh_user) |
| `remote.ssh_host` | string | Yes | SSH server IP/hostname |
| `remote.ssh_port` | number | No | SSH port (default: 22) |
| `destination.database` | string | No | Destination DB name (default: same as source) |
| `destination.host` | string | No | Destination DB host (default: localhost) |
| `destination.port` | number | No | Destination DB port (default: 5432) |
| `destination.user` | string | No | Destination DB user (default: postgres) |
| `destination.password` | string | Conditional | Required if restore is true |
| `destination.restore` | boolean | Yes | Auto-restore after dump |
| `options.exclude_tables` | array | No | Tables to exclude (e.g., ["audit.logged_actions"]) |
| `options.exclude_schemas` | array | No | Schemas to exclude (e.g., ["test_schema"]) |
| `options.compression.enabled` | boolean | No | Enable compression (default: true) |
| `options.compression.type` | string | No | Compression type: gzip, xz, bzip2, none (default: gzip) |
| `options.compression.level` | number | No | Compression level 1-9 (default: 6) |
| `options.compression.pg_dump_level` | number | No | PostgreSQL dump compression 0-9 (default: 6) |
| `options.keep_dumps` | number | No | Keep only last N dumps (overrides global setting) |

#### Global Options

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `global_options.log_file` | string | No | Log file path (default: ./data/dumps/sync.log) |
| `global_options.keep_dumps` | number | No | Default keep dumps for all databases (default: 3) |
| `global_options.telegram.enabled` | boolean | No | Enable Telegram notifications (default: false) |
| `global_options.telegram.bot_token` | string | Conditional | Telegram bot token (required if enabled) |
| `global_options.telegram.chat_id` | string | Conditional | Telegram chat ID (required if enabled) |

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

## ⏰ Scheduling with Cron

Add to crontab (`crontab -e`):

```bash
# Daily at 2 AM
0 2 * * * cd /path/to/postgres-sync-tool && bash sync_database_runner >> ./data/dumps/cron.log 2>&1

# Every 6 hours
0 */6 * * * cd /path/to/postgres-sync-tool && bash sync_database_runner

# Every Sunday at 3 AM
0 3 * * 0 cd /path/to/postgres-sync-tool && bash sync_database_runner

# Weekdays at 1 AM
0 1 * * 1-5 cd /path/to/postgres-sync-tool && bash sync_database_runner
```

## 📂 File Structure

```
.
├── sync_database                # Main sync script
├── sync_database_runner         # Multi-database runner
├── sync_database.json           # Your configuration
├── example_sync_database.json   # Example configurations
├── README.MD                    # This file
└── data/
    └── dumps/                   # Dump files location
        ├── sync.log             # Log file
        └── dbname_*.dump        # Database dumps
```

## 🔍 Command Line Options

### sync_database

```
Options:
  --database NAME            Source database name
  --exclude TABLES           Comma-separated list of tables to exclude
  --exclude-schema SCHEMAS   Comma-separated list of schemas to exclude

Remote Server Options:
  --remote-user USER         SSH username
  --remote-ip IP             SSH server IP (required)
  --remote-port PORT         SSH port (default: 22)

Source Database Options:
  --source-host HOST         Source DB host on remote server (default: localhost)
  --source-port PORT         Source DB port (default: 5432)
  --source-user USER         Source DB username (default: postgres)
  --source-password PASS     Source DB password (required)

Destination Database Options:
  --restore                  Automatically restore after sync
  --dest-database NAME       Destination DB name (default: same as source)
  --dest-host HOST           Destination DB host (default: localhost)
  --dest-port PORT           Destination DB port (default: 5432)
  --dest-user USER           Destination DB username (default: postgres)
  --dest-password PASS       Destination DB password (required for restore)

Compression Options:
  --compression TYPE         Compression type: gzip, xz, bzip2, none (default: gzip)
  --compression-level LEVEL  Compression level 1-9 (default: 6)
  --pg-compression LEVEL     PostgreSQL dump compression 0-9 (default: 6)

Dump Management:
  --keep-dumps N             Keep only last N dumps, delete older ones (default: 0, keep all)

  --help                     Show help message
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
