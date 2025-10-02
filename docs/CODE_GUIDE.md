# Chatlog Code Guide

This document provides a detailed walkthrough of the codebase to help developers understand how Chatlog works internally.

## Table of Contents

1. [Project Structure](#project-structure)
2. [Entry Points](#entry-points)
3. [Core Services](#core-services)
4. [Decryption Process](#decryption-process)
5. [Database Access](#database-access)
6. [HTTP API](#http-api)
7. [MCP Integration](#mcp-integration)
8. [Terminal UI](#terminal-ui)
9. [Key Code Paths](#key-code-paths)

## Project Structure

```
chatlog/
├── cmd/chatlog/           # Command-line interface
│   ├── root.go           # Root command and TUI launcher
│   ├── cmd_key.go        # Key extraction command
│   ├── cmd_decrypt.go    # Decryption command
│   └── cmd_server.go     # HTTP server command
├── internal/
│   ├── chatlog/          # Main application logic
│   │   ├── manager.go    # Central orchestrator
│   │   ├── app.go        # Terminal UI application
│   │   ├── conf/         # Configuration management
│   │   ├── ctx/          # Application context
│   │   ├── database/     # Database service
│   │   ├── http/         # HTTP service
│   │   ├── webhook/      # Webhook callbacks
│   │   └── wechat/       # WeChat operations service
│   ├── errors/           # Error types and handling
│   ├── mcp/              # MCP protocol implementation
│   ├── model/            # Data models
│   ├── ui/               # Terminal UI components
│   ├── wechat/           # Low-level WeChat operations
│   │   ├── decrypt/      # Database decryption
│   │   ├── key/          # Key extraction
│   │   ├── model/        # WeChat models
│   │   └── process/      # Process detection
│   └── wechatdb/         # SQLite database access
├── pkg/                  # Reusable packages
│   ├── config/          # Config file management
│   ├── filemonitor/     # File system monitoring
│   ├── util/            # Utility functions
│   └── version/         # Version information
└── main.go              # Application entry point
```

## Entry Points

### 1. Main Entry (`main.go`)

The simplest possible entry point:

```go
package main

import (
    "log"
    "github.com/sjzar/chatlog/cmd/chatlog"
)

func main() {
    log.SetFlags(log.LstdFlags | log.Lshortfile)
    chatlog.Execute()
}
```

### 2. Root Command (`cmd/chatlog/root.go`)

The default command launches the Terminal UI:

```go
func Root(cmd *cobra.Command, args []string) {
    m := chatlog.New()           // Create manager
    if err := m.Run(""); err != nil {  // Run with TUI
        log.Err(err).Msg("failed to run chatlog instance")
    }
}
```

### 3. Manager Run (`internal/chatlog/manager.go`)

```go
func (m *Manager) Run(configPath string) error {
    // 1. Initialize context (load config)
    m.ctx, err = ctx.New(configPath)
    
    // 2. Create services
    m.wechat = wechat.NewService(m.ctx)
    m.db = database.NewService(m.ctx)
    m.http = http.NewService(m.ctx, m.db)
    
    // 3. Detect WeChat instances
    m.ctx.WeChatInstances = m.wechat.GetWeChatInstances()
    
    // 4. Start HTTP service if enabled
    if m.ctx.HTTPEnabled {
        m.StartService()
    }
    
    // 5. Launch Terminal UI (blocks)
    m.app = NewApp(m.ctx, m)
    m.app.Run()
    
    return nil
}
```

## Core Services

### WeChat Service (`internal/chatlog/wechat/service.go`)

The WeChat service handles all interactions with WeChat data.

#### Getting WeChat Instances

```go
func (s *Service) GetWeChatInstances() []*wechat.Account {
    wechat.Load()  // Scan system for WeChat processes
    return wechat.GetAccounts()
}
```

Internally, `wechat.Load()` (`internal/wechat/wechat.go`):
1. Detects running WeChat processes
2. Reads version from executable
3. Finds data directory
4. Creates Account objects

#### Extracting Keys

```go
func (a *Account) GetKey(ctx context.Context) (string, string, error) {
    // 1. Create key extractor for platform/version
    extractor, err := key.NewExtractor(a.Platform, a.Version)
    
    // 2. Create validator to verify key correctness
    validator, err := decrypt.NewValidator(process.Platform, process.Version, process.DataDir)
    extractor.SetValidate(validator)
    
    // 3. Extract keys from memory
    dataKey, imgKey, err := extractor.Extract(ctx, process)
    
    return dataKey, imgKey, nil
}
```

The extraction process (`internal/wechat/key/`):
- **Windows**: Scans process memory for key patterns
- **macOS**: Uses ptrace to read memory (requires SIP disabled)
- Validates key by attempting to decrypt first database page

#### Decrypting Databases

```go
func (s *Service) DecryptDBFile(dbFile string) error {
    // 1. Get appropriate decryptor
    decryptor, err := decrypt.NewDecryptor(
        s.conf.GetPlatform(), 
        s.conf.GetVersion()
    )
    
    // 2. Prepare output path
    output := filepath.Join(s.conf.GetWorkDir(), ...)
    
    // 3. Decrypt file
    err = decryptor.Decrypt(ctx, dbFile, key, outputFile)
    
    return nil
}
```

### Database Service (`internal/chatlog/database/service.go`)

The database service provides high-level access to WeChat data.

#### Starting the Service

```go
func (s *Service) Start() error {
    // 1. Open all WeChat databases
    db, err := wechatdb.New(
        s.conf.GetWorkDir(),
        s.conf.GetPlatform(),
        s.conf.GetVersion()
    )
    
    // 2. Update state
    s.SetReady()
    s.db = db
    
    // 3. Initialize webhooks
    s.initWebhook()
    
    return nil
}
```

The `wechatdb.New()` function (`internal/wechatdb/db.go`):
1. Scans work directory for .db files
2. Opens SQLite connections for each database
3. Categorizes databases (message, contact, session, media)
4. Creates query interfaces

#### Querying Messages

```go
func (s *Service) GetMessages(start, end time.Time, talker string, sender string, keyword string, limit, offset int) ([]*model.Message, error) {
    return s.db.GetMessages(start, end, talker, sender, keyword, limit, offset)
}
```

The query process (`internal/wechatdb/message.go`):
1. Resolve talker name to WeChat ID
2. Determine which message database(s) to query
3. Build SQL query with filters
4. Execute query across databases
5. Parse results into model objects
6. Enrich with contact information

### HTTP Service (`internal/chatlog/http/service.go`)

The HTTP service exposes APIs and serves the web interface.

#### Service Initialization

```go
func NewService(conf Config, db *database.Service) *Service {
    gin.SetMode(gin.ReleaseMode)
    router := gin.New()
    
    // Middleware
    router.Use(
        errors.RecoveryMiddleware(),
        errors.ErrorHandlerMiddleware(),
        gin.LoggerWithWriter(log.Logger, "/health"),
        corsMiddleware(),
    )
    
    s := &Service{
        conf:   conf,
        db:     db,
        router: router,
    }
    
    s.initMCPServer()
    s.initRouter()
    return s
}
```

#### Route Setup (`internal/chatlog/http/route.go`)

```go
func (s *Service) initRouter() {
    s.initBaseRouter()    // Static files, health check
    s.initMediaRouter()   // Image, video, voice, file
    s.initAPIRouter()     // REST API endpoints
    s.initMCPRouter()     // MCP protocol endpoints
}

func (s *Service) initAPIRouter() {
    api := s.router.Group("/api/v1", s.checkDBStateMiddleware())
    {
        api.GET("/chatlog", s.handleChatlog)
        api.GET("/contact", s.handleContacts)
        api.GET("/chatroom", s.handleChatRooms)
        api.GET("/session", s.handleSessions)
    }
}
```

#### Handling Requests (`internal/chatlog/http/handler.go`)

```go
func (s *Service) handleChatlog(c *gin.Context) {
    // 1. Parse query parameters
    timeRange, _ := c.GetQuery("time")
    talker, _ := c.GetQuery("talker")
    sender, _ := c.GetQuery("sender")
    keyword, _ := c.GetQuery("keyword")
    
    // 2. Parse time range
    start, end := parseTimeRange(timeRange)
    
    // 3. Query database
    messages, err := s.db.GetMessages(start, end, talker, sender, keyword, limit, offset)
    
    // 4. Format response
    format := c.DefaultQuery("format", "json")
    switch format {
    case "csv":
        writeCSV(c, messages)
    default:
        c.JSON(http.StatusOK, messages)
    }
}
```

## Decryption Process

### Overview

WeChat encrypts SQLite databases using AES-256-CBC with HMAC-SHA1/SHA512 for integrity.

### Decryption Algorithm

The decryption process is implemented in `internal/wechat/decrypt/`:

#### 1. Open Database File (`common/common.go`)

```go
func OpenDBFile(dbPath string, pageSize int) (*DBFile, error) {
    // Read first page
    buffer := make([]byte, pageSize)
    io.ReadFull(fp, buffer)
    
    // Check if already decrypted
    if bytes.Equal(buffer[:15], []byte("SQLite format 3")) {
        return nil, errors.ErrAlreadyDecrypted
    }
    
    // Extract salt from first 16 bytes
    return &DBFile{
        Path:       dbPath,
        Salt:       buffer[:16],
        FirstPage:  buffer,
        TotalPages: totalPages,
    }, nil
}
```

#### 2. Derive Keys (`windows/v4.go` example)

```go
func (d *V4Decryptor) deriveKeys(key []byte, salt []byte) ([]byte, []byte) {
    // PBKDF2 with SHA512, 256000 iterations
    derivedKey := pbkdf2.Key(
        key,           // Input key
        salt,          // Salt from database
        256000,        // Iterations (V4)
        64,           // Output length
        sha512.New,   // Hash function
    )
    
    // Split into encryption and MAC keys
    encKey := derivedKey[:32]  // First 32 bytes for AES
    macKey := derivedKey[32:]  // Last 32 bytes for HMAC
    
    return encKey, macKey
}
```

#### 3. Decrypt Each Page

```go
func DecryptPage(pageBuf []byte, encKey []byte, macKey []byte, pageNum int64, ...) ([]byte, error) {
    // 1. Verify HMAC
    mac := hmac.New(hashFunc, macKey)
    mac.Write(pageBuf[offset : pageSize-reserve+IVSize])
    mac.Write(pageNumberBytes)
    
    if !bytes.Equal(calculatedMAC, storedMAC) {
        return nil, errors.ErrDecryptHashVerificationFailed
    }
    
    // 2. Extract IV from page
    iv := pageBuf[pageSize-reserve : pageSize-reserve+IVSize]
    
    // 3. Decrypt with AES-CBC
    block, _ := aes.NewCipher(encKey)
    mode := cipher.NewCBCDecrypter(block, iv)
    mode.CryptBlocks(encrypted, encrypted)
    
    return decryptedPage, nil
}
```

#### 4. Write Decrypted Database

```go
func (d *V4Decryptor) Decrypt(ctx context.Context, dbfile string, hexKey string, output io.Writer) error {
    // Write SQLite header
    output.Write([]byte("SQLite format 3\x00"))
    
    // Decrypt and write each page
    for pageNum := 0; pageNum < totalPages; pageNum++ {
        // Read encrypted page
        pageBuf := make([]byte, pageSize)
        dbFile.Read(pageBuf)
        
        // Decrypt page
        decrypted, _ := common.DecryptPage(pageBuf, encKey, macKey, pageNum, ...)
        
        // Write decrypted page
        output.Write(decrypted)
    }
    
    return nil
}
```

### Platform-Specific Differences

**Windows V3** (`internal/wechat/decrypt/windows/v3.go`):
- Page size: 4096 bytes
- PBKDF2 iterations: 64,000
- Hash function: SHA-1
- HMAC size: 20 bytes

**Windows V4** (`internal/wechat/decrypt/windows/v4.go`):
- Page size: 4096 bytes
- PBKDF2 iterations: 256,000
- Hash function: SHA-512
- HMAC size: 64 bytes

**macOS** (`internal/wechat/decrypt/darwin/`):
- Similar structure to Windows
- Platform-specific key formats

## Database Access

### Database Structure (`internal/wechatdb/`)

The wechatdb package abstracts WeChat's multi-database structure:

```go
type DB struct {
    MessageDBs  []*MessageDB    // MSG0.db, MSG1.db, ...
    MicroMsgDB  *MicroMsgDB     // MicroMsg.db (contacts)
    SessionDB   *SessionDB      // Session.db
    MediaDBs    []*MediaDB      // MediaMSG0.db, ...
}
```

### Query Building (`internal/wechatdb/message.go`)

```go
func (db *DB) GetMessages(start, end time.Time, talker, sender, keyword string, limit, offset int) ([]*model.Message, error) {
    // 1. Resolve names to IDs
    talkerID := db.ResolveTalker(talker)
    senderID := db.ResolveSender(sender)
    
    // 2. Build WHERE clause
    var conditions []string
    if talkerID != "" {
        conditions = append(conditions, "StrTalker = ?")
    }
    if !start.IsZero() {
        conditions = append(conditions, "CreateTime >= ?")
    }
    // ... more conditions
    
    // 3. Query each message database
    var allMessages []*model.Message
    for _, msgDB := range db.MessageDBs {
        query := fmt.Sprintf(
            "SELECT * FROM MSG WHERE %s ORDER BY CreateTime DESC LIMIT ? OFFSET ?",
            strings.Join(conditions, " AND "),
        )
        rows, _ := msgDB.Query(query, params...)
        messages := parseMessages(rows)
        allMessages = append(allMessages, messages...)
    }
    
    // 4. Enrich with contact info
    for _, msg := range allMessages {
        msg.TalkerName = db.GetContactName(msg.Talker)
        msg.SenderName = db.GetContactName(msg.Sender)
    }
    
    return allMessages, nil
}
```

### Model Mapping (`internal/model/`)

Version-specific model definitions handle differences between WeChat versions:

- `message.go`: Generic message interface
- `message_v4.go`: WeChat 4.x message structure
- `message_darwinv3.go`: macOS WeChat 3.x message structure

## HTTP API

### Request Flow

```
Client Request
    ↓
Gin Router (route matching)
    ↓
Middleware Chain:
    - Recovery (panic handling)
    - ErrorHandler (structured errors)
    - Logger
    - CORS
    ↓
checkDBStateMiddleware (verify DB ready)
    ↓
Handler Function
    ↓
Database Service
    ↓
Response Formatting
    ↓
Client Response
```

### Media Serving (`internal/chatlog/http/handler.go`)

```go
func (s *Service) handleMedia(c *gin.Context, mediaType string) {
    // 1. Extract key from URL
    key := c.Param("key")
    
    // 2. Query media info from database
    media, err := s.db.GetMedia(mediaType, key)
    
    // 3. Construct file path
    fullPath := filepath.Join(s.conf.GetDataDir(), media.Path)
    
    // 4. Redirect to data endpoint
    c.Redirect(http.StatusFound, "/data/"+media.Path)
}

func (s *Service) handleMediaData(c *gin.Context) {
    // 1. Get file path
    path := c.Param("path")
    fullPath := filepath.Join(s.conf.GetDataDir(), path)
    
    // 2. Check if file needs decryption
    if isEncryptedImage(fullPath) {
        // Decrypt and serve
        decrypted := decryptImage(fullPath)
        c.Data(http.StatusOK, "image/jpeg", decrypted)
    } else {
        // Serve file directly
        c.File(fullPath)
    }
}
```

### Voice Conversion (`internal/chatlog/http/route.go`)

WeChat voice messages are in SILK format. The handler converts to MP3:

```go
func (s *Service) HandleVoice(c *gin.Context, data []byte) {
    // Attempt conversion
    mp3, err := silk.Silk2MP3(data)
    if err != nil {
        // Fallback: serve original SILK
        c.Data(http.StatusOK, "audio/silk", data)
        return
    }
    c.Data(http.StatusOK, "audio/mp3", mp3)
}
```

## MCP Integration

### MCP Server Setup (`internal/chatlog/http/mcp.go`)

```go
func (s *Service) initMCPServer() {
    // Create MCP server
    mcpServer := server.NewMCPServer(
        "chatlog",
        "1.0.0",
        server.WithToolCapabilities(...),
        server.WithResourceCapabilities(...),
    )
    
    // Register tools
    registerChatlogTools(mcpServer, s.db)
    
    // Register resources
    registerChatlogResources(mcpServer, s.db)
    
    // Create SSE and Streamable HTTP servers
    s.mcpSSEServer = server.NewSSEServer(mcpServer)
    s.mcpStreamableServer = server.NewStreamableHTTPServer(mcpServer)
}
```

### Tool Definitions (`internal/mcp/tool.go`)

```go
func registerChatlogTools(srv *server.MCPServer, db *database.Service) {
    // Query messages tool
    srv.AddTool(
        "query_messages",
        "Query chat messages",
        map[string]interface{}{
            "talker": map[string]string{
                "type": "string",
                "description": "Chat partner identifier",
            },
            "keyword": map[string]string{
                "type": "string", 
                "description": "Keyword to search",
            },
        },
        func(args map[string]interface{}) (*server.ToolResult, error) {
            // Extract parameters
            talker := args["talker"].(string)
            keyword := args["keyword"].(string)
            
            // Query database
            messages, _ := db.GetMessages(time.Time{}, time.Now(), talker, "", keyword, 100, 0)
            
            // Format result
            return &server.ToolResult{
                Content: formatMessages(messages),
            }, nil
        },
    )
    
    // Additional tools...
}
```

### SSE Endpoint (`internal/mcp/mcp.go`)

```go
func (m *MCP) HandleSSE(c *gin.Context) {
    // Create session
    id := uuid.New().String()
    session := NewSession(c, id)
    m.sessions[id] = session
    
    // Stream events
    c.Stream(func(w io.Writer) bool {
        <-c.Request.Context().Done()
        return false
    })
    
    // Cleanup
    delete(m.sessions, id)
}
```

## Terminal UI

### Bubble Tea Architecture

The TUI uses the Bubble Tea framework with a model-view-update pattern.

#### Application Model (`internal/chatlog/app.go`)

```go
type App struct {
    ctx     *ctx.Context
    manager *Manager
    
    // UI state
    currentView string
    menuItems   []MenuItem
    selected    int
    
    // Input state
    textInput   textinput.Model
    inputMode   bool
}

func (a *App) Init() tea.Cmd {
    return nil
}

func (a *App) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        return a.handleKeyPress(msg)
    case updateMsg:
        return a.handleUpdate(msg)
    }
    return a, nil
}

func (a *App) View() string {
    return a.renderCurrentView()
}
```

#### View Rendering (`internal/ui/`)

```go
func (a *App) renderMenu() string {
    var s strings.Builder
    
    // Header
    s.WriteString(styles.Title.Render("Chatlog Menu"))
    s.WriteString("\n\n")
    
    // Menu items
    for i, item := range a.menuItems {
        if i == a.selected {
            s.WriteString(styles.Selected.Render("→ " + item.Name))
        } else {
            s.WriteString(styles.Normal.Render("  " + item.Name))
        }
        s.WriteString("\n")
    }
    
    // Footer
    s.WriteString("\n")
    s.WriteString(styles.Help.Render("↑/↓: Navigate • Enter: Select • Esc: Back • Ctrl+C: Quit"))
    
    return s.String()
}
```

## Key Code Paths

### Path 1: Extract Key from Running WeChat

```
1. User selects "Get Data Key" in TUI
   → internal/chatlog/app.go: handleMenuSelect()

2. Call manager method
   → internal/chatlog/manager.go: GetDataKey()

3. WeChat service extracts key
   → internal/chatlog/wechat/service.go: GetDataKey()
   → internal/wechat/wechat.go: GetKey()

4. Key extraction
   → internal/wechat/key/{platform}/extractor.go: Extract()
   → Scan process memory for key pattern
   → Validate key using validator

5. Update context and config
   → internal/chatlog/ctx/context.go: Refresh()
   → Save to config file
```

### Path 2: Decrypt Databases

```
1. User selects "Decrypt Data" in TUI
   → internal/chatlog/app.go: handleMenuSelect()

2. Call manager method
   → internal/chatlog/manager.go: DecryptDBFiles()

3. WeChat service decrypts files
   → internal/chatlog/wechat/service.go: DecryptDBFiles()

4. For each .db file:
   → internal/chatlog/wechat/service.go: DecryptDBFile()
   → internal/wechat/decrypt/{platform}/{version}.go: Decrypt()
   
5. Decryption process:
   - Open encrypted database
   - Extract salt
   - Derive keys with PBKDF2
   - Decrypt each page with AES-CBC
   - Verify HMAC integrity
   - Write to work directory
```

### Path 3: Query Messages via API

```
1. HTTP Request: GET /api/v1/chatlog?talker=xxx&time=2024-01-01
   → internal/chatlog/http/route.go: router matches

2. Middleware chain
   → errors.RecoveryMiddleware()
   → errors.ErrorHandlerMiddleware()
   → checkDBStateMiddleware()

3. Handler execution
   → internal/chatlog/http/handler.go: handleChatlog()

4. Parse parameters
   - time → start/end timestamps
   - talker → WeChat ID or name
   - sender, keyword, limit, offset

5. Database query
   → internal/chatlog/database/service.go: GetMessages()
   → internal/wechatdb/message.go: GetMessages()

6. SQL query execution
   - Resolve talker name to ID
   - Build WHERE clause
   - Query message databases
   - Parse rows into models
   - Enrich with contact info

7. Format response
   - JSON or CSV format
   - Send to client
```

### Path 4: Auto-Decrypt on File Change

```
1. File monitor detects change
   → pkg/filemonitor/monitor.go: event loop

2. Callback triggered
   → internal/chatlog/wechat/service.go: DecryptFileCallback()

3. Debounce logic
   → Wait for file operations to stabilize
   → Prevent multiple rapid decryptions

4. Decrypt file
   → internal/chatlog/wechat/service.go: DecryptDBFile()
   → (same as Path 2)

5. Database service detects new data
   → internal/chatlog/database/service.go: webhook callbacks

6. Trigger webhooks (if configured)
   → internal/chatlog/webhook/webhook.go: send notification
```

### Path 5: MCP Tool Call

```
1. AI client connects via SSE
   → GET /mcp
   → internal/mcp/mcp.go: HandleSSE()

2. Client sends tool call request
   → POST /mcp/messages?sessionId=xxx
   → internal/mcp/mcp.go: HandleMessages()

3. Tool execution
   → internal/mcp/tool.go: execute tool handler

4. Database query
   → internal/chatlog/database/service.go: (same as Path 3)

5. Format result for AI
   → Convert to MCP response format
   → Send via SSE event

6. AI client receives result
   → Processes and presents to user
```

## Tips for Developers

### Adding a New Feature

1. **Identify the layer**: Is it a service, handler, or UI feature?
2. **Create model** (if needed): Define data structures in `internal/model/`
3. **Implement service logic**: Add methods to appropriate service
4. **Add API endpoint** (if needed): Define route and handler in HTTP service
5. **Update TUI** (if needed): Add menu item and handler in app
6. **Test thoroughly**: Manual testing with real WeChat data

### Debugging Tips

1. **Enable debug logging**: `chatlog --debug`
2. **Check logs**: Watch for error messages and stack traces
3. **Inspect databases**: Use SQLite tools to examine decrypted .db files
4. **Test incrementally**: Test each component separately
5. **Use HTTP directly**: Test API endpoints with curl before UI integration

### Common Gotchas

1. **Key extraction requires privileges**: SIP on macOS, admin on Windows
2. **Database locking**: SQLite can have lock issues with concurrent access
3. **File paths**: Always use absolute paths, handle cross-platform differences
4. **Memory consumption**: Large message databases can use significant memory
5. **Concurrency**: Be careful with goroutines accessing shared state

## Conclusion

This guide covered the major components and code paths in Chatlog. The architecture is modular and well-separated, making it relatively easy to understand and extend. The key is understanding the flow from user interface → manager → services → low-level operations.

For questions or contributions, please refer to the GitHub repository and discussions.
