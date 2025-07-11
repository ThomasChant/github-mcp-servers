---
title: Filesystem MCP Server
category: File System & Storage
github_url: https://github.com/magicmcp/filesystem-mcp
npm_package: "@magicmcp/filesystem"
installation: npm install -g @magicmcp/filesystem
---

# Filesystem MCP Server

The Filesystem MCP Server provides secure, controlled access to the local file system. It's designed with safety in mind, offering configurable access controls and sandboxing capabilities.

## Features

- **Read/Write Operations**: Full CRUD operations on files and directories
- **Path Sandboxing**: Restrict access to specific directories
- **Permission Control**: Fine-grained permission management
- **Streaming Support**: Efficient handling of large files
- **Watch Mode**: Monitor file changes in real-time
- **Metadata Access**: File stats, permissions, and attributes

## Installation

```bash
npm install -g @magicmcp/filesystem
```

Or use with Docker:

```bash
docker pull magicmcp/filesystem:latest
```

## Configuration

Create a `filesystem-mcp.config.json` file:

```json
{
  "allowed_paths": [
    "/home/user/documents",
    "/var/shared/data"
  ],
  "denied_paths": [
    "/etc",
    "/sys",
    "/proc"
  ],
  "max_file_size": "100MB",
  "allow_hidden_files": false,
  "permissions": {
    "read": true,
    "write": true,
    "delete": false,
    "execute": false
  }
}
```

## Usage Examples

### Basic File Operations

```javascript
// Read a file
const content = await mcp.filesystem.read('/path/to/file.txt');

// Write a file
await mcp.filesystem.write('/path/to/file.txt', 'Hello, World!');

// List directory contents
const files = await mcp.filesystem.list('/path/to/directory');

// Create directory
await mcp.filesystem.mkdir('/path/to/new/directory');

// Delete file
await mcp.filesystem.delete('/path/to/file.txt');
```

### Advanced Features

```javascript
// Watch for file changes
const watcher = await mcp.filesystem.watch('/path/to/watch', {
  recursive: true,
  events: ['create', 'change', 'delete']
});

watcher.on('change', (event) => {
  console.log('File changed:', event);
});

// Stream large files
const stream = await mcp.filesystem.stream('/path/to/large-file.bin');
stream.pipe(processStream);

// Get file metadata
const stats = await mcp.filesystem.stat('/path/to/file.txt');
console.log('File size:', stats.size);
console.log('Modified:', stats.mtime);
```

## Security Considerations

1. **Path Traversal Protection**: The server automatically prevents path traversal attacks
2. **Sandboxing**: Use Docker or VM isolation for additional security
3. **Permission Validation**: All operations check against configured permissions
4. **Audit Logging**: All file operations are logged for security auditing

## Performance

- Supports streaming for files larger than 10MB
- Automatic compression for text files
- Connection pooling for concurrent operations
- Caching layer for frequently accessed files

## Error Handling

Common error codes:

- `EACCES`: Permission denied
- `ENOENT`: File or directory not found
- `EEXIST`: File already exists
- `ENOSPC`: No space left on device
- `EMFILE`: Too many open files

## Contributing

We welcome contributions! Please see our [contributing guide](https://github.com/magicmcp/filesystem-mcp/blob/main/CONTRIBUTING.md).

## License

MIT License - see [LICENSE](https://github.com/magicmcp/filesystem-mcp/blob/main/LICENSE) for details.