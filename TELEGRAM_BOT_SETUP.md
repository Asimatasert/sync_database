# Telegram Bot Service Setup Guide

Control database synchronization through Telegram!

## 🚀 Features

- **Remote Control**: Start sync operations via Telegram
- **Real-time Status**: Instant status monitoring
- **Secure**: Only authorized users can run commands
- **Notifications**: Automatic operation result notifications
- **Log Monitoring**: View logs from Telegram

## 📋 Requirements

- Telegram Bot Token
- Telegram Chat ID
- jq (JSON parser)
- curl

## 🔧 Setup

### 1. Bot Token, Chat ID and Authorized Users

Configure all Telegram settings in `sync_database.json`:

```json
{
  "global_options": {
    "telegram": {
      "enabled": true,
      "bot_token": "YOUR_TELEGRAM_BOT_TOKEN",
      "chat_id": "YOUR_TELEGRAM_CHAT_ID",
      "authorized_users": [
        "123456789",
        "987654321"
      ]
    }
  }
}
```

**Explanations:**

- **`enabled`**: Enable bot service (true/false)
- **`bot_token`**: Bot token received from BotFather
- **`chat_id`**: Default chat for notifications (group or user)
- **`authorized_users`**: User IDs allowed to run bot commands (array)

**Authorized User Management:**

```json
// If empty: Everyone in the configured chat is authorized
"authorized_users": []

// Specific users: Only these IDs are authorized
"authorized_users": ["123456789", "987654321"]
```

**Getting Your User ID:**

1. Send `/start` command to the bot
2. Bot will show you your User ID
3. Add this ID to config and restart the service

### 2. First Run and Setup

When you run the bot for the first time, it automatically asks about systemd service installation:

```bash
# Start the bot
bash telegram_bot_service

# Or with custom config
bash telegram_bot_service /path/to/sync_database.json
```

**On first run:**

```
╔════════════════════════════════════════════════════════════╗
║  Systemd Service Installation                             ║
╚════════════════════════════════════════════════════════════╝

Telegram bot service is not installed on the system.

If you install as systemd service:
  ✅ Bot starts automatically (on system boot)
  ✅ Automatically restarts on crash
  ✅ Easy management with systemctl
  ✅ Log management (journalctl)

If you run manually:
  ⚠️  Bot stops when terminal closes
  ⚠️  You need to start manually each time

Install as systemd service? (y/n):
```

**Options:**

- **"y" (Yes)**: Performs automatic installation, starts service
- **"n" (No)**: Runs in manual mode, you can install later with `--install-service`

**To install service later:**

```bash
bash telegram_bot_service --install-service
```

### 3. Systemd Service Management

**Management Commands:**

```bash
# Check status
sudo systemctl status telegram-bot-sync

# View logs
sudo journalctl -u telegram-bot-sync -f

# Stop service
sudo systemctl stop telegram-bot-sync

# Restart service
sudo systemctl restart telegram-bot-sync
```

## 🤖 Telegram Bot Commands

### Basic Commands

| Command | Description |
|---------|-------------|
| `/start` | Start bot and get welcome message |
| `/help` | Show help menu |
| `/sync` | Synchronize all active databases |
| `/sync [dbname]` | Synchronize specific database |
| `/status` | Check current sync status |
| `/list` | List all databases |
| `/logs [N]` | Show last N lines of log (default: 20) |
| `/stop` | Stop running sync operation |

### Usage Examples

```
# Start full sync
/sync

# Sync specific database
/sync Database-1

# Check status
/status

# Show last 50 lines of log
/logs 50

# List databases
/list

# Stop running operation
/stop
```

## 📱 Telegram Bot Setup

If you want to create a new bot:

### 1. Creating a Bot

