# Chatlog Beginner's Guide

Welcome! This guide is designed for first-time users who want to understand and use Chatlog. No prior technical knowledge required.

## Table of Contents

1. [What is Chatlog?](#what-is-chatlog)
2. [Why Use Chatlog?](#why-use-chatlog)
3. [Basic Concepts](#basic-concepts)
4. [Getting Started](#getting-started)
5. [Understanding the Interface](#understanding-the-interface)
6. [Common Tasks](#common-tasks)
7. [Tips for Success](#tips-for-success)
8. [Frequently Asked Questions](#frequently-asked-questions)
9. [Getting Help](#getting-help)

## What is Chatlog?

Chatlog is a tool that helps you **access and manage your WeChat chat history** on your computer. Think of it as a personal search engine and backup tool for your WeChat messages.

### What Can It Do?

- ✅ **Read your chat history**: View old messages even if WeChat doesn't show them
- ✅ **Search messages**: Find specific conversations or keywords across all chats
- ✅ **Export data**: Save your chats as files (CSV, JSON) for backup or analysis
- ✅ **View media**: Access images, videos, and voice messages
- ✅ **AI Integration**: Use AI assistants to analyze and search your chats
- ✅ **Privacy-focused**: Everything runs on your computer, no data sent to external servers

### What It Can't Do

- ❌ **Cannot access someone else's chats**: Only works with your own data
- ❌ **Cannot send messages**: Read-only access
- ❌ **Cannot recover deleted messages**: Can only read what's in WeChat's database
- ❌ **Cannot work without WeChat**: Requires WeChat to be installed

## Why Use Chatlog?

### Problem: WeChat's Limitations

1. **Limited search**: WeChat's built-in search is basic
2. **Can't bulk export**: Hard to backup or migrate chats
3. **Mobile-first**: Difficult to view old messages on computer
4. **No analytics**: Can't analyze chat patterns or statistics

### Solution: Chatlog

1. **Powerful search**: Find messages by keyword, date, person, or content type
2. **Easy export**: Save chats in standard formats (CSV, JSON)
3. **Full access**: View your complete chat history on computer
4. **Analytics ready**: Export data for analysis or AI processing

## Basic Concepts

Before we start, let's understand some key concepts:

### 1. Encryption Key (Data Key)

**What is it?**  
WeChat encrypts (locks) your chat databases for security. The encryption key is like a password that unlocks them.

**Why do I need it?**  
Chatlog needs this key to read your encrypted chat files.

**How do I get it?**  
Chatlog can extract it from WeChat's memory while WeChat is running.

**Is it safe?**  
Yes, the key is stored only on your computer. Chatlog never sends it anywhere.

### 2. Database Files

**What are they?**  
WeChat stores your chats in SQLite database files (like `.db` files). These are specialized files that organize your data.

**Where are they?**  
- **Windows**: `C:\Users\YourName\Documents\WeChat Files\YourWeChatID\Msg`
- **macOS**: `~/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/*/Message`

**Do I need to know this?**  
Not really! Chatlog finds them automatically.

### 3. Decryption

**What is it?**  
Converting WeChat's encrypted (locked) database files into regular SQLite files that Chatlog can read.

**How long does it take?**  
Usually 1-5 minutes, depending on how much data you have.

**Do I need to do it often?**  
Only when:
- First time using Chatlog
- WeChat updates its databases with new messages (use auto-decrypt)
- You want to update your accessible chat history

### 4. Work Directory

**What is it?**  
A folder where Chatlog stores the decrypted (unlocked) database files.

**Where is it?**  
Default: `~/.chatlog/work/` (can be customized)

**Why separate?**  
Keeps original WeChat files untouched and safe.

### 5. HTTP Service / API

**What is it?**  
A web server that runs on your computer, providing access to your chats through a web browser or programs.

**What's the address?**  
Default: `http://localhost:5030` (only accessible from your computer)

**Do I need to understand HTTP?**  
No! Just think of it as "turning on web access to your chats."

## Getting Started

### Prerequisites

**What You Need:**
1. A computer (Windows or macOS)
2. WeChat installed and logged in
3. 5-10 minutes of time
4. Basic computer skills (can use terminal/command line)

**macOS Users Additional Requirement:**
- Temporarily disable SIP (System Integrity Protection) for key extraction
- Don't worry! We'll guide you through it

### Installation

#### Option 1: Download Pre-built Binary (Easiest)

**Step 1**: Visit the releases page
```
https://github.com/sjzar/chatlog/releases
```

**Step 2**: Download the right file for your system
- Windows: `chatlog_windows_amd64.zip`
- macOS (Intel): `chatlog_darwin_amd64.tar.gz`
- macOS (Apple Silicon): `chatlog_darwin_arm64.tar.gz`

**Step 3**: Extract the file
- Windows: Right-click → "Extract All"
- macOS: Double-click the `.tar.gz` file

**Step 4**: You now have a `chatlog` or `chatlog.exe` file!

#### Option 2: Install with Go (For Developers)

If you have Go installed:
```bash
go install github.com/sjzar/chatlog@latest
```

### First Run

#### Windows

**Step 1**: Open Terminal
- Press `Win + R`
- Type `cmd` and press Enter

**Step 2**: Navigate to chatlog folder
```cmd
cd Downloads\chatlog
```

**Step 3**: Run Chatlog
```cmd
chatlog.exe
```

#### macOS

**Step 1**: Open Terminal
- Press `Cmd + Space`
- Type "Terminal" and press Enter

**Step 2**: Navigate to chatlog folder
```bash
cd ~/Downloads/chatlog
```

**Step 3**: Make it executable (first time only)
```bash
chmod +x chatlog
```

**Step 4**: Run Chatlog
```bash
./chatlog
```

### Initial Setup

You'll see a menu-based interface. Don't worry, it's simple!

```
╔════════════════════════════════════╗
║           Chatlog Menu             ║
╚════════════════════════════════════╝

  → Get Data Key
    Decrypt Data
    Start HTTP Service
    Settings
    Exit

↑/↓: Navigate • Enter: Select • Ctrl+C: Quit
```

**Step 1: Get Data Key**

1. Press `↓` to move to "Get Data Key"
2. Press `Enter`
3. Wait for the process to complete
4. You'll see: `Data Key: [abc123...]`

**Important for macOS users:**
- If you get an error, you need to disable SIP first
- See [macOS SIP Guide](#disabling-sip-on-macos) below

**Step 2: Decrypt Data**

1. Press `↓` to move to "Decrypt Data"
2. Press `Enter`
3. Wait for decryption to complete (1-5 minutes)
4. You'll see progress: `Decrypting MSG0.db... ✓`

**Step 3: Start HTTP Service**

1. Press `↓` to move to "Start HTTP Service"
2. Press `Enter`
3. You'll see: `HTTP server started on 127.0.0.1:5030`

**Step 4: Access Your Data**

Open your web browser and go to:
```
http://localhost:5030
```

You'll see the Chatlog web interface! 🎉

## Understanding the Interface

### Terminal UI (Text-Based Menu)

```
Current Menu Structure:

Main Menu
├── Get Data Key          (Extract encryption key)
├── Decrypt Data          (Decrypt databases)
├── Start HTTP Service    (Enable web access)
├── Start Auto Decrypt    (Real-time sync)
├── Switch Account        (Multiple WeChat accounts)
├── Settings              (Change configuration)
│   ├── Set HTTP Address
│   └── Set Work Directory
└── Exit                  (Close Chatlog)
```

**Navigation:**
- `↑` `↓`: Move between options
- `Enter`: Select option
- `Esc`: Go back
- `Ctrl+C`: Exit program

### Web Interface (Browser)

Visit `http://localhost:5030` to see:

**Tabs:**

1. **Chatlog**: Query messages
   - Select time range
   - Enter contact name
   - Add keywords
   - Choose format (JSON/CSV/Text)

2. **Contact**: View contacts
   - Search by name
   - View details

3. **Chatroom**: Group chats
   - List all groups
   - See members

4. **Session**: Recent chats
   - See last message
   - Unread count

**Example Query:**

To find messages from "Alice" today:
1. Go to "Chatlog" tab
2. Time: Select "Today"
3. Talker: Type "Alice"
4. Click "Query"

## Common Tasks

### Task 1: Search for a Keyword

**Scenario**: Find all messages containing "meeting"

**Using Web UI:**
1. Open `http://localhost:5030`
2. Go to "Chatlog" tab
3. Leave "Time" and "Talker" empty
4. Keyword: Type "meeting"
5. Click "Query"

**Using Command Line:**
```bash
curl "http://localhost:5030/api/v1/chatlog?keyword=meeting"
```

**Result**: All messages containing "meeting" will be displayed

---

### Task 2: Export Chat with a Friend

**Scenario**: Save all chats with "Bob" to a file

**Using Web UI:**
1. Open `http://localhost:5030`
2. Go to "Chatlog" tab
3. Talker: Type "Bob"
4. Format: Select "CSV"
5. Click "Query"
6. Copy the results and save to file

**Using Command Line:**
```bash
curl "http://localhost:5030/api/v1/chatlog?talker=Bob&format=csv" > bob_chat.csv
```

**Result**: All chats saved to `bob_chat.csv` file

---

### Task 3: View Today's Messages

**Scenario**: See what was discussed today

**Using Web UI:**
1. Open `http://localhost:5030`
2. Go to "Chatlog" tab
3. Time: Select "Today"
4. Click "Query"

**Using Command Line:**
```bash
curl "http://localhost:5030/api/v1/chatlog?time=today"
```

---

### Task 4: Find Messages from Last Week

**Scenario**: Review last week's conversations

**Using Web UI:**
1. Time: Select "Last 7 Days"
2. Click "Query"

**Using Command Line:**
```bash
curl "http://localhost:5030/api/v1/chatlog?time=last7days"
```

---

### Task 5: Download Images from a Chat

**Scenario**: Save all images from conversation with "Alice"

**Using Command Line:**
```bash
# Create folder
mkdir alice_images

# Get messages with images (type=3)
curl -s "http://localhost:5030/api/v1/chatlog?talker=Alice" | \
  grep -o '"imgPath":"[^"]*"' | \
  cut -d'"' -f4 | \
  while read path; do
    curl "http://localhost:5030/data/$path" > "alice_images/$(basename $path)"
  done
```

**Note**: This requires some terminal knowledge. Consider using the web UI to view images first.

---

### Task 6: Enable Real-Time Sync

**Scenario**: Keep chat database up-to-date automatically

**Steps:**
1. In Chatlog TUI menu
2. Select "Start Auto Decrypt"
3. Leave Chatlog running in background

**What happens:**
- Chatlog monitors WeChat's database files
- When new messages arrive, automatically decrypts them
- Your queries always return latest data
- Webhook notifications can be configured

---

## Tips for Success

### Performance Tips

1. **Use specific filters**
   - Always specify time range when possible
   - Specify talker/contact name
   - This speeds up queries significantly

2. **Pagination for large results**
   - Don't try to load all messages at once
   - Use limit and offset parameters
   - Example: `?limit=100&offset=0`

3. **Export to CSV for large datasets**
   - CSV is more efficient than JSON
   - Easier to open in Excel or Google Sheets

### Privacy Tips

1. **Keep your key secure**
   - Don't share your data key
   - Config file contains sensitive info
   - Location: `~/.chatlog/chatlog.json`

2. **Local access only**
   - Don't expose HTTP service to internet
   - Default `127.0.0.1` means localhost only
   - Use firewall if exposing to network

3. **Backup your data**
   - Regularly backup decrypted databases
   - Keep encrypted originals safe
   - Export important chats

### Troubleshooting Tips

1. **Key extraction fails**
   - Ensure WeChat is running
   - macOS: Check if SIP is disabled
   - Windows: Run as Administrator
   - Try `chatlog key --force`

2. **Decryption fails**
   - Verify key is correct: `chatlog key`
   - Check WeChat version matches setting
   - Ensure sufficient disk space

3. **Can't access web UI**
   - Check if service is running
   - Verify port 5030 is not in use
   - Try different port: `chatlog server --addr :8080`

4. **No messages found**
   - Ensure decryption completed
   - Check work directory has .db files
   - Verify talker name is correct
   - Try searching without filters first

## Disabling SIP on macOS

**Why?**  
macOS System Integrity Protection (SIP) prevents Chatlog from reading WeChat's memory to extract the key.

**Is it safe?**  
Yes, temporarily disabling SIP for key extraction is safe. You can re-enable it immediately after.

**Steps:**

**Intel Mac:**
1. Restart your Mac
2. Hold `Command + R` during boot
3. Wait for Recovery Mode to load
4. Click "Utilities" → "Terminal"
5. Type: `csrutil disable`
6. Press Enter
7. Restart Mac
8. Run Chatlog and extract key
9. **Optional**: Re-enable SIP (repeat steps 1-4, then type `csrutil enable`)

**Apple Silicon Mac:**
1. Shut down your Mac
2. Hold power button until "Loading startup options" appears
3. Select "Options" → "Continue"
4. Select a user and enter password
5. Click "Utilities" → "Terminal"
6. Type: `csrutil disable`
7. Press Enter
8. Restart Mac
9. Run Chatlog and extract key
10. **Optional**: Re-enable SIP (repeat steps 1-6, then type `csrutil enable`)

**Important Notes:**
- Disabling SIP is only needed for key extraction
- After extracting the key, you can re-enable SIP
- Re-enabling SIP doesn't affect Chatlog's usage
- You only need to extract key once (unless you reinstall WeChat)

## Frequently Asked Questions

### General Questions

**Q: Is Chatlog safe to use?**  
A: Yes. Chatlog runs entirely on your computer and never sends your data anywhere. It's open-source, so you can review the code.

**Q: Will it affect my WeChat?**  
A: No. Chatlog only reads data, never modifies WeChat's files.

**Q: Can I use it without technical knowledge?**  
A: Yes! The Terminal UI is designed to be simple. Follow the on-screen instructions.

**Q: Does it work on phone?**  
A: No, Chatlog is for computer use only (Windows/macOS).

### Usage Questions

**Q: How often should I decrypt data?**  
A: When you want to access new messages. Or use "Auto Decrypt" mode for automatic updates.

**Q: Can I use it while WeChat is running?**  
A: Yes, but close WeChat when decrypting to avoid file conflicts.

**Q: What if I have multiple WeChat accounts?**  
A: Use "Switch Account" in the menu to manage multiple accounts.

**Q: Can I query messages from years ago?**  
A: Yes, as long as they exist in WeChat's database.

### Technical Questions

**Q: Where is my data stored?**  
A: 
- Config: `~/.chatlog/chatlog.json`
- Decrypted databases: `~/.chatlog/work/`
- Original WeChat data: Unchanged in WeChat's directory

**Q: How much disk space needed?**  
A: Approximately same size as your WeChat data (typically 1-10GB).

**Q: Can I move it to another computer?**  
A: Yes, copy the work directory and config file.

**Q: Does it support other languages?**  
A: Yes, Chatlog supports Unicode and multiple languages in messages.

### Error Questions

**Q: "Permission denied" error?**  
A: 
- macOS: Disable SIP for key extraction
- Windows: Run as Administrator
- Check file permissions

**Q: "Database not found"?**  
A: Ensure you've run "Decrypt Data" first.

**Q: "Incorrect key" error?**  
A: Extract key again with `chatlog key --force`.

**Q: Web UI shows blank page?**  
A: 
- Ensure HTTP service is running
- Try accessing `http://localhost:5030/health`
- Check browser console for errors

## Getting Help

### Self-Help Resources

1. **Documentation**
   - [Architecture Guide](ARCHITECTURE.md) - System design
   - [API Reference](API_REFERENCE.md) - API documentation
   - [Workflow Guide](WORKFLOW.md) - Advanced usage
   - [Code Guide](CODE_GUIDE.md) - For developers

2. **Examples**
   - Check `docs/prompt.md` for AI assistant usage
   - See `docs/docker.md` for Docker deployment
   - Read `docs/mcp.md` for MCP integration

### Community Support

1. **GitHub Issues**
   - Report bugs: https://github.com/sjzar/chatlog/issues
   - Check existing issues first

2. **GitHub Discussions**
   - Ask questions: https://github.com/sjzar/chatlog/discussions
   - Share experiences
   - Request features

3. **README**
   - Main documentation: https://github.com/sjzar/chatlog/blob/main/README.md
   - Quick start guide
   - Platform-specific notes

### When Reporting Issues

Please include:
1. Your operating system (Windows/macOS + version)
2. WeChat version
3. Chatlog version
4. What you were trying to do
5. What error message you got
6. What you've tried to fix it

**Example:**
```
OS: macOS 13.5
WeChat: 4.0.2
Chatlog: 1.0.0
Issue: Cannot extract key
Error: "Permission denied"
Tried: Disabled SIP, restarted Mac
```

## Next Steps

Now that you understand the basics:

1. **Try the examples** in [Common Tasks](#common-tasks)
2. **Explore the API** in [API Reference](API_REFERENCE.md)
3. **Set up automation** with [Workflow Guide](WORKFLOW.md)
4. **Integrate AI** using [MCP docs](mcp.md)
5. **Deploy to NAS** with [Docker Guide](docker.md)

## Conclusion

Congratulations! You now know:
- ✅ What Chatlog is and what it can do
- ✅ Basic concepts (keys, encryption, databases)
- ✅ How to install and run Chatlog
- ✅ How to extract keys and decrypt data
- ✅ How to query and export your chats
- ✅ Where to get help

Remember: Start simple, explore gradually. Chatlog is powerful but user-friendly. Happy exploring! 🚀

---

**Still have questions?** Visit our [GitHub Discussions](https://github.com/sjzar/chatlog/discussions) - we're here to help!
