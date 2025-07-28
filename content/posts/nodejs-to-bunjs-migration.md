+++
title = "Replacing Node.js with Bun.js: The Complete Migration Guide for Modern JavaScript Development"
date = "2025-07-28T15:00:00+05:30"
author = ""
draft = true
authorTwitter = "" #do not include @
cover = ""
tags = ["bun", "nodejs", "javascript", "typescript", "performance", "migration", "web-development", "runtime"]
keywords = ["bun.js", "node.js replacement", "javascript runtime", "bun migration", "bun vs node", "javascript performance", "typescript runtime", "web development"]
description = "Complete guide to migrating from Node.js to Bun.js - the blazingly fast JavaScript runtime that's revolutionizing web development with superior performance and built-in tooling."
showFullContent = false
readingTime = false
hideComments = false
+++

JavaScript development is experiencing a renaissance with the emergence of Bun.js - a lightning-fast JavaScript runtime that's challenging Node.js's dominance. Built from the ground up with performance in mind, Bun offers dramatically faster execution, built-in bundling, testing, and package management, all while maintaining compatibility with the Node.js ecosystem.

## What is Bun.js?

Bun is an all-in-one JavaScript runtime and toolkit designed for speed. Written in Zig and powered by JavaScriptCore (Safari's JavaScript engine), Bun aims to be a drop-in replacement for Node.js while providing significantly better performance and developer experience.

Key features of Bun:
- **Blazing Fast Runtime**: 3-4x faster than Node.js in many scenarios
- **Built-in Bundler**: No need for Webpack, Rollup, or Parcel
- **Native TypeScript Support**: Run TypeScript files directly without compilation
- **Built-in Test Runner**: Jest-compatible testing without additional setup
- **Fast Package Manager**: npm-compatible but significantly faster
- **Web APIs**: Built-in support for fetch, WebSocket, and other web standards
- **Hot Reloading**: Built-in watch mode for development

## Why Replace Node.js with Bun?

### 1. **Unprecedented Performance**
Bun's performance improvements are not incremental - they're revolutionary:
- **Startup time**: 3-4x faster than Node.js
- **HTTP requests**: 2-3x faster server response times
- **File I/O**: Significantly faster file operations
- **Package installation**: 10-25x faster than npm

### 2. **Simplified Toolchain**
Instead of juggling multiple tools (Node.js + npm + Webpack + Jest + TypeScript), Bun provides everything in one binary:
- Runtime + Package Manager + Bundler + Test Runner + TypeScript compiler

### 3. **Better Developer Experience**
- Zero configuration for most common tasks
- Built-in hot reloading
- Native TypeScript support
- Modern JavaScript APIs built-in

### 4. **Ecosystem Compatibility**
Bun is designed to be compatible with the Node.js ecosystem, meaning most npm packages work without modification.

## Installation and Setup

### Installing Bun

```bash
# macOS and Linux (recommended)
curl -fsSL https://bun.sh/install | bash

# Using Homebrew (macOS)
brew tap oven-sh/bun
brew install bun

# Using npm (if you have Node.js)
npm install -g bun

# Windows (using PowerShell)
powershell -c "irm bun.sh/install.ps1 | iex"

# Verify installation
bun --version
```

### Setting up Shell Integration

```bash
# Add to your shell profile (.bashrc, .zshrc, etc.)
echo 'export BUN_INSTALL="$HOME/.bun"' >> ~/.bashrc
echo 'export PATH="$BUN_INSTALL/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Enable shell completions
bun completions
```

### Version Management with Bun

```bash
# Install specific Bun version
bun upgrade --to-version 1.0.25

# Check current version
bun --version

# Update to latest
bun upgrade
```

## Basic Bun Commands and Usage

### Package Management

```bash
# Initialize new project
bun init

# Install dependencies (reads package.json)
bun install

# Add packages
bun add express
bun add -d typescript @types/node  # dev dependencies
bun add -g nodemon                 # global packages

# Remove packages
bun remove express

# Update packages
bun update

# Run scripts from package.json
bun run dev
bun run build
bun run test

# Run files directly
bun index.js
bun app.ts  # TypeScript files run directly!
```

### Running JavaScript/TypeScript

```bash
# Run JavaScript
bun run server.js

# Run TypeScript directly (no compilation needed!)
bun run app.ts

# Run with watch mode (hot reloading)
bun --watch server.ts

# Run with environment variables
bun --env-file=.env.local run app.ts
```

## Migrating Existing Node.js Projects

### Step 1: Project Assessment

Before migrating, assess your project's compatibility:

```bash
# Check your current Node.js project
cd your-nodejs-project

# Review package.json dependencies
cat package.json

# Check for Node.js specific APIs that might need attention
grep -r "process\." src/
grep -r "__dirname\|__filename" src/
grep -r "require(" src/
```

### Step 2: Install Bun and Dependencies

```bash
# Install Bun in your project directory
bun install

# This reads your existing package.json and creates bun.lockb
# Bun is compatible with package.json, so no changes needed initially
```

### Step 3: Update Scripts in package.json

```json
{
  "name": "my-app",
  "scripts": {
    "dev": "bun --watch src/index.ts",
    "build": "bun build src/index.ts --outdir ./dist",
    "start": "bun src/index.ts",
    "test": "bun test",
    "lint": "bun run eslint src/"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  },
  "devDependencies": {
    "@types/express": "^4.17.17",
    "@types/cors": "^2.8.13",
    "typescript": "^5.0.0"
  }
}
```

### Step 4: Handle Node.js Specific Code

#### File System and Path Operations
```typescript
// Before (Node.js)
import * as fs from 'fs';
import * as path from 'path';

const __dirname = path.dirname(new URL(import.meta.url).pathname);
const data = fs.readFileSync(path.join(__dirname, 'data.json'), 'utf8');

// After (Bun - more modern approach)
import { file } from 'bun';

const data = await file('./data.json').text();
// Or using Bun's built-in file API
const data = await Bun.file('./data.json').text();
```

#### Environment Variables
```typescript
// Both Node.js and Bun (compatible)
const port = process.env.PORT || 3000;

// Bun also supports
const port = Bun.env.PORT || 3000;
```

#### HTTP Server
```typescript
// Before (Node.js with Express)
import express from 'express';
const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello World' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});

// After (Bun with built-in server - optional)
const server = Bun.serve({
  port: 3000,
  fetch(req) {
    const url = new URL(req.url);

    if (url.pathname === '/') {
      return new Response(JSON.stringify({ message: 'Hello World' }), {
        headers: { 'Content-Type': 'application/json' }
      });
    }

    return new Response('Not Found', { status: 404 });
  },
});

console.log(`Server running on http://localhost:${server.port}`);

// Or continue using Express (fully compatible)
import express from 'express';
const app = express();
// ... same Express code works with Bun
```

## Real-World Migration Examples

### Example 1: Express.js API Server

#### Original Node.js Setup
```typescript
// server.ts (Node.js version)
import express from 'express';
import cors from 'cors';
import { readFileSync } from 'fs';
import { join } from 'path';

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

// Load configuration
const config = JSON.parse(
  readFileSync(join(__dirname, 'config.json'), 'utf8')
);

app.get('/api/users', async (req, res) => {
  // Simulate database query
  const users = await getUsersFromDB();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const newUser = await createUser(req.body);
  res.status(201).json(newUser);
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### Migrated Bun Version (Option 1: Keep Express)
```typescript
// server.ts (Bun with Express - drop-in replacement)
import express from 'express';
import cors from 'cors';

const app = express();
const PORT = Bun.env.PORT || 3000;

app.use(cors());
app.use(express.json());

// Load configuration using Bun's file API
const config = await Bun.file('./config.json').json();

app.get('/api/users', async (req, res) => {
  const users = await getUsersFromDB();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const newUser = await createUser(req.body);
  res.status(201).json(newUser);
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Run with: bun --watch server.ts
```

#### Migrated Bun Version (Option 2: Native Bun Server)
```typescript
// server.ts (Native Bun server - better performance)
const config = await Bun.file('./config.json').json();

const server = Bun.serve({
  port: Bun.env.PORT || 3000,

  async fetch(req) {
    const url = new URL(req.url);

    // CORS headers
    const corsHeaders = {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    };

    if (req.method === 'OPTIONS') {
      return new Response(null, { headers: corsHeaders });
    }

    // Routes
    if (url.pathname === '/api/users' && req.method === 'GET') {
      const users = await getUsersFromDB();
      return new Response(JSON.stringify(users), {
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    if (url.pathname === '/api/users' && req.method === 'POST') {
      const body = await req.json();
      const newUser = await createUser(body);
      return new Response(JSON.stringify(newUser), {
        status: 201,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    return new Response('Not Found', {
      status: 404,
      headers: corsHeaders
    });
  },
});

console.log(`Server running on http://localhost:${server.port}`);
```

### Example 2: React Development Server

#### package.json for React with Bun
```json
{
  "name": "react-bun-app",
  "scripts": {
    "dev": "bun --watch src/index.tsx",
    "build": "bun build src/index.tsx --outdir ./dist --minify",
    "preview": "bun run build && bun serve dist/",
    "test": "bun test"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "typescript": "^5.0.0"
  }
}
```

#### Development Server Setup
```typescript
// dev-server.ts
const server = Bun.serve({
  port: 3000,

  async fetch(req) {
    const url = new URL(req.url);

    // Serve static files
    if (url.pathname.startsWith('/static/')) {
      const filePath = `.${url.pathname}`;
      const file = Bun.file(filePath);
      return new Response(file);
    }

    // Serve index.html for all routes (SPA)
    const indexFile = Bun.file('./public/index.html');
    return new Response(indexFile, {
      headers: { 'Content-Type': 'text/html' }
    });
  },
});

console.log(`React dev server running on http://localhost:${server.port}`);
```

### Example 3: Next.js-like Framework with Bun

```typescript
// framework.ts - Simple Next.js-like framework
import { join } from 'path';

interface Route {
  path: string;
  handler: (req: Request) => Response | Promise<Response>;
}

class BunFramework {
  private routes: Route[] = [];

  get(path: string, handler: (req: Request) => Response | Promise<Response>) {
    this.routes.push({ path, handler });
  }

  post(path: string, handler: (req: Request) => Response | Promise<Response>) {
    this.routes.push({ path, handler });
  }

  async serve(port: number = 3000) {
    const server = Bun.serve({
      port,
      async fetch(req) {
        const url = new URL(req.url);

        // Find matching route
        const route = this.routes.find(r => r.path === url.pathname);

        if (route) {
          return await route.handler(req);
        }

        // Serve static files
        if (url.pathname.startsWith('/public/')) {
          const filePath = `.${url.pathname}`;
          const file = Bun.file(filePath);
          if (await file.exists()) {
            return new Response(file);
          }
        }

        return new Response('Not Found', { status: 404 });
      },
    });

    console.log(`Server running on http://localhost:${server.port}`);
    return server;
  }
}

// Usage
const app = new BunFramework();

app.get('/', (req) => {
  return new Response(`
    <!DOCTYPE html>
    <html>
      <head><title>Bun Framework</title></head>
      <body>
        <h1>Welcome to Bun Framework!</h1>
        <p>Built with Bun.js</p>
      </body>
    </html>
  `, {
    headers: { 'Content-Type': 'text/html' }
  });
});

app.get('/api/hello', (req) => {
  return new Response(JSON.stringify({ message: 'Hello from Bun!' }), {
    headers: { 'Content-Type': 'application/json' }
  });
});

app.serve(3000);
```

## Built-in Tools and Features

### Bundling with Bun

```bash
# Bundle for production
bun build src/index.ts --outdir ./dist

# Bundle with minification
bun build src/index.ts --outdir ./dist --minify

# Bundle for different targets
bun build src/index.ts --outdir ./dist --target browser
bun build src/index.ts --outdir ./dist --target node

# Watch mode for development
bun build src/index.ts --outdir ./dist --watch
```

#### Advanced Bundling Configuration
```typescript
// build.ts - Custom build script
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  minify: true,
  sourcemap: 'external',
  target: 'browser',
  splitting: true,
  plugins: [
    // Custom plugins can be added here
  ],
});
```

### Testing with Bun

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}
```

```typescript
// math.test.ts
import { expect, test, describe } from 'bun:test';
import { add, multiply } from './math';

describe('Math functions', () => {
  test('add function', () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
  });

  test('multiply function', () => {
    expect(multiply(3, 4)).toBe(12);
    expect(multiply(0, 5)).toBe(0);
  });
});

// Run with: bun test
```

#### Advanced Testing Features
```typescript
// advanced.test.ts
import { expect, test, beforeAll, afterAll, mock } from 'bun:test';

// Mocking
const mockFetch = mock(() => Promise.resolve({
  json: () => Promise.resolve({ data: 'mocked' })
}));

beforeAll(() => {
  global.fetch = mockFetch;
});

afterAll(() => {
  mock.restore();
});

test('API call with mock', async () => {
  const response = await fetch('/api/data');
  const data = await response.json();

  expect(data).toEqual({ data: 'mocked' });
  expect(mockFetch).toHaveBeenCalledTimes(1);
});

// Snapshot testing
test('component snapshot', () => {
  const component = { name: 'Button', props: { color: 'blue' } };
  expect(component).toMatchSnapshot();
});
```

### Environment and Configuration

```typescript
// config.ts
export const config = {
  port: Bun.env.PORT || 3000,
  database: {
    url: Bun.env.DATABASE_URL || 'sqlite://./dev.db',
    maxConnections: parseInt(Bun.env.DB_MAX_CONNECTIONS || '10'),
  },
  jwt: {
    secret: Bun.env.JWT_SECRET || 'dev-secret',
    expiresIn: Bun.env.JWT_EXPIRES_IN || '24h',
  },
  redis: {
    url: Bun.env.REDIS_URL || 'redis://localhost:6379',
  },
};

// Load environment files
// .env.local, .env.development, .env.production
// Bun automatically loads these files
```

## Performance Comparisons and Benchmarks

### HTTP Server Performance

```typescript
// benchmark-server.ts
const server = Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response('Hello World!');
  },
});

// Benchmark results (approximate):
// Node.js (Express): ~15,000 req/sec
// Bun (native): ~45,000 req/sec
// Improvement: ~3x faster
```

### Package Installation Speed

```bash
# Time comparison for installing React + dependencies
# npm install: ~45 seconds
# yarn install: ~30 seconds
# pnpm install: ~20 seconds
# bun install: ~2 seconds

# Benchmark
time npm install    # 45.2s
time bun install    # 1.8s
# Result: ~25x faster
```

### File I/O Performance

```typescript
// file-io-benchmark.ts
import { readFileSync } from 'fs';

// Node.js approach
console.time('Node.js file read');
const nodeData = readFileSync('./large-file.json', 'utf8');
console.timeEnd('Node.js file read');

// Bun approach
console.time('Bun file read');
const bunData = await Bun.file('./large-file.json').text();
console.timeEnd('Bun file read');

// Typical results:
// Node.js file read: 45.2ms
// Bun file read: 12.8ms
// Improvement: ~3.5x faster
```

## Database Integration

### SQLite with Bun

```typescript
// database.ts
import { Database } from 'bun:sqlite';

const db = new Database('myapp.sqlite');

// Create tables
db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`);

