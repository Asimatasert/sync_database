# Telegram Bot - Quick Start 🚀

Database synchronization from Telegram in 5 minutes!

## 📋 Prerequisites

✅ Telegram bot token and chat ID (available in config)
✅ jq and curl installed
✅ PostgreSQL and sync_database_runner working

## ⚡ 3 Steps to Setup

### 1️⃣ Prepare Config

Check your `sync_database.json` file:

```json
{
  "global_options": {
    "telegram": {
      "enabled": true,
      "bot_token": "YOUR_TELEGRAM_BOT_TOKEN",
      "chat_id": "YOUR_TELEGRAM_CHAT_ID",
      "authorized_users": []
    }
  }
}
```

**Note:** `authorized_users` empty = everyone authorized, filled = only list

### 2️⃣ Start the Bot

```bash
# Make executable
chmod +x telegram_bot_service

# Start
bash telegram_bot_service
```

**On first run it will ask:**

```
Install as systemd service? (y/n):
```

- **Type "y"** → Automatic installation, auto-start ✅
- **Type "n"** → Manual mode, stops when terminal closes ⚠️

### 3️⃣ Test from Telegram

1. Open your bot on Telegram
2. Type `/start`
3. Type `/sync`
4. Sync started! 🎉

## 🎮 Bot Commands

```
/start         - Start bot and get welcome message
/sync          - Synchronize all databases
/sync [dbname] - Synchronize specific database
/status        - View current sync status
/list          - Show database list
/logs [N]      - Show last N lines of log (default: 20)
/stop          - Stop running sync
/help          - Show commands and examples
```

### 📝 Usage Examples

```
/sync                → Sync all databases
/sync Database-1     → Sync only Database-1
/sync Database-2     → Sync only Database-2
/logs 100            → Last 100 lines of log
/stop                → Stop running operation
```

## 🔧 Management

### If Service is Installed

```bash
# Check status
sudo systemctl status telegram-bot-sync

# Watch logs
sudo journalctl -u telegram-bot-sync -f

# Restart
sudo systemctl restart telegram-bot-sync

# Stop
sudo systemctl stop telegram-bot-sync
```

### If in Manual Mode

```bash
# Start
bash telegram_bot_service

# Stop
Ctrl+C

# Convert to service
bash telegram_bot_service --install-service
```

## 👥 Adding Authorized Users

### 1. Get User ID

Send `/start` to the bot, it will show your User ID:

```
Your Info:
User ID: 123456789
```

### 2. Add to Config

```json
{
  "telegram": {
    "authorized_users": [
      "123456789",
      "987654321"
    ]
  }
}
```

### 3. Restart Service

```bash
sudo systemctl restart telegram-bot-sync
```

## 🐛 Quick Troubleshooting

### Bot not responding?

```bash
# Check service status
sudo systemctl status telegram-bot-sync

# Check logs
sudo journalctl -u telegram-bot-sync -f
```

### "Unauthorized Access" error?

1. Send `/start` to bot
2. Copy your User ID
3. Add to config
4. Restart service

### Sync not starting?

```bash
# Manual test
bash sync_database_runner

# Check permissions
chmod +x sync_database_runner
chmod +x telegram_bot_service
```

## 📞 Help

For detailed information:
- `TELEGRAM_BOT_SETUP.md` - Complete setup guide
- `README.md` - Sync database documentation

## 💡 Tips

1. **Use a group**: Manage together with team members
2. **Log tracking**: Use `/logs 100` for detailed logs
3. **Automatic installation**: Choose "y" on first run
4. **Status check**: Always see status with `/status`

---

**Ready!** You can now manage databases from Telegram! 🎉