1. Open [@BotFather](https://t.me/BotFather) on Telegram
2. Send `/newbot` command
3. Choose a name for your bot (e.g., "Database Sync Bot")
4. Choose a username for your bot (e.g., "my_db_sync_bot")
5. BotFather will give you a Bot Token

### 2. Finding Chat ID

**For a group:**

1. Add the bot to the group
2. Send a message in the group
3. Open this URL in your browser:
   ```
   https://api.telegram.org/bot<BOT_TOKEN>/getUpdates
   ```
4. Find the chat ID in the format `"chat":{"id":-1001234567890}`

**For private messages:**

1. Send a message to the bot
2. Use the URL above
3. Find the user ID in the format `"chat":{"id":123456789}`

### 3. Updating Config

Update `sync_database.json` file:

```json
{
  "global_options": {
    "telegram": {
      "enabled": true,
      "bot_token": "YOUR_BOT_TOKEN",
      "chat_id": "YOUR_CHAT_ID",
      "authorized_users": ["123456789", "987654321"]
    }
  }
}
```

**Note:** If `authorized_users` is left empty, all users in the configured chat will be authorized.

## 🔒 Security

### Authorized User Management

All authorization settings are configured in `sync_database.json`:

```json
{
  "global_options": {
    "telegram": {
      "authorized_users": [
        "123456789",   // User 1
        "987654321",   // User 2
        "555555555"    // User 3
      ]
    }
  }
}
```

**To add/remove users:**

1. Edit the config file
2. Restart the service:
   ```bash
   sudo systemctl restart telegram-bot-sync
   ```

### Security Tips

1. **Don't share your Bot Token** - This token should not be public
2. **Only add trusted people** to the authorized users list
3. **If using a group** make it private
4. **Check log files** regularly

## 📊 Monitoring

### Log Files

```bash
# Bot logs
tail -f data/dumps/telegram_bot.log

# Sync logs
tail -f data/dumps/sync.log

# Systemd logs (if running as service)
sudo journalctl -u telegram-bot-sync -f
```

### Status Check

```bash
# Is bot running?
ps aux | grep telegram_bot_service

# Is there an active sync?
cat .sync_running.pid

# Recent messages
tail -20 data/dumps/telegram_bot.log
```

## 🐛 Troubleshooting

### Bot not responding

```bash
# Check service status
sudo systemctl status telegram-bot-sync

# Check logs
tail -50 data/dumps/telegram_bot.log

# Test bot token
curl "https://api.telegram.org/bot<BOT_TOKEN>/getMe"
```

### "Unauthorized Access" error

1. Get your User ID (visible in bot's error message)
2. Add to config file:
   ```json
   "authorized_users": ["123456789", "YOUR_USER_ID"]
   ```
3. Restart the service:
   ```bash
   sudo systemctl restart telegram-bot-sync
   ```

### Sync not starting

1. Check that `sync_database_runner` script is executable:
   ```bash
   chmod +x sync_database_runner
   ```
2. Verify config file is correct
3. Test by running manually:
   ```bash
   bash sync_database_runner
   ```

## 📝 Example Usage Scenario

```
User: /start
Bot: 👋 Welcome to Database Sync Bot!
     Available Commands:
     /sync - Start full database sync
     /sync [dbname] - Sync specific database
     /status - Check sync status
     ...

User: /list
Bot: 📊 Configured Databases:
     Database-1 - ✅ Enabled
     Database-2 - ✅ Enabled
     Database-3 - ✅ Enabled
     ...

User: /sync
Bot: 🚀 Starting Full Sync
     Syncing all enabled databases...
     This may take several minutes.
     I'll notify you when complete.

[A few minutes later]

Bot: ✅ Sync Completed Successfully
     Total: 3, Success: 3, Failed: 0

User: /sync Database-1
Bot: 🚀 Starting Sync: Database-1
     This may take several minutes.

[A minute later]

Bot: ✅ Sync Completed: Database-1
     Status: Success
```

## 🔄 Updates

To update the service:

```bash
# Stop service
sudo systemctl stop telegram-bot-sync

# Copy new script
cp telegram_bot_service /home/YOUR_USERNAME/

# Set permissions
chmod +x /home/YOUR_USERNAME/telegram_bot_service

# Start service
sudo systemctl start telegram-bot-sync
```

## 💡 Tips

1. **Scheduled Syncs**: You can set up automatic syncs with cron
2. **Multiple Bots**: You can run multiple bots with different configs
3. **Custom Commands**: Edit the script to add custom commands
4. **Notifications**: Automatic notification when sync completes

## 📞 Support

If you have issues:
1. Check log files
2. Check service status
3. Ensure bot token and chat ID are correct
