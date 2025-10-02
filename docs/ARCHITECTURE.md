# Chatlog Architecture Documentation

## Table of Contents

1. [Overview](#overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [Technology Stack](#technology-stack)
6. [Platform Support](#platform-support)
7. [Database Schema](#database-schema)
8. [Configuration](#configuration)
9. [Security Considerations](#security-considerations)
10. [Extension Points](#extension-points)
11. [Performance Considerations](#performance-considerations)
12. [Error Handling](#error-handling)
13. [Future Enhancements](#future-enhancements)

## Overview

Chatlog is a comprehensive WeChat chat log management tool written in Go. It provides functionality to decrypt, query, and serve WeChat chat data through multiple interfaces including a Terminal UI, HTTP REST API, and MCP (Model Context Protocol) integration.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Interfaces                         │
├─────────────────┬───────────────────┬───────────────────────────┤
│   Terminal UI   │   HTTP REST API   │   MCP Protocol Server     │
│   (Bubble Tea)  │   (Gin Router)    │   (SSE/Streamable HTTP)   │
└────────┬────────┴─────────┬─────────┴────────────┬──────────────┘
         │                  │                      │
         └──────────────────┼──────────────────────┘
                           │
              ┌────────────▼────────────┐
              │   Manager (Orchestrator) │
              │   - State Management     │
              │   - Service Coordination │
              └─────────────┬────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼─────┐    ┌─────▼──────┐   ┌─────▼──────┐
    │  WeChat  │    │  Database  │   │    HTTP    │
    │ Service  │    │  Service   │   │  Service   │
    └────┬─────┘    └─────┬──────┘   └─────┬──────┘
         │                │                 │
    ┌────▼─────┐    ┌─────▼──────┐   ┌─────▼──────┐
    │ Decrypt  │    │  WechatDB  │   │   Router   │
    │  Key     │    │  SQLite    │   │  Handlers  │
    │ Extract  │    │   Access   │   │   Media    │
    └──────────┘    └────────────┘   └────────────┘
```

## Core Components

### 1. Manager (`internal/chatlog/manager.go`)

The Manager is the central orchestrator that coordinates all services and manages application lifecycle.

**Responsibilities:**
- Initialize and manage all services (WeChat, Database, HTTP)
- Handle account switching and multi-instance management
- Coordinate service startup/shutdown sequences
- Manage configuration and state persistence

**Key Methods:**
- `Run()`: Starts the application in Terminal UI mode
- `CommandHTTPServer()`: Starts in server-only mode
- `CommandDecrypt()`: Runs decrypt-only operation
- `StartService()`: Starts HTTP and Database services
- `DecryptDBFiles()`: Initiates database decryption

### 2. WeChat Service (`internal/chatlog/wechat/service.go`)

Handles all WeChat-specific operations including process detection, key extraction, and database decryption.

**Responsibilities:**
- Detect running WeChat processes
- Extract encryption keys from WeChat memory
- Decrypt encrypted SQLite databases
- Monitor database changes (auto-decrypt mode)
- Support multiple WeChat accounts

**Key Methods:**
- `GetWeChatInstances()`: Lists all running WeChat processes
- `GetDataKey()`: Extracts encryption keys
- `DecryptDBFiles()`: Batch decrypt all database files
- `StartAutoDecrypt()`: Enable real-time database monitoring
- `DecryptFileCallback()`: Handle file change events

### 3. Database Service (`internal/chatlog/database/service.go`)

Manages access to decrypted WeChat SQLite databases and provides data query capabilities.

**Responsibilities:**
- Open and manage decrypted SQLite databases
- Provide high-level query interfaces for messages, contacts, sessions
- Handle database state transitions
- Manage webhook callbacks for new messages

**Key Methods:**
- `Start()`: Initialize database connections
- `GetMessages()`: Query chat messages
- `GetContacts()`: Retrieve contact list
- `GetChatRooms()`: Retrieve group chat list
- `GetSessions()`: Get recent chat sessions
- `GetMedia()`: Fetch media file information

### 4. HTTP Service (`internal/chatlog/http/service.go`)

Provides HTTP REST API and serves the web interface.

**Responsibilities:**
- Expose REST API endpoints for data access
- Serve static web UI files
- Handle media file serving (images, videos, voice)
- Support CORS and error handling
- Provide MCP protocol endpoints

**Key Methods:**
- `Start()`: Start HTTP server
- `initRouter()`: Configure all routes
- `initAPIRouter()`: Setup API endpoints
- `initMCPRouter()`: Setup MCP protocol endpoints
- `handleChatlog()`: Handle chat log queries

### 5. MCP Service (`internal/mcp/mcp.go`)

Implements the Model Context Protocol for AI assistant integration.

**Responsibilities:**
- Provide SSE (Server-Sent Events) endpoints
- Handle MCP JSON-RPC requests
- Manage session state for AI clients
- Expose tools and resources to AI assistants

**Components:**
- Session management for client connections
- Tool definitions for AI interaction
- Resource providers for data access
- Prompt templates for common queries

### 6. Context (`internal/chatlog/ctx/context.go`)

Application-wide context management for state and configuration.

**Responsibilities:**
- Store current account information
- Manage configuration persistence
- Track service states (HTTP enabled, auto-decrypt)
- Handle account switching

## Data Flow

### Decryption Flow

```
1. User requests decrypt
   ↓
2. Manager.DecryptDBFiles()
   ↓
3. WeChat Service scans data directory
   ↓
4. For each encrypted .db file:
   - Read file header and salt
   - Derive encryption keys using PBKDF2
   - Decrypt pages using AES-CBC
   - Verify HMAC integrity
   - Write to work directory
   ↓
5. Decrypted databases ready in work directory
```

### Query Flow

```
1. HTTP Request: GET /api/v1/chatlog?talker=xxx
   ↓
2. HTTP Service validates request
   ↓
3. Database Service.GetMessages()
   ↓
4. WechatDB executes SQLite query
   ↓
5. Model objects created and returned
   ↓
6. HTTP response serialized (JSON/CSV)
   ↓
7. Client receives formatted data
```

### Media Serving Flow

```
1. Request: GET /image/<key>
   ↓
2. HTTP Service.handleMedia()
   ↓
3. Database Service.GetMedia()
   ↓
4. Resolve media file path
   ↓
5. If encrypted: decrypt on-the-fly
   ↓
6. Return 302 redirect or file content
```

### Auto-Decrypt Flow

```
1. File Monitor watches data directory
   ↓
2. Detect .db file changes (Write/Create events)
   ↓
3. Debounce multiple rapid events
   ↓
4. Trigger decryption after stabilization
   ↓
5. Decrypt to work directory
   ↓
6. Database service detects new data
   ↓
7. Webhook callbacks triggered (if configured)
```

## Technology Stack

### Core Technologies
- **Language**: Go 1.21+
- **Database**: SQLite (via mattn/go-sqlite3)
- **HTTP Framework**: Gin
- **Terminal UI**: Bubble Tea / Lip Gloss
- **File Monitoring**: fsnotify

### Key Libraries
- **Cryptography**: golang.org/x/crypto (AES, PBKDF2)
- **MCP Protocol**: mark3labs/mcp-go
- **CLI Framework**: spf13/cobra
- **Logging**: rs/zerolog
- **Media Processing**: go-lame, go-silk (voice conversion)

## Platform Support

### Windows
- **Versions**: WeChat 3.x, 4.x
- **Key Extraction**: Memory scanning via process injection
- **Decryption**: PBKDF2 with SHA1 (v3) or SHA512 (v4)

### macOS
- **Versions**: WeChat 3.x, 4.x (Darwin)
- **Key Extraction**: Requires SIP disabled for memory access
- **Decryption**: Similar to Windows with platform-specific offsets

## Database Schema

WeChat uses multiple SQLite databases:

### Core Databases
- **MSG0.db, MSG1.db, ...**: Message storage (sharded)
- **MicroMsg.db**: Contacts and account information
- **Session.db**: Recent chat sessions
- **MediaMSG0.db, ...**: Media file metadata

### Key Tables
- **MSG**: Messages (content, type, time, sender/talker)
- **Contact**: User contacts and group members
- **Session**: Active chat sessions with last message time
- **Media**: File paths and metadata for images/videos/files

## Configuration

### Config File Location
- **Linux/macOS**: `~/.chatlog/chatlog.json`
- **Windows**: `%USERPROFILE%/.chatlog/chatlog.json`

### Configuration Structure
```json
{
  "history": [],
  "last_account": "wxid_xxx",
  "http_addr": "127.0.0.1:5030",
  "http_enabled": false,
  "auto_decrypt": false,
  "webhook": {
    "host": "localhost:5030",
    "items": [...]
  }
}
```

### Environment Variables (Server Mode)
- `CHATLOG_PLATFORM`: windows, darwin
- `CHATLOG_VERSION`: 3, 4
- `CHATLOG_DATA_KEY`: Encryption key (hex)
- `CHATLOG_IMG_KEY`: Image encryption key (hex)
- `CHATLOG_DATA_DIR`: WeChat data directory path
- `CHATLOG_WORK_DIR`: Decrypted database output directory
- `CHATLOG_AUTO_DECRYPT`: Enable auto-decrypt
- `CHATLOG_HTTP_ADDR`: HTTP server address

## Security Considerations

1. **Key Storage**: Encryption keys stored in config file (should be protected)
2. **Memory Access**: Key extraction requires elevated privileges
3. **Data Privacy**: Decrypted data stored locally in work directory
4. **HTTP Access**: Default binding to localhost only
5. **No External Upload**: All processing happens locally

## Extension Points

### Adding New WeChat Versions
1. Implement version-specific decryptor in `internal/wechat/decrypt/`
2. Add key extraction offsets in `internal/wechat/key/`
3. Register in factory methods

### Adding New API Endpoints
1. Define handler in `internal/chatlog/http/handler.go`
2. Register route in `internal/chatlog/http/route.go`
3. Add corresponding database query if needed

### Adding MCP Tools
1. Define tool in `internal/mcp/tool.go`
2. Implement handler logic
3. Register in MCP server initialization

## Performance Considerations

1. **Decryption**: CPU-intensive due to PBKDF2 iterations (64,000-256,000)
2. **Database Queries**: SQLite indexes critical for large message tables
3. **Media Serving**: On-the-fly decryption can be slow for large files
4. **File Monitoring**: Debouncing prevents excessive re-decryption
5. **Concurrent Access**: SQLite read concurrency supported

## Error Handling

The application uses a custom error package (`internal/errors`) that provides:
- Structured error types
- Error middleware for HTTP handlers
- Context-aware error messages
- Recovery from panics

## Testing

While the repository doesn't include extensive tests, the architecture supports:
- Unit testing of individual services
- Integration testing of service coordination
- Mock implementations for testing without WeChat

## Future Enhancements

Potential areas for expansion:
1. Support for additional messaging platforms
2. Enhanced search capabilities (full-text search)
3. Message export formats (PDF, HTML)
4. Cloud storage integration
5. Mobile app companion
6. Enhanced analytics and visualizations
