---
title: "TypeScript MCP Server Tutorial: From Zero to Production"
language: TypeScript
difficulty: Intermediate
updated: 2024-01-15
prerequisites:
  - Basic TypeScript/JavaScript knowledge
  - Node.js 18+ installed
  - npm or yarn package manager
  - Familiarity with async/await
---

# TypeScript MCP Server Tutorial

Build your first MCP server using TypeScript and Node.js. This tutorial will guide you through creating a production-ready MCP server with all the essential features.

## Table of Contents

1. [Introduction](#introduction)
2. [Environment Setup](#environment-setup)
3. [Project Structure](#project-structure)
4. [Basic MCP Server](#basic-mcp-server)
5. [Implementing Core Features](#implementing-core-features)
6. [Error Handling](#error-handling)
7. [Testing](#testing)
8. [Deployment](#deployment)
9. [Best Practices](#best-practices)

## Introduction

MCP (Model Context Protocol) servers enable AI models to interact with external systems. In this tutorial, we'll build a file system MCP server that can read, write, and manage files.

## Environment Setup

### 1. Initialize Project

```bash
mkdir my-mcp-server
cd my-mcp-server
npm init -y
```

### 2. Install Dependencies

```bash
npm install @magicmcp/sdk express ws
npm install -D typescript @types/node @types/express @types/ws
npm install -D tsx nodemon eslint prettier
```

### 3. TypeScript Configuration

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Project Structure

```
my-mcp-server/
├── src/
│   ├── index.ts          # Entry point
│   ├── server.ts         # MCP server implementation
│   ├── handlers/         # Request handlers
│   │   ├── filesystem.ts
│   │   └── system.ts
│   ├── middleware/       # Middleware functions
│   │   ├── auth.ts
│   │   └── validation.ts
│   ├── utils/           # Utility functions
│   │   └── logger.ts
│   └── types/           # TypeScript types
│       └── index.ts
├── tests/               # Test files
├── config/              # Configuration files
├── package.json
├── tsconfig.json
└── README.md
```

## Basic MCP Server

### 1. Create Types

`src/types/index.ts`:

```typescript
export interface MCPRequest {
  id: string;
  method: string;
  params?: any;
}

export interface MCPResponse {
  id: string;
  result?: any;
  error?: MCPError;
}

export interface MCPError {
  code: number;
  message: string;
  data?: any;
}

export interface ServerConfig {
  port: number;
  host: string;
  maxConnections: number;
  authRequired: boolean;
}
```

### 2. Create Logger

`src/utils/logger.ts`:

```typescript
export class Logger {
  private context: string;

  constructor(context: string) {
    this.context = context;
  }

  info(message: string, meta?: any): void {
    console.log(`[${new Date().toISOString()}] [INFO] [${this.context}] ${message}`, meta || '');
  }

  error(message: string, error?: any): void {
    console.error(`[${new Date().toISOString()}] [ERROR] [${this.context}] ${message}`, error || '');
  }

  debug(message: string, meta?: any): void {
    if (process.env.DEBUG) {
      console.log(`[${new Date().toISOString()}] [DEBUG] [${this.context}] ${message}`, meta || '');
    }
  }
}
```

### 3. Create Main Server

`src/server.ts`:

```typescript
import { WebSocket, WebSocketServer } from 'ws';
import { MCPRequest, MCPResponse, ServerConfig } from './types';
import { Logger } from './utils/logger';
import { FileSystemHandler } from './handlers/filesystem';
import { SystemHandler } from './handlers/system';

export class MCPServer {
  private wss: WebSocketServer;
  private logger: Logger;
  private handlers: Map<string, Function>;
  private config: ServerConfig;

  constructor(config: ServerConfig) {
    this.config = config;
    this.logger = new Logger('MCPServer');
    this.handlers = new Map();
    this.registerHandlers();
  }

  private registerHandlers(): void {
    const fsHandler = new FileSystemHandler();
    const sysHandler = new SystemHandler();

    // File system operations
    this.handlers.set('fs.read', fsHandler.read.bind(fsHandler));
    this.handlers.set('fs.write', fsHandler.write.bind(fsHandler));
    this.handlers.set('fs.list', fsHandler.list.bind(fsHandler));
    this.handlers.set('fs.delete', fsHandler.delete.bind(fsHandler));

    // System operations
    this.handlers.set('system.info', sysHandler.getInfo.bind(sysHandler));
    this.handlers.set('system.health', sysHandler.getHealth.bind(sysHandler));
  }

  async start(): Promise<void> {
    this.wss = new WebSocketServer({
      port: this.config.port,
      host: this.config.host
    });

    this.wss.on('connection', (ws: WebSocket) => {
      this.logger.info('New client connected');
      
      ws.on('message', async (data: Buffer) => {
        try {
          const request: MCPRequest = JSON.parse(data.toString());
          const response = await this.handleRequest(request);
          ws.send(JSON.stringify(response));
        } catch (error) {
          this.logger.error('Error handling message', error);
          ws.send(JSON.stringify({
            id: 'unknown',
            error: {
              code: -32700,
              message: 'Parse error'
            }
          }));
        }
      });

      ws.on('close', () => {
        this.logger.info('Client disconnected');
      });

      ws.on('error', (error) => {
        this.logger.error('WebSocket error', error);
      });
    });

    this.logger.info(`MCP Server started on ${this.config.host}:${this.config.port}`);
  }

  private async handleRequest(request: MCPRequest): Promise<MCPResponse> {
    this.logger.debug('Handling request', request);

    const handler = this.handlers.get(request.method);
    
    if (!handler) {
      return {
        id: request.id,
        error: {
          code: -32601,
          message: 'Method not found'
        }
      };
    }

    try {
      const result = await handler(request.params);
      return {
        id: request.id,
        result
      };
    } catch (error) {
      this.logger.error(`Error in handler ${request.method}`, error);
      return {
        id: request.id,
        error: {
          code: -32603,
          message: 'Internal error',
          data: error.message
        }
      };
    }
  }

  async stop(): Promise<void> {
    if (this.wss) {
      this.wss.close(() => {
        this.logger.info('Server stopped');
      });
    }
  }
}
```

## Implementing Core Features

### File System Handler

`src/handlers/filesystem.ts`:

```typescript
import * as fs from 'fs/promises';
import * as path from 'path';
import { Logger } from '../utils/logger';

export class FileSystemHandler {
  private logger: Logger;
  private basePath: string;

  constructor() {
    this.logger = new Logger('FileSystemHandler');
    this.basePath = process.env.MCP_BASE_PATH || process.cwd();
  }

  async read(params: { path: string }): Promise<string> {
    const fullPath = this.resolvePath(params.path);
    this.validatePath(fullPath);
    
    const content = await fs.readFile(fullPath, 'utf-8');
    this.logger.info(`Read file: ${params.path}`);
    return content;
  }

  async write(params: { path: string; content: string }): Promise<void> {
    const fullPath = this.resolvePath(params.path);
    this.validatePath(fullPath);
    
    await fs.writeFile(fullPath, params.content, 'utf-8');
    this.logger.info(`Wrote file: ${params.path}`);
  }

  async list(params: { path: string }): Promise<string[]> {
    const fullPath = this.resolvePath(params.path);
    this.validatePath(fullPath);
    
    const files = await fs.readdir(fullPath);
    this.logger.info(`Listed directory: ${params.path}`);
    return files;
  }

  async delete(params: { path: string }): Promise<void> {
    const fullPath = this.resolvePath(params.path);
    this.validatePath(fullPath);
    
    await fs.unlink(fullPath);
    this.logger.info(`Deleted file: ${params.path}`);
  }

  private resolvePath(relativePath: string): string {
    return path.resolve(this.basePath, relativePath);
  }

  private validatePath(fullPath: string): void {
    if (!fullPath.startsWith(this.basePath)) {
      throw new Error('Path traversal attempt detected');
    }
  }
}
```

### Authentication Middleware

`src/middleware/auth.ts`:

```typescript
import { MCPRequest } from '../types';
import crypto from 'crypto';

export class AuthMiddleware {
  private tokens: Set<string>;

  constructor() {
    this.tokens = new Set();
    // In production, load from secure storage
    this.tokens.add(process.env.MCP_AUTH_TOKEN || 'default-token');
  }

  async authenticate(request: MCPRequest): Promise<boolean> {
    const authHeader = request.params?.auth;
    
    if (!authHeader) {
      return false;
    }

    const [type, token] = authHeader.split(' ');
    
    if (type !== 'Bearer' || !this.tokens.has(token)) {
      return false;
    }

    return true;
  }

  generateToken(): string {
    const token = crypto.randomBytes(32).toString('hex');
    this.tokens.add(token);
    return token;
  }

  revokeToken(token: string): void {
    this.tokens.delete(token);
  }
}
```

## Error Handling

Create comprehensive error handling:

`src/utils/errors.ts`:

```typescript
export class MCPError extends Error {
  code: number;
  data?: any;

  constructor(code: number, message: string, data?: any) {
    super(message);
    this.code = code;
    this.data = data;
  }
}

export class ValidationError extends MCPError {
  constructor(message: string, data?: any) {
    super(-32602, `Invalid params: ${message}`, data);
  }
}

export class NotFoundError extends MCPError {
  constructor(resource: string) {
    super(-32001, `Resource not found: ${resource}`);
  }
}

export class PermissionError extends MCPError {
  constructor(operation: string) {
    super(-32002, `Permission denied: ${operation}`);
  }
}
```

## Testing

### Unit Tests

`tests/handlers/filesystem.test.ts`:

```typescript
import { describe, it, expect, beforeEach, afterEach } from '@jest/globals';
import { FileSystemHandler } from '../../src/handlers/filesystem';
import * as fs from 'fs/promises';
import * as path from 'path';

describe('FileSystemHandler', () => {
  let handler: FileSystemHandler;
  const testDir = path.join(__dirname, 'test-files');

  beforeEach(async () => {
    handler = new FileSystemHandler();
    await fs.mkdir(testDir, { recursive: true });
  });

  afterEach(async () => {
    await fs.rm(testDir, { recursive: true, force: true });
  });

  describe('read', () => {
    it('should read file content', async () => {
      const testFile = path.join(testDir, 'test.txt');
      await fs.writeFile(testFile, 'Hello, World!');

      const content = await handler.read({ path: 'test.txt' });
      expect(content).toBe('Hello, World!');
    });

    it('should throw error for non-existent file', async () => {
      await expect(handler.read({ path: 'non-existent.txt' }))
        .rejects.toThrow();
    });
  });

  describe('write', () => {
    it('should write file content', async () => {
      await handler.write({ 
        path: 'test.txt', 
        content: 'New content' 
      });

      const content = await fs.readFile(
        path.join(testDir, 'test.txt'), 
        'utf-8'
      );
      expect(content).toBe('New content');
    });
  });
});
```

### Integration Tests

`tests/server.test.ts`:

```typescript
import { MCPServer } from '../src/server';
import WebSocket from 'ws';

describe('MCPServer Integration', () => {
  let server: MCPServer;
  let client: WebSocket;

  beforeEach(async () => {
    server = new MCPServer({
      port: 8081,
      host: 'localhost',
      maxConnections: 10,
      authRequired: false
    });
    await server.start();
  });

  afterEach(async () => {
    if (client) client.close();
    await server.stop();
  });

  it('should handle filesystem operations', (done) => {
    client = new WebSocket('ws://localhost:8081');

    client.on('open', () => {
      client.send(JSON.stringify({
        id: '1',
        method: 'fs.list',
        params: { path: '.' }
      }));
    });

    client.on('message', (data) => {
      const response = JSON.parse(data.toString());
      expect(response.id).toBe('1');
      expect(response.result).toBeInstanceOf(Array);
      done();
    });
  });
});
```

## Deployment

### Docker Configuration

`Dockerfile`:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist

EXPOSE 8080

USER node

CMD ["node", "dist/index.js"]
```

### Production Configuration

`config/production.json`:

```json
{
  "server": {
    "port": 8080,
    "host": "0.0.0.0",
    "maxConnections": 100,
    "authRequired": true
  },
  "security": {
    "rateLimiting": {
      "windowMs": 60000,
      "maxRequests": 100
    },
    "cors": {
      "origin": ["https://magicmcp.net"],
      "credentials": true
    }
  },
  "logging": {
    "level": "info",
    "format": "json"
  }
}
```

## Best Practices

### 1. Security
- Always validate and sanitize input
- Implement proper authentication
- Use rate limiting
- Enable CORS appropriately
- Never expose sensitive data in logs

### 2. Performance
- Use connection pooling for databases
- Implement caching where appropriate
- Stream large files instead of loading into memory
- Use compression for large responses

### 3. Error Handling
- Use structured error codes
- Provide meaningful error messages
- Log errors appropriately
- Implement retry logic for transient failures

### 4. Code Organization
- Keep handlers focused and single-purpose
- Use dependency injection
- Write comprehensive tests
- Document your API thoroughly

### 5. Monitoring
- Implement health checks
- Use structured logging
- Monitor performance metrics
- Set up alerts for critical errors

## Next Steps

1. **Add More Features**: Implement additional handlers for your specific use case
2. **Enhance Security**: Add OAuth2 or JWT authentication
3. **Scale Horizontally**: Implement clustering for high availability
4. **Add Persistence**: Integrate with databases for state management
5. **Create Client SDK**: Build a TypeScript client library

## Resources

- [MCP Protocol Specification](https://docs.magicmcp.net/protocol)
- [TypeScript MCP SDK](https://github.com/magicmcp/typescript-sdk)
- [Example MCP Servers](https://github.com/magicmcp/examples)
- [Community Forum](https://forum.magicmcp.net)

Congratulations! You've built your first MCP server with TypeScript. Continue exploring the MCP ecosystem and build amazing integrations for AI models.