// Prepared statements for better performance
const insertUser = db.prepare('INSERT INTO users (name, email) VALUES (?, ?)');
const getUserById = db.prepare('SELECT * FROM users WHERE id = ?');
const getAllUsers = db.prepare('SELECT * FROM users ORDER BY created_at DESC');

export class UserService {
  static create(name: string, email: string) {
    return insertUser.run(name, email);
  }

  static findById(id: number) {
    return getUserById.get(id);
  }

  static findAll() {
    return getAllUsers.all();
  }
}

// Usage
const newUser = UserService.create('John Doe', 'john@example.com');
const user = UserService.findById(newUser.lastInsertRowid);
const allUsers = UserService.findAll();
```

### PostgreSQL Integration

```typescript
// postgres.ts
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: Bun.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
});

export class PostgresService {
  static async query(text: string, params?: any[]) {
    const client = await pool.connect();
    try {
      const result = await client.query(text, params);
      return result;
    } finally {
      client.release();
    }
  }

  static async getUsers() {
    const result = await this.query('SELECT * FROM users ORDER BY created_at DESC');
    return result.rows;
  }

  static async createUser(name: string, email: string) {
    const result = await this.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [name, email]
    );
    return result.rows[0];
  }
}
```

## WebSocket and Real-time Features

```typescript
// websocket-server.ts
const server = Bun.serve({
  port: 3000,

  fetch(req, server) {
    const url = new URL(req.url);

    if (url.pathname === '/ws') {
      // Upgrade to WebSocket
      const success = server.upgrade(req, {
        data: { userId: url.searchParams.get('userId') }
      });

      if (success) {
        return undefined; // Upgrade successful
      }

      return new Response('WebSocket upgrade failed', { status: 400 });
    }

    // Serve static files or API routes
    return new Response('Hello World!');
  },

  websocket: {
    open(ws) {
      console.log('Client connected:', ws.data.userId);
      ws.subscribe('global');
    },

    message(ws, message) {
      console.log('Received:', message);

      // Broadcast to all clients
      ws.publish('global', `User ${ws.data.userId}: ${message}`);
    },

    close(ws) {
      console.log('Client disconnected:', ws.data.userId);
    },
  },
});

