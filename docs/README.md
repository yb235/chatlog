# Chatlog Documentation

Welcome to the Chatlog documentation! This directory contains comprehensive guides to help you understand, use, and extend Chatlog.

## 📚 Documentation Overview

### For First-Time Users

Start here if you're new to Chatlog:

- **[Beginner's Guide](BEGINNERS_GUIDE.md)** 🌟
  - Complete introduction for non-technical users
  - Step-by-step installation and setup
  - Basic concepts explained simply
  - Common tasks and examples
  - FAQ and troubleshooting

### For Daily Users

These guides help you get the most out of Chatlog:

- **[Workflow Guide](WORKFLOW.md)** 🔄
  - Common usage patterns
  - Daily operation workflows
  - Advanced scenarios
  - Docker deployment
  - Automation examples

- **[Prompt Guide](prompt.md)** 💬
  - AI assistant integration examples
  - Effective prompts for chat analysis
  - MCP usage patterns

### For Developers and Advanced Users

Technical documentation for those who want to dig deeper:

- **[Architecture Documentation](ARCHITECTURE.md)** 🏗️
  - System architecture overview
  - Component design and interactions
  - Data flow diagrams
  - Technology stack details
  - Extension points

- **[Code Guide](CODE_GUIDE.md)** 💻
  - Detailed code walkthrough
  - Project structure
  - Key code paths
  - Implementation details
  - Developer tips

- **[API Reference](API_REFERENCE.md)** 📡
  - Complete REST API documentation
  - Endpoint specifications
  - Request/response formats
  - MCP protocol details
  - Usage examples

### For Deployment

Deployment and integration guides:

- **[Docker Guide](docker.md)** 🐳
  - Docker installation
  - Container configuration
  - NAS deployment
  - Environment variables

- **[MCP Integration](mcp.md)** 🤖
  - Model Context Protocol setup
  - AI assistant integration
  - Platform-specific guides (Claude, ChatWise, Cherry Studio)

## 🚀 Quick Navigation

### By User Type

#### I'm a Complete Beginner
1. Start with [Beginner's Guide](BEGINNERS_GUIDE.md)
2. Follow the installation steps
3. Learn basic concepts
4. Try common tasks

#### I Want to Use Chatlog Daily
1. Read [Workflow Guide](WORKFLOW.md)
2. Check [API Reference](API_REFERENCE.md) for query syntax
3. Explore [Prompt Guide](prompt.md) for AI integration

#### I'm a Developer
1. Review [Architecture Documentation](ARCHITECTURE.md)
2. Study [Code Guide](CODE_GUIDE.md)
3. Check [API Reference](API_REFERENCE.md)
4. Explore the source code

#### I Want to Deploy on Server/NAS
1. Read [Docker Guide](docker.md)
2. Follow [Workflow Guide](WORKFLOW.md) for deployment scenarios
3. Check [API Reference](API_REFERENCE.md) for remote access

### By Task

#### Installing Chatlog
- [Beginner's Guide → Getting Started](BEGINNERS_GUIDE.md#getting-started)
- [Docker Guide](docker.md) (for server deployment)

#### Extracting Keys
- [Beginner's Guide → Initial Setup](BEGINNERS_GUIDE.md#initial-setup)
- [Workflow Guide → First-Time Setup](WORKFLOW.md#first-time-setup)

#### Querying Messages
- [API Reference → Query Chat Logs](API_REFERENCE.md#query-chat-logs)
- [Workflow Guide → Daily Usage](WORKFLOW.md#daily-usage-workflows)

#### Exporting Data
- [Workflow Guide → Export Chat History](WORKFLOW.md#workflow-2-export-chat-history)
- [API Reference → Response Formats](API_REFERENCE.md#response-formats)

#### AI Integration
- [MCP Integration](mcp.md)
- [Prompt Guide](prompt.md)
- [Workflow Guide → AI Assistant Integration](WORKFLOW.md#workflow-6-ai-assistant-integration-mcp)

#### Troubleshooting
- [Beginner's Guide → FAQ](BEGINNERS_GUIDE.md#frequently-asked-questions)
- [Workflow Guide → Troubleshooting](WORKFLOW.md#troubleshooting-workflow)

## 📖 Documentation Structure

```
docs/
├── README.md                 # This file - Documentation index
├── BEGINNERS_GUIDE.md       # Complete beginner's guide
├── ARCHITECTURE.md          # System architecture
├── CODE_GUIDE.md            # Code walkthrough
├── API_REFERENCE.md         # API documentation
├── WORKFLOW.md              # Usage workflows
├── docker.md                # Docker deployment
├── mcp.md                   # MCP integration
└── prompt.md                # AI prompt examples
```

## 🎯 Learning Paths

### Path 1: Quick Start (30 minutes)
1. Read [Beginner's Guide → What is Chatlog?](BEGINNERS_GUIDE.md#what-is-chatlog)
2. Follow [Beginner's Guide → Getting Started](BEGINNERS_GUIDE.md#getting-started)
3. Try [Beginner's Guide → Common Tasks](BEGINNERS_GUIDE.md#common-tasks)

### Path 2: Power User (2 hours)
1. Complete Path 1
2. Study [API Reference](API_REFERENCE.md)
3. Practice [Workflow Guide → Daily Usage](WORKFLOW.md#daily-usage-workflows)
4. Explore [Workflow Guide → Advanced Workflows](WORKFLOW.md#advanced-workflows)

### Path 3: Developer (4 hours)
1. Complete Path 1
2. Read [Architecture Documentation](ARCHITECTURE.md)
3. Study [Code Guide](CODE_GUIDE.md)
4. Review [API Reference](API_REFERENCE.md)
5. Explore source code

### Path 4: NAS/Server Deployment (1 hour)
1. Read [Docker Guide](docker.md)
2. Follow [Workflow Guide → Docker Deployment](WORKFLOW.md#docker-deployment-workflow)
3. Review [API Reference](API_REFERENCE.md) for remote access

## 🔍 Key Topics

### Understanding Core Concepts

**Encryption & Decryption**
- [Beginner's Guide → Encryption Key](BEGINNERS_GUIDE.md#1-encryption-key-data-key)
- [Code Guide → Decryption Process](CODE_GUIDE.md#decryption-process)
- [Architecture → Data Flow](ARCHITECTURE.md#data-flow)

**Database Structure**
- [Beginner's Guide → Database Files](BEGINNERS_GUIDE.md#2-database-files)
- [Architecture → Database Schema](ARCHITECTURE.md#database-schema)
- [Code Guide → Database Access](CODE_GUIDE.md#database-access)

**HTTP API**
- [Beginner's Guide → HTTP Service](BEGINNERS_GUIDE.md#5-http-service--api)
- [API Reference](API_REFERENCE.md)
- [Architecture → HTTP Service](ARCHITECTURE.md#4-http-service)

**MCP Protocol**
- [MCP Integration](mcp.md)
- [API Reference → MCP Protocol](API_REFERENCE.md#mcp-protocol)
- [Architecture → MCP Service](ARCHITECTURE.md#5-mcp-service)

### Common Operations

**Key Extraction**
- How it works: [Code Guide → Key Extraction](CODE_GUIDE.md#getting-wechat-instances)
- Usage: [Workflow Guide → Extract Key](WORKFLOW.md#scenario-1-local-developmentpersonal-use-macos)
- Troubleshooting: [Workflow Guide → Cannot Extract Key](WORKFLOW.md#issue-1-cannot-extract-key)

**Database Decryption**
- Algorithm: [Code Guide → Decryption Algorithm](CODE_GUIDE.md#decryption-algorithm)
- Usage: [Workflow Guide → Decrypt Databases](WORKFLOW.md#path-2-decrypt-databases)
- Details: [Architecture → Decryption Flow](ARCHITECTURE.md#decryption-flow)

**Querying Messages**
- API: [API Reference → Query Chat Logs](API_REFERENCE.md#query-chat-logs)
- Examples: [Workflow Guide → Query Recent Messages](WORKFLOW.md#workflow-1-query-recent-messages)
- Code path: [Code Guide → Query Messages via API](CODE_GUIDE.md#path-3-query-messages-via-api)

## 🛠️ Tools & Integrations

### Web Interface
- Access: `http://localhost:5030`
- Guide: [Beginner's Guide → Web Interface](BEGINNERS_GUIDE.md#web-interface-browser)
- Features: [API Reference → Base Information](API_REFERENCE.md#base-information)

### Command Line
- Tools: curl, Python, bash scripts
- Examples: [API Reference → Examples](API_REFERENCE.md#examples)
- Scripts: [Workflow Guide → Advanced Workflows](WORKFLOW.md#advanced-workflows)

### AI Assistants
- Claude Desktop: [MCP Guide → Claude Desktop](mcp.md)
- ChatWise: [MCP Guide → ChatWise](mcp.md)
- Cherry Studio: [MCP Guide → Cherry Studio](mcp.md)
- Prompts: [Prompt Guide](prompt.md)

### Docker
- Installation: [Docker Guide](docker.md)
- Configuration: [Docker Guide → Environment Variables](docker.md)
- Deployment: [Workflow Guide → Docker Deployment](WORKFLOW.md#docker-deployment-workflow)

## 💡 Tips for Reading Documentation

### For Beginners
- Start from the beginning, don't skip sections
- Try examples as you read
- Don't worry about understanding everything
- Focus on tasks you need
- Ask questions in GitHub Discussions

### For Advanced Users
- Skim familiar sections
- Focus on architecture and code guides
- Try advanced workflows
- Read API reference for integration
- Contribute improvements via Pull Requests

### For Developers
- Read architecture first to understand design
- Study code guide for implementation details
- API reference for interface contracts
- Check source code for latest changes
- Follow coding conventions

## 🤝 Contributing to Documentation

We welcome documentation improvements! Here's how:

### Reporting Issues
- Found an error? [Open an issue](https://github.com/sjzar/chatlog/issues)
- Missing information? Request it in [Discussions](https://github.com/sjzar/chatlog/discussions)
- Have a suggestion? Create a discussion topic

### Improving Docs
1. Fork the repository
2. Edit documentation files
3. Submit a pull request
4. Describe your changes

### Documentation Standards
- Clear, concise language
- Code examples that work
- Proper formatting (Markdown)
- Links to related sections
- Keep beginners in mind

## 📞 Getting Help

### Resources by Type

**I Need Help Using Chatlog**
- Read: [Beginner's Guide → FAQ](BEGINNERS_GUIDE.md#frequently-asked-questions)
- Ask: [GitHub Discussions](https://github.com/sjzar/chatlog/discussions)

**I Found a Bug**
- Check: [Existing issues](https://github.com/sjzar/chatlog/issues)
- Report: [New issue](https://github.com/sjzar/chatlog/issues/new)
- Include: OS, WeChat version, error messages

**I Want a Feature**
- Check: [Existing discussions](https://github.com/sjzar/chatlog/discussions)
- Request: [New discussion](https://github.com/sjzar/chatlog/discussions/new)
- Describe: Use case and benefits

**I Want to Contribute**
- Read: [Code Guide](CODE_GUIDE.md)
- Review: [Architecture](ARCHITECTURE.md)
- Discuss: [Contribution ideas](https://github.com/sjzar/chatlog/discussions)

## 🔗 External Resources

### WeChat Information
- WeChat for Windows: https://windows.weixin.qq.com/
- WeChat for Mac: https://mac.weixin.qq.com/

### Related Technologies
- SQLite: https://www.sqlite.org/
- Go Language: https://golang.org/
- Gin Framework: https://gin-gonic.com/
- Bubble Tea (TUI): https://github.com/charmbracelet/bubbletea

### MCP Protocol
- Specification: https://spec.modelcontextprotocol.io/
- Python SDK: https://github.com/modelcontextprotocol/python-sdk
- Go Implementation: https://github.com/mark3labs/mcp-go

## 📋 Documentation Versions

Current documentation is for:
- **Chatlog Version**: 1.0.0+
- **Last Updated**: 2024
- **Compatible with**: WeChat 3.x, 4.x

For older versions, check the git history or releases.

## 🎓 Additional Learning

### Tutorials
- Coming soon: Video tutorials
- Coming soon: Step-by-step guides
- Coming soon: Use case studies

### Community Content
- [GitHub Discussions](https://github.com/sjzar/chatlog/discussions) - User experiences
- [Issues](https://github.com/sjzar/chatlog/issues) - Common problems and solutions

### Advanced Topics
- Database optimization
- Custom integrations
- Security hardening
- Performance tuning

## ⚖️ Legal & Privacy

**Important Information:**
- [Disclaimer](../DISCLAIMER.md) - Usage terms and liability
- [License](../LICENSE) - Apache-2.0 License
- [Privacy Policy](../README.md#隐私政策) - Data handling

**Remember:**
- Use only with your own data
- Never access others' chats without permission
- Keep encryption keys secure
- Be aware of privacy laws in your region

## 📝 Documentation Maintenance

This documentation is actively maintained. If you notice:
- Outdated information
- Broken links
- Missing content
- Errors or typos

Please let us know via [GitHub Issues](https://github.com/sjzar/chatlog/issues) or submit a pull request!

---

**Ready to get started?** Begin with the [Beginner's Guide](BEGINNERS_GUIDE.md)!

**Have questions?** Visit [GitHub Discussions](https://github.com/sjzar/chatlog/discussions)!

**Want to contribute?** Read the [Code Guide](CODE_GUIDE.md) and [Architecture](ARCHITECTURE.md)!
