---
title: PostgreSQL MCP Server
category: Databases
github_url: https://github.com/magicmcp/postgresql-mcp
npm_package: "@magicmcp/postgresql"
installation: npm install -g @magicmcp/postgresql
---

# PostgreSQL MCP Server

The PostgreSQL MCP Server provides production-ready integration with PostgreSQL databases, offering secure query execution, connection pooling, and comprehensive database management capabilities.

## Features

- **Connection Pooling**: Efficient connection management with configurable pool settings
- **Prepared Statements**: Protection against SQL injection attacks
- **Transaction Support**: Full ACID transaction management
- **Schema Management**: Create, modify, and manage database schemas
- **Query Builder**: Type-safe query construction
- **Streaming Results**: Efficient handling of large result sets
- **Real-time Notifications**: PostgreSQL LISTEN/NOTIFY support
- **Backup & Restore**: Database backup and restoration utilities

## Installation

```bash
npm install -g @magicmcp/postgresql
```

## Configuration

Create a `postgresql-mcp.config.json` file:

```json
{
  "connection": {
    "host": "localhost",
    "port": 5432,
    "database": "myapp",
    "user": "dbuser",
    "password": "secure_password",
    "ssl": {
      "rejectUnauthorized": true,
      "ca": "/path/to/ca.crt"
    }
  },
  "pool": {
    "min": 2,
    "max": 20,
    "idleTimeoutMillis": 30000,
    "connectionTimeoutMillis": 2000
  },
  "security": {
    "allowed_schemas": ["public", "app_data"],
    "read_only": false,
    "statement_timeout": 30000,
    "allowed_operations": ["SELECT", "INSERT", "UPDATE", "DELETE"]
  }
}
```

## Connection String

Alternatively, use a connection string:

```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/myapp?ssl=true"
```

## Usage Examples

### Basic Queries

```javascript
// Simple SELECT query
const users = await mcp.postgresql.query(
  'SELECT * FROM users WHERE active = $1',
  [true]
);

// INSERT with returning
const newUser = await mcp.postgresql.query(
  'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
  ['John Doe', 'john@example.com']
);

// UPDATE operation
const updated = await mcp.postgresql.query(
  'UPDATE users SET last_login = NOW() WHERE id = $1',
  [userId]
);
```

### Query Builder

```javascript
// Type-safe query building
const query = mcp.postgresql.select()
  .from('users')
  .where('age', '>=', 18)
  .where('active', true)
  .orderBy('created_at', 'DESC')
  .limit(10);

const results = await query.execute();

// Complex joins
const posts = await mcp.postgresql.select()
  .from('posts')
  .join('users', 'posts.user_id', 'users.id')
  .join('categories', 'posts.category_id', 'categories.id')
  .where('posts.published', true)
  .select(['posts.*', 'users.name as author', 'categories.name as category'])
  .execute();
```

### Transactions

```javascript
// Transaction with automatic rollback on error
await mcp.postgresql.transaction(async (client) => {
  // Deduct from account A
  await client.query(
    'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
    [100, accountA]
  );
  
  // Add to account B
  await client.query(
    'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
    [100, accountB]
  );
  
  // Log transaction
  await client.query(
    'INSERT INTO transactions (from_account, to_account, amount) VALUES ($1, $2, $3)',
    [accountA, accountB, 100]
  );
});

// Manual transaction control
const client = await mcp.postgresql.getClient();
try {
  await client.query('BEGIN');
  // ... perform operations
  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK');
  throw error;
} finally {
  client.release();
}
```

### Schema Management

```javascript
// Create table
await mcp.postgresql.schema.createTable('products', {
  id: { type: 'SERIAL', primaryKey: true },
  name: { type: 'VARCHAR(255)', notNull: true },
  price: { type: 'DECIMAL(10,2)', notNull: true },
  description: { type: 'TEXT' },
  created_at: { type: 'TIMESTAMP', default: 'NOW()' }
});

// Add index
await mcp.postgresql.schema.createIndex('products', ['name'], {
  unique: false,
  method: 'btree'
});

// Alter table
await mcp.postgresql.schema.alterTable('products', {
  add: {
    category_id: { type: 'INTEGER', references: 'categories(id)' }
  },
  modify: {
    price: { type: 'DECIMAL(12,2)' }
  }
});
```

### Streaming Large Results

```javascript
// Stream results for memory efficiency
const stream = await mcp.postgresql.stream(
  'SELECT * FROM large_table WHERE created_at >= $1',
  [startDate]
);

stream.on('data', (row) => {
  // Process each row
  processRow(row);
});

stream.on('end', () => {
  console.log('Stream completed');
});

stream.on('error', (error) => {
  console.error('Stream error:', error);
});
```

### LISTEN/NOTIFY

```javascript
// Set up real-time notifications
await mcp.postgresql.listen('order_updates', (payload) => {
  console.log('New order update:', payload);
  // React to the notification
});

// Send notification from another connection
await mcp.postgresql.notify('order_updates', {
  orderId: 12345,
  status: 'shipped'
});
```

### Backup and Restore

```javascript
// Create backup
const backup = await mcp.postgresql.backup({
  format: 'custom',
  compress: true,
  schema_only: false,
  tables: ['users', 'orders', 'products']
});

// Restore from backup
await mcp.postgresql.restore({
  backup_file: '/path/to/backup.dump',
  clean: true,
  create: true,
  exit_on_error: false
});
```

## Performance Optimization

### Connection Pooling Best Practices

```javascript
// Monitor pool statistics
const stats = await mcp.postgresql.pool.stats();
console.log('Total connections:', stats.total);
console.log('Idle connections:', stats.idle);
console.log('Waiting requests:', stats.waiting);

// Adjust pool size dynamically
await mcp.postgresql.pool.setSize({
  min: 5,
  max: 50
});
```

### Query Optimization

```javascript
// Explain query plan
const plan = await mcp.postgresql.explain(
  'SELECT * FROM users WHERE email = $1',
  ['user@example.com']
);

// Analyze query performance
const analysis = await mcp.postgresql.analyze(
  'SELECT * FROM orders WHERE created_at >= $1',
  [startDate]
);
```

## Security Features

1. **SQL Injection Prevention**: All queries use parameterized statements
2. **Connection Encryption**: SSL/TLS support for secure connections
3. **Row-Level Security**: Support for PostgreSQL RLS policies
4. **Audit Logging**: Comprehensive query logging capabilities
5. **Access Control**: Schema and operation-level restrictions

## Error Handling

Common PostgreSQL error codes:

- `23505`: Unique constraint violation
- `23503`: Foreign key constraint violation
- `42P01`: Table does not exist
- `42703`: Column does not exist
- `57014`: Query cancelled due to timeout

```javascript
try {
  await mcp.postgresql.query('INSERT INTO users (email) VALUES ($1)', [email]);
} catch (error) {
  if (error.code === '23505') {
    console.error('Email already exists');
  } else {
    throw error;
  }
}
```

## Monitoring and Metrics

```javascript
// Get database statistics
const stats = await mcp.postgresql.getDatabaseStats();
console.log('Database size:', stats.size);
console.log('Active connections:', stats.connections);
console.log('Cache hit ratio:', stats.cache_hit_ratio);

// Monitor slow queries
const slowQueries = await mcp.postgresql.getSlowQueries({
  threshold: 1000, // ms
  limit: 10
});
```

## License

MIT License - see [LICENSE](https://github.com/magicmcp/postgresql-mcp/blob/main/LICENSE) for details.