console.log(`WebSocket server running on http://localhost:${server.port}`);
```

### Real-time Chat Application

```typescript
// chat-server.ts
interface ChatMessage {
  id: string;
  userId: string;
  username: string;
  message: string;
  timestamp: number;
}

const messages: ChatMessage[] = [];
const users = new Map<string, { username: string; ws: any }>();

const server = Bun.serve({
  port: 3000,

  fetch(req, server) {
    const url = new URL(req.url);

    if (url.pathname === '/chat') {
      return new Response(await Bun.file('./chat.html').text(), {
        headers: { 'Content-Type': 'text/html' }
      });
    }

    if (url.pathname === '/ws') {
      const username = url.searchParams.get('username');
      if (!username) {
        return new Response('Username required', { status: 400 });
      }

      const success = server.upgrade(req, {
        data: { username, userId: crypto.randomUUID() }
      });

      return success ? undefined : new Response('Upgrade failed', { status: 400 });
    }

    return new Response('Not Found', { status: 404 });
  },

  websocket: {
    open(ws) {
      const { userId, username } = ws.data;
      users.set(userId, { username, ws });

      // Send recent messages to new user
      ws.send(JSON.stringify({
        type: 'history',
        messages: messages.slice(-50) // Last 50 messages
      }));

      // Notify others about new user
      ws.publish('chat', JSON.stringify({
        type: 'user_joined',
        username,
        timestamp: Date.now()
      }));

      ws.subscribe('chat');
    },

    message(ws, message) {
      const { userId, username } = ws.data;

      try {
        const data = JSON.parse(message);

        if (data.type === 'message') {
          const chatMessage: ChatMessage = {
            id: crypto.randomUUID(),
            userId,
            username,
            message: data.message,
            timestamp: Date.now()
          };

          messages.push(chatMessage);

          // Keep only last 1000 messages
          if (messages.length > 1000) {
            messages.splice(0, messages.length - 1000);
          }

          // Broadcast to all users
          ws.publish('chat', JSON.stringify({
            type: 'message',
            ...chatMessage
          }));
        }
      } catch (error) {
        console.error('Invalid message format:', error);
      }
    },

    close(ws) {
      const { userId, username } = ws.data;
      users.delete(userId);

      // Notify others about user leaving
      ws.publish('chat', JSON.stringify({
        type: 'user_left',
        username,
        timestamp: Date.now()
      }));
    },
  },
});

