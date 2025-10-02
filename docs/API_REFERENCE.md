# Chatlog API Reference

Complete reference documentation for Chatlog's HTTP REST API and MCP protocol endpoints.

## Table of Contents

1. [Base Information](#base-information)
2. [Authentication](#authentication)
3. [REST API Endpoints](#rest-api-endpoints)
4. [Media Endpoints](#media-endpoints)
5. [MCP Protocol](#mcp-protocol)
6. [Error Handling](#error-handling)
7. [Response Formats](#response-formats)
8. [Examples](#examples)

## Base Information

### Default Configuration

- **Base URL**: `http://127.0.0.1:5030`
- **API Version**: v1
- **API Base Path**: `/api/v1`
- **Content-Type**: `application/json`
- **Character Encoding**: UTF-8

### Customization

You can customize the server address:

```bash
# Environment variable (server mode)
export CHATLOG_HTTP_ADDR="0.0.0.0:8080"
chatlog server

# Config file (TUI mode)
# Edit ~/.chatlog/chatlog.json
{
  "http_addr": "127.0.0.1:8080"
}
```

## Authentication

Currently, Chatlog does not implement authentication. The API is designed for local use only. 

**Security Recommendations:**
- Bind to localhost (127.0.0.1) only
- Use firewall rules if exposing to network
- Consider reverse proxy with authentication for remote access

## REST API Endpoints

### Health Check

Check if the server is running.

```http
GET /health
```

**Response:**
```json
{
  "status": "ok"
}
```

**Status Codes:**
- `200 OK`: Server is running

---

### Query Chat Logs

Retrieve chat messages based on various filters.

```http
GET /api/v1/chatlog
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `time` | string | No | Time range filter (see formats below) |
| `talker` | string | No | Chat partner (wxid, name, remark, or group ID) |
| `sender` | string | No | Message sender (for group chats) |
| `keyword` | string | No | Search keyword in message content |
| `limit` | integer | No | Number of records to return (default: 100, max: 1000) |
| `offset` | integer | No | Offset for pagination (default: 0) |
| `format` | string | No | Response format: `json`, `csv`, or `text` (default: `json`) |

**Time Format Examples:**

```
# Specific date
time=2024-01-15

# Date range
time=2024-01-01~2024-01-31

# Relative time
time=today
time=yesterday
time=last7days
time=last30days

# ISO 8601 datetime
time=2024-01-15T10:00:00
time=2024-01-15T10:00:00~2024-01-15T18:00:00
```

**Response (JSON):**

```json
{
  "items": [
    {
      "seq": 1234567890,
      "time": "2024-01-15T14:30:00+08:00",
      "talker": "wxid_abc123",
      "talkerName": "Alice",
      "isChatRoom": false,
      "sender": "wxid_abc123",
      "senderName": "Alice",
      "isSelf": false,
      "type": 1,
      "subType": 0,
      "content": "Hello, how are you?",
      "contents": {}
    },
    {
      "seq": 1234567891,
      "time": "2024-01-15T14:31:00+08:00",
      "talker": "wxid_abc123",
      "talkerName": "Alice",
      "isChatRoom": false,
      "sender": "self",
      "senderName": "Me",
      "isSelf": true,
      "type": 1,
      "subType": 0,
      "content": "I'm good, thanks!",
      "contents": {}
    }
  ],
  "total": 2,
  "limit": 100,
  "offset": 0
}
```

**Message Types:**

| Type | Description |
|------|-------------|
| 1 | Text message |
| 3 | Image |
| 34 | Voice message |
| 43 | Video |
| 47 | Emoji/Sticker |
| 49 | Share link/Mini program |
| 10000 | System notification |

**Response (CSV):**

When `format=csv`:

```csv
Seq,Time,Talker,TalkerName,Sender,SenderName,Type,Content
1234567890,2024-01-15T14:30:00+08:00,wxid_abc123,Alice,wxid_abc123,Alice,1,"Hello, how are you?"
1234567891,2024-01-15T14:31:00+08:00,wxid_abc123,Alice,self,Me,1,"I'm good, thanks!"
```

**Response (Text):**

When `format=text`:

```
[2024-01-15 14:30:00] Alice: Hello, how are you?
[2024-01-15 14:31:00] Me: I'm good, thanks!
```

**Example Requests:**

```bash
# Get all messages from today
curl "http://localhost:5030/api/v1/chatlog?time=today"

# Get messages from a specific person
curl "http://localhost:5030/api/v1/chatlog?talker=Alice"

# Search for keyword in date range
curl "http://localhost:5030/api/v1/chatlog?time=2024-01-01~2024-01-31&keyword=meeting"

# Get messages from a group chat sent by specific person
curl "http://localhost:5030/api/v1/chatlog?talker=GroupName&sender=Alice"

# Pagination
curl "http://localhost:5030/api/v1/chatlog?limit=50&offset=100"

# Export to CSV
curl "http://localhost:5030/api/v1/chatlog?time=today&format=csv" > chat.csv
```

**Status Codes:**
- `200 OK`: Success
- `400 Bad Request`: Invalid parameters
- `500 Internal Server Error`: Server error

---

### Get Contacts

Retrieve the contact list.

```http
GET /api/v1/contact
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | No | Search keyword (name, remark, wxid) |
| `limit` | integer | No | Number of records (default: 100) |
| `offset` | integer | No | Offset for pagination (default: 0) |

**Response:**

```json
{
  "items": [
    {
      "userName": "wxid_abc123",
      "alias": "alice_wx",
      "remark": "Alice",
      "nickName": "Alice Wang",
      "type": 3,
      "verifyFlag": 0,
      "reserved1": 0,
      "reserved2": 0
    },
    {
      "userName": "wxid_def456",
      "alias": "bob_wx",
      "remark": "Bob",
      "nickName": "Bob Chen",
      "type": 3,
      "verifyFlag": 0,
      "reserved1": 0,
      "reserved2": 0
    }
  ],
  "total": 2,
  "limit": 100,
  "offset": 0
}
```

**Contact Types:**

| Type | Description |
|------|-------------|
| 1 | WeChat official account |
| 2 | Group chat |
| 3 | Personal contact |
| 4 | Public account |

**Example Requests:**

```bash
# Get all contacts
curl "http://localhost:5030/api/v1/contact"

# Search contacts
curl "http://localhost:5030/api/v1/contact?key=Alice"

# Pagination
curl "http://localhost:5030/api/v1/contact?limit=50&offset=50"
```

---

### Get Chat Rooms

Retrieve the group chat list.

```http
GET /api/v1/chatroom
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | No | Search keyword (group name, room ID) |
| `limit` | integer | No | Number of records (default: 100) |
| `offset` | integer | No | Offset for pagination (default: 0) |

**Response:**

```json
{
  "items": [
    {
      "chatRoomName": "12345678@chatroom",
      "roomData": {
        "name": "Project Team",
        "members": [
          {"userName": "wxid_abc123", "nickName": "Alice"},
          {"userName": "wxid_def456", "nickName": "Bob"}
        ]
      },
      "memberCount": 5,
      "displayName": "Project Team"
    }
  ],
  "total": 1,
  "limit": 100,
  "offset": 0
}
```

**Example Requests:**

```bash
# Get all group chats
curl "http://localhost:5030/api/v1/chatroom"

# Search group chats
curl "http://localhost:5030/api/v1/chatroom?key=Project"
```

---

### Get Sessions

Retrieve recent chat sessions.

```http
GET /api/v1/session
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | No | Search keyword |
| `limit` | integer | No | Number of records (default: 100) |
| `offset` | integer | No | Offset for pagination (default: 0) |

**Response:**

```json
{
  "items": [
    {
      "userName": "wxid_abc123",
      "nickName": "Alice",
      "nTime": "2024-01-15T14:35:00+08:00",
      "nContent": "See you tomorrow!",
      "nType": 1,
      "unreadCount": 2
    }
  ],
  "total": 1,
  "limit": 100,
  "offset": 0
}
```

**Example Requests:**

```bash
# Get recent sessions
curl "http://localhost:5030/api/v1/session"

# Get sessions with pagination
curl "http://localhost:5030/api/v1/session?limit=20"
```

---

## Media Endpoints

### Get Image

Retrieve an image file.

```http
GET /image/:key
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | Image identifier (from message content) |

**Response:**
- Redirects (302) to the actual image file
- Automatically decrypts encrypted images
- Returns the image data

**Example:**

```bash
curl "http://localhost:5030/image/abc123.dat"
```

---

### Get Video

Retrieve a video file.

```http
GET /video/:key
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | Video identifier (from message content) |

**Response:**
- Redirects (302) to the actual video file
- Returns the video data

---

### Get Voice

Retrieve a voice message.

```http
GET /voice/:key
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | Voice message identifier |

**Response:**
- Converts SILK format to MP3
- Returns MP3 audio data
- Falls back to SILK if conversion fails

**Example:**

```bash
# Save voice message as MP3
curl "http://localhost:5030/voice/voice_abc123.dat" > voice.mp3
```

---

### Get File

Retrieve a file attachment.

```http
GET /file/:key
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | File identifier |

**Response:**
- Redirects to the actual file
- Preserves original file type

---

### Get Media Data

Direct access to media files from the data directory.

```http
GET /data/*path
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | string | Relative path within data directory |

**Response:**
- Serves file directly
- Decrypts encrypted images on-the-fly
- Handles various media types

**Example:**

```bash
curl "http://localhost:5030/data/FileStorage/Image/2024-01/abc123.dat"
```

---

## MCP Protocol

Model Context Protocol enables AI assistant integration.

### MCP Endpoint

Access the MCP server via Streamable HTTP.

```http
GET /mcp
```

**Supported Protocols:**
- Streamable HTTP (primary)
- Server-Sent Events (SSE)

**MCP Tools Available:**

1. **query_messages**: Query chat messages
2. **get_contacts**: Retrieve contact list
3. **get_chatrooms**: Get group chat list
4. **get_sessions**: Fetch recent sessions

**MCP Resources Available:**

1. **contacts**: Contact information
2. **chatrooms**: Group chat details
3. **recent_sessions**: Latest chat sessions

### SSE Connection

```http
GET /mcp/sse
```

Establishes a Server-Sent Events connection for real-time communication.

### Message Endpoint

```http
POST /mcp/message?session_id=<id>
```

Send JSON-RPC requests to the MCP server.

**Request Body:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "query_messages",
    "arguments": {
      "talker": "Alice",
      "limit": 10
    }
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found 10 messages..."
      }
    ]
  }
}
```

### Integration Examples

**Claude Desktop:**

Use [mcp-proxy](https://github.com/sparfenyuk/mcp-proxy) for integration.

**ChatWise:**

Direct Streamable HTTP support:
```
http://127.0.0.1:5030/mcp
```

**Cherry Studio:**

MCP server configuration:
```json
{
  "name": "chatlog",
  "url": "http://127.0.0.1:5030/mcp"
}
```

---

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "error_code",
    "message": "Human-readable error message",
    "details": "Additional error details"
  }
}
```

### Common Error Codes

| Code | Status | Description |
|------|--------|-------------|
| `db_not_ready` | 503 | Database not initialized |
| `invalid_parameter` | 400 | Invalid query parameter |
| `not_found` | 404 | Resource not found |
| `internal_error` | 500 | Internal server error |

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Request successful |
| 302 | Found - Redirect to media file |
| 400 | Bad Request - Invalid parameters |
| 404 | Not Found - Resource not found |
| 500 | Internal Server Error |
| 503 | Service Unavailable - Database not ready |

---

## Response Formats

### JSON Format (Default)

Standard JSON with proper indentation and UTF-8 encoding.

```json
{
  "items": [...],
  "total": 100,
  "limit": 50,
  "offset": 0
}
```

### CSV Format

Comma-separated values with header row.

```csv
Header1,Header2,Header3
Value1,Value2,Value3
```

**Notes:**
- Fields containing commas are quoted
- UTF-8 with BOM for Excel compatibility
- Line endings: CRLF (Windows) or LF (Unix)

### Text Format

Human-readable plain text format.

```
[Time] Sender: Message
[Time] Sender: Message
```

---

## Examples

### Example 1: Export Today's Chat to CSV

```bash
#!/bin/bash

# Get today's date
TODAY=$(date +%Y-%m-%d)

# Export to CSV
curl -s "http://localhost:5030/api/v1/chatlog?time=${TODAY}&format=csv" \
  > "chat_${TODAY}.csv"

echo "Exported to chat_${TODAY}.csv"
```

### Example 2: Search Messages with Python

```python
import requests
import json

BASE_URL = "http://localhost:5030/api/v1"

def search_messages(keyword, limit=100):
    """Search messages containing keyword"""
    params = {
        'keyword': keyword,
        'limit': limit,
        'format': 'json'
    }
    
    response = requests.get(f"{BASE_URL}/chatlog", params=params)
    response.raise_for_status()
    
    data = response.json()
    return data['items']

# Usage
messages = search_messages('meeting')
for msg in messages:
    print(f"[{msg['time']}] {msg['senderName']}: {msg['content']}")
```

### Example 3: Get Contact List with JavaScript

```javascript
async function getContacts() {
    const response = await fetch('http://localhost:5030/api/v1/contact');
    const data = await response.json();
    
    data.items.forEach(contact => {
        console.log(`${contact.remark || contact.nickName} (${contact.userName})`);
    });
}

getContacts();
```

### Example 4: Download Voice Messages

```bash
#!/bin/bash

# Get messages with voice
curl -s "http://localhost:5030/api/v1/chatlog?talker=Alice&limit=1000" | \
  jq -r '.items[] | select(.type == 34) | .content' | \
  while read key; do
    echo "Downloading $key..."
    curl -s "http://localhost:5030/voice/$key" > "voice_${key}.mp3"
  done
```

### Example 5: Monitor New Messages via Webhook

```bash
# Start a simple webhook receiver
python3 -m http.server 8080 &

# Configure webhook in ~/.chatlog/chatlog.json
cat > ~/.chatlog/chatlog.json << EOF
{
  "webhook": {
    "host": "localhost:5030",
    "items": [
      {
        "url": "http://localhost:8080/webhook",
        "talker": "Alice"
      }
    ]
  }
}
EOF

# Enable auto-decrypt and wait for messages
chatlog server --auto-decrypt
```

### Example 6: Query with Time Ranges

```bash
# Last 7 days
curl "http://localhost:5030/api/v1/chatlog?time=last7days"

# Specific date range
curl "http://localhost:5030/api/v1/chatlog?time=2024-01-01~2024-01-31"

# Today only
curl "http://localhost:5030/api/v1/chatlog?time=today"

# Yesterday
curl "http://localhost:5030/api/v1/chatlog?time=yesterday"
```

### Example 7: Batch Export by Contact

```bash
#!/bin/bash

# Get all contacts
CONTACTS=$(curl -s "http://localhost:5030/api/v1/contact" | jq -r '.items[].userName')

# Export messages for each contact
for CONTACT in $CONTACTS; do
    NAME=$(curl -s "http://localhost:5030/api/v1/contact?key=$CONTACT" | jq -r '.items[0].remark')
    echo "Exporting messages with $NAME..."
    curl -s "http://localhost:5030/api/v1/chatlog?talker=$CONTACT&format=csv" \
      > "export_${NAME}.csv"
done
```

---

## Rate Limiting

Currently, there are no rate limits implemented. For production use, consider:
- Implementing rate limiting at reverse proxy level
- Using connection pooling for batch operations
- Throttling bulk export operations

## CORS Support

CORS is enabled by default for all origins to support web-based clients. 

**Headers:**
- `Access-Control-Allow-Origin: *`
- `Access-Control-Allow-Methods: GET, POST, OPTIONS`
- `Access-Control-Allow-Headers: *`

## Best Practices

1. **Use Pagination**: Always use `limit` and `offset` for large result sets
2. **Filter Early**: Use time ranges and talker filters to reduce data transfer
3. **Choose Appropriate Format**: Use CSV for exports, JSON for programmatic access
4. **Cache Results**: Cache frequently accessed data on client side
5. **Handle Errors**: Always check HTTP status codes and error responses
6. **Respect Privacy**: Only access data you have permission to view

## Support

For API issues or questions:
- GitHub Issues: https://github.com/sjzar/chatlog/issues
- Discussions: https://github.com/sjzar/chatlog/discussions

## Changelog

### Version 1.0
- Initial API release
- REST endpoints for messages, contacts, chatrooms, sessions
- Media serving with decryption
- MCP protocol support
- Multiple response formats (JSON, CSV, text)
