# Chatlog Workflow Guide

This guide explains the typical workflows for using Chatlog, from initial setup to advanced usage scenarios.

## Table of Contents

1. [Quick Start Workflow](#quick-start-workflow)
2. [First-Time Setup](#first-time-setup)
3. [Daily Usage Workflows](#daily-usage-workflows)
4. [Advanced Workflows](#advanced-workflows)
5. [Docker Deployment Workflow](#docker-deployment-workflow)
6. [Troubleshooting Workflow](#troubleshooting-workflow)

## Quick Start Workflow

### The Simplest Path: Terminal UI

```
┌─────────────────────────────────────────┐
│  1. Download and Install Chatlog       │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  2. Run: chatlog                        │
│     (Launch Terminal UI)                │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  3. Select "Get Data Key"               │
│     (Extract encryption key)            │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  4. Select "Decrypt Data"               │
│     (Decrypt WeChat databases)          │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  5. Select "Start HTTP Service"         │
│     (Enable API access)                 │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  6. Access Data                         │
│     - Web UI: http://localhost:5030     │
│     - API: http://localhost:5030/api/v1 │
│     - MCP: http://localhost:5030/mcp    │
└─────────────────────────────────────────┘
```

**Time Required**: 5-10 minutes for first-time setup

## First-Time Setup

### Scenario 1: Local Development/Personal Use (macOS)

**Prerequisites:**
- macOS with WeChat installed
- Xcode Command Line Tools
- SIP temporarily disabled (for key extraction only)

**Step-by-Step:**

```bash
# 1. Install Xcode Command Line Tools
xcode-select --install

# 2. Download Chatlog
# Visit: https://github.com/sjzar/chatlog/releases
# Or install via Go:
go install github.com/sjzar/chatlog@latest

# 3. Disable SIP (for key extraction)
# Restart Mac in Recovery Mode:
#   - Intel Mac: Hold Command+R during boot
#   - Apple Silicon: Hold power button until "Loading startup options" appears
# In Recovery Mode terminal:
csrutil disable
# Restart Mac

# 4. Run WeChat
open -a WeChat

# 5. Run Chatlog
chatlog

# 6. In the TUI menu:
#    - Select "Get Data Key" (↓ arrow, Enter)
#    - Wait for key extraction
#    - Select "Decrypt Data"
#    - Wait for decryption to complete
#    - Select "Start HTTP Service"

# 7. Access Web UI
open http://localhost:5030

# 8. (Optional) Re-enable SIP
# Restart in Recovery Mode again:
csrutil enable
# Restart Mac
# Note: Re-enabling SIP doesn't affect usage, only key re-extraction
```

**Expected Output:**
```
Data Key: [c0163e***ac3dc6]
Image Key: [38636***653361]

Decrypting databases...
✓ MicroMsg.db
✓ MSG0.db
✓ MSG1.db
✓ Session.db
✓ MediaMSG0.db
...
Decryption complete!

HTTP server started on 127.0.0.1:5030
```

---

### Scenario 2: Windows Personal Use

**Prerequisites:**
- Windows 10/11
- WeChat for Windows installed
- Windows Terminal (recommended)

**Step-by-Step:**

```powershell
# 1. Download Chatlog
# Visit: https://github.com/sjzar/chatlog/releases
# Download: chatlog_windows_amd64.zip

# 2. Extract and run
.\chatlog.exe

# 3. In the TUI:
#    - Navigate with arrow keys
#    - Select "Get Data Key" (press Enter)
#    - Select "Decrypt Data"
#    - Select "Start HTTP Service"

# 4. Access Web UI
start http://localhost:5030
```

**Windows-Specific Notes:**
- Use Windows Terminal for best TUI experience
- May need to run as Administrator for key extraction
- Default data location: `%APPDATA%\Tencent\WeChat`

---

### Scenario 3: Server/NAS Deployment (Docker)

**Prerequisites:**
- Docker installed
- Pre-extracted WeChat data key
- WeChat data directory accessible

**Step-by-Step:**

```bash
# 1. Get encryption key from your main computer
# On your Windows/Mac machine:
chatlog key
# Output:
# Data Key: [abc123...]
# Image Key: [def456...]

# 2. Copy WeChat data to server
# Example using rsync:
rsync -av /Users/yourname/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat/*/Message \
  yourserver:/data/wechat/

# 3. Create docker-compose.yml on server
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  chatlog:
    image: sjzar/chatlog:latest
    restart: unless-stopped
    ports:
      - "5030:5030"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - CHATLOG_PLATFORM=darwin
      - CHATLOG_VERSION=4
      - CHATLOG_DATA_KEY=your-data-key-here
      - CHATLOG_IMG_KEY=your-img-key-here
      - CHATLOG_AUTO_DECRYPT=true
    volumes:
      - /data/wechat:/app/data
      - chatlog-work:/app/work

volumes:
  chatlog-work:
EOF

# 4. Start container
docker-compose up -d

# 5. Check logs
docker-compose logs -f

# 6. Access service
curl http://localhost:5030/health
```

**Expected Output:**
```json
{"status":"ok"}
```

---

## Daily Usage Workflows

### Workflow 1: Query Recent Messages

**Scenario**: Find messages from today

```bash
# Via API
curl "http://localhost:5030/api/v1/chatlog?time=today" | jq

# Via TUI
# (Open http://localhost:5030 in browser)
# 1. Navigate to Chatlog tab
# 2. Select time: "Today"
# 3. Click "Query"
```

**Python Script:**
```python
#!/usr/bin/env python3
import requests
from datetime import date

def get_today_messages():
    url = "http://localhost:5030/api/v1/chatlog"
    params = {'time': 'today', 'limit': 1000}
    
    response = requests.get(url, params=params)
    data = response.json()
    
    print(f"Found {data['total']} messages today:")
    for msg in data['items']:
        print(f"[{msg['time']}] {msg['senderName']}: {msg['content']}")

if __name__ == '__main__':
    get_today_messages()
```

---

### Workflow 2: Export Chat History

**Scenario**: Export all messages with a specific contact

```bash
#!/bin/bash

CONTACT="Alice"
OUTPUT_FILE="chat_with_${CONTACT}.csv"

echo "Exporting chat history with $CONTACT..."

curl -s "http://localhost:5030/api/v1/chatlog?talker=${CONTACT}&format=csv" \
  > "$OUTPUT_FILE"

echo "Exported to $OUTPUT_FILE"
echo "Total lines: $(wc -l < "$OUTPUT_FILE")"
```

**Advanced Export with Date Range:**

```python
#!/usr/bin/env python3
import requests
import csv
from datetime import datetime, timedelta

def export_chat_range(contact, days_back=30):
    """Export chat history for the last N days"""
    
    end_date = datetime.now()
    start_date = end_date - timedelta(days=days_back)
    
    url = "http://localhost:5030/api/v1/chatlog"
    params = {
        'talker': contact,
        'time': f"{start_date.strftime('%Y-%m-%d')}~{end_date.strftime('%Y-%m-%d')}",
        'limit': 10000,
        'format': 'json'
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    filename = f"chat_{contact}_{start_date.date()}_to_{end_date.date()}.csv"
    
    with open(filename, 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(['Time', 'Sender', 'Content', 'Type'])
        
        for msg in data['items']:
            writer.writerow([
                msg['time'],
                msg['senderName'],
                msg['content'],
                msg['type']
            ])
    
    print(f"Exported {len(data['items'])} messages to {filename}")

if __name__ == '__main__':
    export_chat_range('Alice', days_back=30)
```

---

### Workflow 3: Search Across All Chats

**Scenario**: Find all messages containing a keyword

```bash
# Search for "meeting"
curl -s "http://localhost:5030/api/v1/chatlog?keyword=meeting&limit=100" | \
  jq -r '.items[] | "\(.time) - \(.talkerName): \(.content)"'
```

**Output:**
```
2024-01-15T10:30:00+08:00 - Alice: Don't forget our meeting tomorrow
2024-01-14T14:20:00+08:00 - Bob: Meeting at 3pm?
2024-01-13T09:15:00+08:00 - Charlie: Meeting notes attached
```

---

### Workflow 4: Download Media Files

**Scenario**: Download all images from a conversation

```bash
#!/bin/bash

CONTACT="Alice"
OUTPUT_DIR="images_from_${CONTACT}"

mkdir -p "$OUTPUT_DIR"

# Get all messages with images (type=3)
curl -s "http://localhost:5030/api/v1/chatlog?talker=${CONTACT}" | \
  jq -r '.items[] | select(.type == 3) | .contents.imgPath' | \
  while read img_path; do
    if [ ! -z "$img_path" ]; then
      filename=$(basename "$img_path")
      echo "Downloading $filename..."
      curl -s "http://localhost:5030/data/${img_path}" \
        > "${OUTPUT_DIR}/${filename}"
    fi
  done

echo "Download complete. Files saved to $OUTPUT_DIR"
```

---

## Advanced Workflows

### Workflow 5: Real-Time Message Monitoring with Webhook

**Scenario**: Get notified of new messages from specific contacts

**Step 1: Create Webhook Receiver**

```python
#!/usr/bin/env python3
from flask import Flask, request, jsonify
import json

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    
    print(f"\n=== New Message Received ===")
    print(f"Talker: {data.get('talker')}")
    print(f"Messages: {data.get('length')}")
    
    for msg in data.get('messages', []):
        print(f"[{msg['time']}] {msg['senderName']}: {msg['content']}")
    
    return jsonify({'status': 'ok'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

**Step 2: Configure Chatlog**

Edit `~/.chatlog/chatlog.json`:

```json
{
  "webhook": {
    "host": "localhost:5030",
    "items": [
      {
        "url": "http://localhost:8080/webhook",
        "talker": "Alice",
        "keyword": ""
      }
    ]
  }
}
```

**Step 3: Enable Auto-Decrypt**

```bash
# In TUI mode
# Select "Start Auto Decrypt"

# Or in server mode
chatlog server --auto-decrypt
```

**Expected Flow:**
```
1. New message arrives in WeChat
2. Database file updated
3. Chatlog detects change (after ~13 seconds)
4. Database decrypted
5. Webhook triggered
6. Your receiver gets notification
```

---

### Workflow 6: AI Assistant Integration (MCP)

**Scenario**: Use Claude Desktop to query chat history

**Step 1: Start Chatlog with HTTP Service**

```bash
chatlog
# Select "Start HTTP Service"
```

**Step 2: Install and Configure mcp-proxy**

```bash
# Install mcp-proxy
npm install -g mcp-proxy

# Create proxy config
mkdir -p ~/.claude
cat > ~/.claude/mcp-proxy-config.json << 'EOF'
{
  "chatlog": {
    "url": "http://127.0.0.1:5030/mcp"
  }
}
EOF
```

**Step 3: Configure Claude Desktop**

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "chatlog": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-proxy",
        "--config",
        "/Users/yourname/.claude/mcp-proxy-config.json",
        "chatlog"
      ]
    }
  }
}
```

**Step 4: Restart Claude Desktop and Test**

Prompt examples:
- "Show me my recent messages"
- "Find all messages about 'project deadline'"
- "Who did I talk to most last week?"
- "Export my chat with Alice to CSV"

---

### Workflow 7: Multi-Account Management

**Scenario**: Switch between multiple WeChat accounts

```bash
# 1. Run chatlog in TUI mode
chatlog

# 2. Select "Switch Account"
#    - Use ↑/↓ to navigate
#    - Press Enter to select account
#    - Or select "Add History Account"

# 3. For history account:
#    - Enter account identifier
#    - Select data directory
#    - Select work directory
#    - Enter data key
#    - Enter image key

# 4. Account automatically switches
#    - HTTP service restarts
#    - New database loaded
```

**Config Structure for Multiple Accounts:**

`~/.chatlog/chatlog.json`:
```json
{
  "history": [
    {
      "account": "account1",
      "data_dir": "/path/to/account1/data",
      "work_dir": "/path/to/account1/work",
      "data_key": "key1...",
      "img_key": "imgkey1..."
    },
    {
      "account": "account2",
      "data_dir": "/path/to/account2/data",
      "work_dir": "/path/to/account2/work",
      "data_key": "key2...",
      "img_key": "imgkey2..."
    }
  ],
  "last_account": "account1"
}
```

---

### Workflow 8: Automated Backup

**Scenario**: Daily automated backup of chat databases

```bash
#!/bin/bash
# backup-chats.sh

BACKUP_DIR="/backup/chatlog"
DATE=$(date +%Y%m%d)
WORK_DIR="$HOME/.chatlog/work"

# Create backup directory
mkdir -p "$BACKUP_DIR/$DATE"

# Copy decrypted databases
cp -r "$WORK_DIR"/*.db "$BACKUP_DIR/$DATE/"

# Compress
cd "$BACKUP_DIR"
tar -czf "chatlog_${DATE}.tar.gz" "$DATE"
rm -rf "$DATE"

# Keep only last 30 days
find "$BACKUP_DIR" -name "chatlog_*.tar.gz" -mtime +30 -delete

echo "Backup complete: chatlog_${DATE}.tar.gz"
```

**Cron job (run daily at 2 AM):**

```cron
0 2 * * * /path/to/backup-chats.sh
```

---

## Docker Deployment Workflow

### Scenario: NAS Deployment with Auto-Sync

**Architecture:**

```
WeChat (PC) → Syncthing → NAS → Docker Chatlog → Web Access
```

**Step 1: Setup Syncthing on Both Machines**

On PC:
```bash
# Install Syncthing
# Configure folder to sync WeChat data directory
# Share folder with NAS
```

On NAS:
```bash
# Install Syncthing
# Accept shared folder
# Set sync directory: /volume1/wechat_data
```

**Step 2: Deploy Chatlog on NAS**

```yaml
# docker-compose.yml
version: '3.8'

services:
  chatlog:
    image: sjzar/chatlog:latest
    container_name: chatlog
    restart: unless-stopped
    ports:
      - "5030:5030"
    environment:
      - TZ=Asia/Shanghai
      - CHATLOG_PLATFORM=windows
      - CHATLOG_VERSION=4
      - CHATLOG_DATA_KEY=${DATA_KEY}
      - CHATLOG_IMG_KEY=${IMG_KEY}
      - CHATLOG_AUTO_DECRYPT=true
      - CHATLOG_DATA_DIR=/app/data
      - CHATLOG_WORK_DIR=/app/work
    volumes:
      - /volume1/wechat_data:/app/data:ro
      - chatlog_work:/app/work

volumes:
  chatlog_work:
```

**Step 3: Create .env File**

```bash
# .env
DATA_KEY=your_actual_data_key_here
IMG_KEY=your_actual_img_key_here
```

**Step 4: Deploy**

```bash
docker-compose up -d
```

**Step 5: Setup Reverse Proxy (Optional)**

Using Nginx:

```nginx
# /etc/nginx/sites-available/chatlog
server {
    listen 80;
    server_name chatlog.yournas.local;

    location / {
        proxy_pass http://localhost:5030;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Troubleshooting Workflow

### Issue 1: Cannot Extract Key

**Symptoms:**
- "Failed to extract key" error
- "Permission denied" error

**macOS Solution:**

```bash
# Check SIP status
csrutil status

# If enabled, disable it:
# 1. Restart in Recovery Mode
# 2. Open Terminal
csrutil disable
# 3. Restart

# Try extracting key again
chatlog key

# After success, can re-enable SIP
# (Re-enabling doesn't affect future use, only re-extraction)
```

**Windows Solution:**

```powershell
# Run as Administrator
Right-click chatlog.exe → "Run as administrator"

# Check if WeChat is running
Get-Process | Where-Object {$_.ProcessName -like "*WeChat*"}

# If multiple WeChat processes, specify PID
chatlog key --pid <process_id>
```

---

### Issue 2: Decryption Fails

**Symptoms:**
- "Incorrect key" error
- "Hash verification failed"

**Solution Workflow:**

```bash
# 1. Verify key is correct
chatlog key --force

# 2. Check WeChat version
# Ensure CHATLOG_VERSION matches your WeChat version

# 3. Try different platform setting
# If auto-detection fails, manually specify:
chatlog decrypt --platform windows --version 4

# 4. Check database file permissions
ls -la /path/to/wechat/data

# 5. Verify database is actually encrypted
hexdump -C /path/to/MSG0.db | head
# Should NOT start with "SQLite format 3"
```

---

### Issue 3: HTTP Service Won't Start

**Symptoms:**
- "Address already in use"
- Cannot access http://localhost:5030

**Solution Workflow:**

```bash
# 1. Check if port is in use
lsof -i :5030
# or
netstat -an | grep 5030

# 2. Change port
chatlog server --addr :8080

# Or edit config:
{
  "http_addr": "127.0.0.1:8080"
}

# 3. Check firewall
# macOS:
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# Linux:
sudo ufw status

# 4. Test connectivity
curl http://localhost:5030/health
```

---

### Issue 4: Database Not Found

**Symptoms:**
- "No database files found"
- "Database not ready"

**Solution Workflow:**

```bash
# 1. Verify work directory exists and has data
ls -la ~/.chatlog/work/

# 2. Check if decryption completed
ls -la ~/.chatlog/work/**/*.db

# 3. Re-decrypt if needed
chatlog decrypt

# 4. Verify database structure
sqlite3 ~/.chatlog/work/MSG0.db ".schema"

# 5. Check logs
chatlog --debug
```

---

### Issue 5: Images Won't Display

**Symptoms:**
- 404 errors for images
- Encrypted images show as corrupted

**Solution Workflow:**

```bash
# 1. Verify image key
chatlog key
# Ensure Image Key is displayed

# 2. For WeChat 4.x, check xor key
chatlog key --show-xor-key

# 3. Manually test image decryption
curl http://localhost:5030/image/<image_path>

# 4. Check data directory path
# Ensure CHATLOG_DATA_DIR points to correct location

# 5. Verify file exists
ls -la /path/to/wechat/data/FileStorage/Image/
```

---

## Performance Optimization Workflows

### Workflow 9: Optimize Large Database Queries

**Scenario**: Speed up queries on databases with millions of messages

```bash
# 1. Create indexes (if not exist)
sqlite3 ~/.chatlog/work/MSG0.db << 'EOF'
CREATE INDEX IF NOT EXISTS idx_createtime ON MSG(CreateTime);
CREATE INDEX IF NOT EXISTS idx_talker ON MSG(StrTalker);
CREATE INDEX IF NOT EXISTS idx_talker_time ON MSG(StrTalker, CreateTime);
EOF

# 2. Analyze database
sqlite3 ~/.chatlog/work/MSG0.db "ANALYZE;"

# 3. Use pagination for large results
curl "http://localhost:5030/api/v1/chatlog?limit=100&offset=0"
curl "http://localhost:5030/api/v1/chatlog?limit=100&offset=100"

# 4. Always use time filters
curl "http://localhost:5030/api/v1/chatlog?time=last7days"
```

---

### Workflow 10: Reduce Memory Usage

**Scenario**: Running on resource-constrained devices

```bash
# 1. Limit result set size
# Default limit=100, but can reduce:
curl "http://localhost:5030/api/v1/chatlog?limit=50"

# 2. Use CSV format instead of JSON
# (Lower memory overhead)
curl "http://localhost:5030/api/v1/chatlog?format=csv"

# 3. Query specific databases only
# Don't load all message databases

# 4. Docker: Set memory limits
docker run --memory="512m" --memory-swap="1g" ...
```

---

## Best Practices

### Security Best Practices

1. **Keep keys secure**
   ```bash
   chmod 600 ~/.chatlog/chatlog.json
   ```

2. **Bind to localhost only**
   ```bash
   # Don't expose to public internet
   CHATLOG_HTTP_ADDR=127.0.0.1:5030
   ```

3. **Use reverse proxy with auth**
   ```nginx
   location /chatlog {
       auth_basic "Restricted";
       auth_basic_user_file /etc/nginx/.htpasswd;
       proxy_pass http://localhost:5030;
   }
   ```

### Data Management Best Practices

1. **Regular backups**
   - Backup decrypted databases
   - Keep encrypted databases secure
   - Version your backups

2. **Disk space monitoring**
   ```bash
   # WeChat data can be large (10GB+)
   df -h ~/.chatlog/work
   ```

3. **Periodic cleanup**
   ```bash
   # Remove old decrypted databases
   find ~/.chatlog/work -name "*.db" -mtime +90 -delete
   ```

### Performance Best Practices

1. **Use SSD for work directory**
2. **Enable auto-decrypt for real-time access**
3. **Use indexes for large databases**
4. **Implement caching for frequently accessed data**

---

## Conclusion

This workflow guide covers common usage patterns from basic setup to advanced scenarios. The modular architecture of Chatlog allows you to choose the workflow that best fits your needs, whether it's simple local usage, Docker deployment, or integration with AI assistants.

For more specific use cases or questions, please refer to:
- [Architecture Documentation](ARCHITECTURE.md)
- [API Reference](API_REFERENCE.md)
- [Code Guide](CODE_GUIDE.md)
- [GitHub Discussions](https://github.com/sjzar/chatlog/discussions)