console.log(`Chat server running on http://localhost:${server.port}`);
```

## Production Deployment

### Docker Configuration

```dockerfile
# Dockerfile
FROM oven/bun:1.0-slim

WORKDIR /app

# Copy package files
COPY package.json bun.lockb ./

# Install dependencies
RUN bun install --frozen-lockfile --production

# Copy source code
COPY src/ ./src/
COPY public/ ./public/

# Build application
RUN bun build src/index.ts --outdir ./dist

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Run application
CMD ["bun", "dist/index.js"]
```

### Docker Compose for Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./src:/app/src
      - ./public:/app/public
    depends_on:
      - db
      - redis
    command: bun --watch src/index.ts

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Production Optimization

```typescript
// production-server.ts
const isProduction = Bun.env.NODE_ENV === 'production';

const server = Bun.serve({
  port: Bun.env.PORT || 3000,
  hostname: isProduction ? '0.0.0.0' : 'localhost',

  // Production optimizations
  development: !isProduction,

  async fetch(req) {
    // Security headers
    const securityHeaders = {
      'X-Content-Type-Options': 'nosniff',
      'X-Frame-Options': 'DENY',
      'X-XSS-Protection': '1; mode=block',
      'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
      'Content-Security-Policy': "default-src 'self'",
    };

    // GZIP compression for text responses
    const acceptEncoding = req.headers.get('accept-encoding') || '';
    const supportsGzip = acceptEncoding.includes('gzip');

    // Your application logic here
    const response = await handleRequest(req);

    // Add security headers
    Object.entries(securityHeaders).forEach(([key, value]) => {
      response.headers.set(key, value);
    });

    return response;
  },
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('Received SIGTERM, shutting down gracefully');
  server.stop();
  process.exit(0);
});

