---
title: GitHub MCP Server
category: Development Tools
github_url: https://github.com/magicmcp/github-mcp
npm_package: "@magicmcp/github"
installation: npm install -g @magicmcp/github
---

# GitHub MCP Server

The GitHub MCP Server provides comprehensive integration with GitHub's API, enabling AI models to interact with repositories, issues, pull requests, and more.

## Features

- **Repository Management**: Create, clone, fork, and manage repositories
- **Issues & Projects**: Create and manage issues, projects, and milestones
- **Pull Requests**: Create, review, and merge pull requests
- **GitHub Actions**: Trigger and monitor workflow runs
- **Gists**: Create and manage GitHub Gists
- **Webhooks**: Set up and manage repository webhooks
- **Search**: Search repositories, code, issues, and users

## Installation

```bash
npm install -g @magicmcp/github
```

## Authentication

The GitHub MCP requires a GitHub Personal Access Token (PAT):

1. Go to GitHub Settings → Developer settings → Personal access tokens
2. Generate a new token with required scopes:
   - `repo` - Full repository access
   - `workflow` - GitHub Actions access
   - `gist` - Gist access
   - `admin:org` - Organization management (optional)

3. Set the token in your environment:
```bash
export GITHUB_TOKEN=your_personal_access_token
```

## Configuration

Create a `github-mcp.config.json` file:

```json
{
  "default_owner": "your-username",
  "default_branch": "main",
  "api_version": "2022-11-28",
  "per_page": 100,
  "timeout": 30000,
  "retry_attempts": 3,
  "rate_limit_strategy": "pause"
}
```

## Usage Examples

### Repository Operations

```javascript
// Create a new repository
const repo = await mcp.github.createRepo({
  name: 'my-new-project',
  description: 'A new project created via MCP',
  private: false,
  auto_init: true,
  license_template: 'mit'
});

// Clone a repository
await mcp.github.clone({
  owner: 'magicmcp',
  repo: 'example-repo',
  local_path: '/path/to/local/clone'
});

// List repository contents
const contents = await mcp.github.getContents({
  owner: 'magicmcp',
  repo: 'example-repo',
  path: 'src/'
});
```

### Issues and Pull Requests

```javascript
// Create an issue
const issue = await mcp.github.createIssue({
  owner: 'magicmcp',
  repo: 'example-repo',
  title: 'Bug: Application crashes on startup',
  body: 'Detailed description of the issue...',
  labels: ['bug', 'high-priority'],
  assignees: ['username']
});

// Create a pull request
const pr = await mcp.github.createPullRequest({
  owner: 'magicmcp',
  repo: 'example-repo',
  title: 'Fix: Resolve startup crash',
  head: 'fix-startup-crash',
  base: 'main',
  body: 'This PR fixes the startup crash by...'
});

// Review a pull request
await mcp.github.reviewPullRequest({
  owner: 'magicmcp',
  repo: 'example-repo',
  pull_number: pr.number,
  event: 'APPROVE',
  body: 'LGTM! Great work.'
});
```

### GitHub Actions

```javascript
// Trigger a workflow
await mcp.github.triggerWorkflow({
  owner: 'magicmcp',
  repo: 'example-repo',
  workflow_id: 'ci.yml',
  ref: 'main',
  inputs: {
    environment: 'production',
    deploy: true
  }
});

// Get workflow runs
const runs = await mcp.github.getWorkflowRuns({
  owner: 'magicmcp',
  repo: 'example-repo',
  workflow_id: 'ci.yml',
  status: 'completed'
});
```

### Search Operations

```javascript
// Search repositories
const repos = await mcp.github.searchRepos({
  q: 'language:typescript mcp-server',
  sort: 'stars',
  order: 'desc'
});

// Search code
const code = await mcp.github.searchCode({
  q: 'extension:ts repo:magicmcp/github-mcp TODO',
  per_page: 50
});

// Search issues
const issues = await mcp.github.searchIssues({
  q: 'is:open is:issue label:bug org:magicmcp',
  sort: 'created',
  order: 'desc'
});
```

## Rate Limiting

The GitHub MCP Server handles rate limiting automatically:

- **Pause Strategy**: Waits when rate limit is reached
- **Backoff Strategy**: Exponential backoff on rate limit errors
- **Proactive Strategy**: Slows down requests as limit approaches

Monitor rate limits:

```javascript
const limits = await mcp.github.getRateLimit();
console.log('Remaining requests:', limits.rate.remaining);
console.log('Reset time:', new Date(limits.rate.reset * 1000));
```

## Webhooks

Set up webhooks to receive GitHub events:

```javascript
// Create a webhook
const webhook = await mcp.github.createWebhook({
  owner: 'magicmcp',
  repo: 'example-repo',
  config: {
    url: 'https://your-server.com/webhooks/github',
    content_type: 'json',
    secret: 'your-webhook-secret'
  },
  events: ['push', 'pull_request', 'issues']
});
```

## Error Handling

Common error scenarios:

- `401 Unauthorized`: Invalid or missing token
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Repository or resource not found
- `422 Unprocessable Entity`: Invalid request parameters
- `429 Too Many Requests`: Rate limit exceeded

## Best Practices

1. **Use Fine-grained Tokens**: Only request necessary permissions
2. **Cache Responses**: Reduce API calls for frequently accessed data
3. **Batch Operations**: Use GraphQL for complex queries
4. **Monitor Rate Limits**: Implement proper rate limit handling
5. **Use Webhooks**: For real-time updates instead of polling

## License

MIT License - see [LICENSE](https://github.com/magicmcp/github-mcp/blob/main/LICENSE) for details.