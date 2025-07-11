---
layout: default
title: MCP Servers Directory
permalink: /mcp-servers/
---

# MCP Servers Directory

Browse our collection of MCP servers organized by category. Each server includes detailed documentation, installation instructions, and practical examples.

## Categories

### 📁 File System & Storage
- [Filesystem MCP Server](./filesystem-mcp/) - Complete local file system access
- [S3 MCP Server](./s3-mcp/) - Amazon S3 integration
- [Google Drive MCP](./google-drive-mcp/) - Google Drive file operations
- [Dropbox MCP](./dropbox-mcp/) - Dropbox file management

### 🗄️ Databases
- [PostgreSQL MCP](./postgresql-mcp/) - PostgreSQL database operations
- [MongoDB MCP](./mongodb-mcp/) - MongoDB NoSQL database
- [Redis MCP](./redis-mcp/) - Redis cache and data store
- [SQLite MCP](./sqlite-mcp/) - Lightweight SQL database

### 🌐 Web & APIs
- [HTTP Client MCP](./http-client-mcp/) - Generic HTTP/REST API client
- [Web Browser MCP](./web-browser-mcp/) - Headless browser automation
- [GraphQL MCP](./graphql-mcp/) - GraphQL API interactions
- [WebSocket MCP](./websocket-mcp/) - Real-time WebSocket connections

### 💬 Communication
- [Slack MCP](./slack-mcp/) - Slack workspace integration
- [Discord MCP](./discord-mcp/) - Discord bot and API
- [Email MCP](./email-mcp/) - Send and receive emails
- [Telegram MCP](./telegram-mcp/) - Telegram bot integration

### 🛠️ Development Tools
- [GitHub MCP](./github-mcp/) - GitHub repository management
- [GitLab MCP](./gitlab-mcp/) - GitLab integration
- [Docker MCP](./docker-mcp/) - Docker container management
- [Kubernetes MCP](./kubernetes-mcp/) - Kubernetes cluster operations

### 📊 Data Processing
- [CSV MCP](./csv-mcp/) - CSV file processing
- [JSON MCP](./json-mcp/) - JSON data manipulation
- [Excel MCP](./excel-mcp/) - Excel file operations
- [PDF MCP](./pdf-mcp/) - PDF generation and parsing

### 🤖 AI & ML
- [OpenAI MCP](./openai-mcp/) - OpenAI API integration
- [Anthropic MCP](./anthropic-mcp/) - Anthropic Claude API
- [Hugging Face MCP](./huggingface-mcp/) - Hugging Face models
- [Vector DB MCP](./vectordb-mcp/) - Vector database operations

### 🔧 Utilities
- [Shell MCP](./shell-mcp/) - Command execution
- [Cron MCP](./cron-mcp/) - Scheduled task management
- [Environment MCP](./env-mcp/) - Environment variable management
- [Crypto MCP](./crypto-mcp/) - Cryptographic operations

## Featured Servers

<div class="featured-servers">
  <div class="server-card">
    <h3>🌟 Filesystem MCP Server</h3>
    <p>The most popular MCP server for file system operations. Provides safe, sandboxed access to local files with comprehensive security controls.</p>
    <a href="./filesystem-mcp/">Learn more →</a>
  </div>

  <div class="server-card">
    <h3>🚀 GitHub MCP</h3>
    <p>Complete GitHub integration including repository management, issues, pull requests, and GitHub Actions.</p>
    <a href="./github-mcp/">Learn more →</a>
  </div>

  <div class="server-card">
    <h3>🗃️ PostgreSQL MCP</h3>
    <p>Production-ready PostgreSQL integration with connection pooling, prepared statements, and transaction support.</p>
    <a href="./postgresql-mcp/">Learn more →</a>
  </div>
</div>

## How to Choose an MCP Server

When selecting an MCP server for your project, consider:

1. **Compatibility**: Ensure the server is compatible with your AI model and framework
2. **Security**: Review the security features and access controls
3. **Performance**: Check benchmarks and performance characteristics
4. **Documentation**: Look for comprehensive documentation and examples
5. **Community**: Consider the size and activity of the community

## Submit Your MCP Server

Have you built an MCP server? We'd love to include it in our directory!

[Submit your MCP server →](https://github.com/magicmcp/mcp-servers/issues/new?template=new-server.md)

<style>
.featured-servers {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  margin: 30px 0;
}

.server-card {
  border: 1px solid #e1e4e8;
  border-radius: 8px;
  padding: 20px;
  background: #f6f8fa;
}

.server-card h3 {
  margin-top: 0;
}

.server-card a {
  font-weight: bold;
}
</style>