console.log(`Server running on http://localhost:${server.port}`);
```

## Troubleshooting Common Migration Issues

### Node.js Compatibility Issues

```typescript
// compatibility-fixes.ts

// 1. __dirname and __filename equivalents
import { dirname } from 'path';
import { fileURLToPath } from 'url';

// In Node.js modules
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// Bun alternative (simpler)
const currentDir = import.meta.dir;

// 2. Process.cwd() vs import.meta.dir
// Node.js
const configPath = path.join(process.cwd(), 'config.json');

// Bun (more reliable)
const configPath = path.join(import.meta.dir, 'config.json');

// 3. CommonJS require() in ESM
// Instead of require()
const config = await import('./config.json', { assert: { type: 'json' } });

// Or use Bun's file API
const config = await Bun.file('./config.json').json();
```

### Package Compatibility

```bash
# Check package compatibility
bun install --dry-run

# Force install problematic packages
bun install --force

# Use npm for specific packages if needed
npm install problematic-package
bun install  # Install rest with Bun
```

### Performance Debugging

```typescript
// performance-debug.ts
import { performance } from 'perf_hooks';

// Measure function performance
function measurePerformance<T>(fn: () => T, name: string): T {
  const start = performance.now();
  const result = fn();
  const end = performance.now();
  console.log(`${name} took ${end - start} milliseconds`);
  return result;
}

// Memory usage monitoring
function logMemoryUsage(label: string) {
  const usage = process.memoryUsage();
  console.log(`${label} - Memory usage:`, {
    rss: `${Math.round(usage.rss / 1024 / 1024)} MB`,
    heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)} MB`,
    heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)} MB`,
    external: `${Math.round(usage.external / 1024 / 1024)} MB`,
  });
}

// Usage
measurePerformance(() => {
  // Your code here
}, 'Database query');

logMemoryUsage('After processing');
```

## Best Practices and Recommendations

### Project Structure

```
my-bun-app/
├── src/
│   ├── controllers/
│   ├── services/
│   ├── models/
│   ├── middleware/
│   ├── utils/
│   └── index.ts
├── tests/
├── public/
├── dist/
├── package.json
├── bun.lockb
├── tsconfig.json
├── .env.example
└── README.md
```

### Configuration Management

```typescript
// config/index.ts
const config = {
  server: {
    port: parseInt(Bun.env.PORT || '3000'),
    host: Bun.env.HOST || 'localhost',
  },
  database: {
    url: Bun.env.DATABASE_URL || 'sqlite://./dev.db',
  },
  auth: {
    jwtSecret: Bun.env.JWT_SECRET || 'dev-secret',
    jwtExpiry: Bun.env.JWT_EXPIRY || '24h',
  },
  redis: {
    url: Bun.env.REDIS_URL || 'redis://localhost:6379',
  },
};

// Validate required environment variables
const requiredEnvVars = ['JWT_SECRET'];
if (Bun.env.NODE_ENV === 'production') {
  requiredEnvVars.push('DATABASE_URL');
}

for (const envVar of requiredEnvVars) {
  if (!Bun.env[envVar]) {
    throw new Error(`Required environment variable ${envVar} is not set`);
  }
}

export default config;
```

### Error Handling

```typescript
// error-handler.ts
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export function errorHandler(error: Error): Response {
  console.error('Error:', error);

  if (error instanceof AppError) {
    return new Response(JSON.stringify({
      error: error.message,
      status: error.statusCode
    }), {
      status: error.statusCode,
      headers: { 'Content-Type': 'application/json' }
    });
  }

  // Unknown error
  return new Response(JSON.stringify({
    error: 'Internal Server Error',
    status: 500
  }), {
    status: 500,
    headers: { 'Content-Type': 'application/json' }
  });
}

// Usage in server
const server = Bun.serve({
  async fetch(req) {
    try {
      return await handleRequest(req);
    } catch (error) {
      return errorHandler(error);
    }
  },
});
```

## Conclusion

Migrating from Node.js to Bun.js represents a significant leap forward in JavaScript runtime performance and developer experience. The benefits are compelling:

**Performance Gains:**
- 3-4x faster startup times
- 2-3x faster HTTP request handling
- 10-25x faster package installation
- Significantly improved file I/O performance

**Developer Experience:**
- Built-in TypeScript support without compilation
- Integrated bundler, test runner, and package manager
- Hot reloading out of the box
- Modern web APIs built-in

**Ecosystem Compatibility:**
- Drop-in replacement for most Node.js applications
- npm package compatibility
- Familiar APIs and patterns

**Migration Strategy:**
1. Start with new projects to gain familiarity
2. Migrate development environments first
3. Gradually migrate production workloads
4. Leverage Bun's built-in tools to simplify your toolchain

While Bun is still evolving, its performance advantages and modern approach make it an excellent choice for new projects and a worthy consideration for migrating existing Node.js applications. The JavaScript ecosystem is moving toward faster, more integrated tooling, and Bun is leading this transformation.

Whether you're building APIs, web applications, or command-line tools, Bun provides the performance and developer experience improvements that can significantly enhance your development workflow and application performance.