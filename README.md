# DBHub MCP Server

A fully-featured Model Context Protocol (MCP) server providing HTTP transport for direct integration with modern AI systems. Enables seamless SQL database access across Claude, Cursor, VS Code, and all MCP-compatible AI clients.

## Overview

DBHub is a lightweight, dependency-free MCP server implementing the Model Context Protocol specification. It provides a standardized interface for AI models to interact with SQL databases—PostgreSQL, MySQL, SQL Server, MariaDB, and SQLite—through HTTP transport with optional stdio fallback.

## Features

- **Protocol Compliance**: Full MCP 2024-11-05 specification implementation
- **HTTP Transport**: Native HTTP/HTTPS support for direct AI model integration
- **Multi-Database**: Unified interface across PostgreSQL, MySQL, SQL Server, MariaDB, SQLite
- **Multi-Connection**: Configure and manage multiple database sources simultaneously
- **Security Controls**: Read-only enforcement, query timeouts, row limiting
- **SSH Tunneling**: Bastion host and multi-hop SSH support for remote database access
- **TLS/SSL Support**: End-to-end encryption for database and transport layers

## Installation

### Prerequisites

- Node.js 20+ (LTS recommended)
- pnpm (or npm/yarn)
- Database connectivity (network access to target database)

### Quick Start

```bash
# Clone and install
git clone <repository-url>
cd dbhub
pnpm install

# Configure your database
cp dbhub.toml.example dbhub.toml
# Edit dbhub.toml with your database credentials

# Start the server
pnpm dev
```

### Configuration

#### TOML Method (Recommended)

Create `dbhub.toml` in the project root:

```toml
[[sources]]
id = "primary_db"
type = "sqlserver"              # postgres, mysql, mariadb, sqlserver, sqlite
host = "your-db-host.example.com"
port = 1433
database = "your_database"
user = "db_username"
password = "secure_password"
sslmode = "require"             # require, disable
connection_timeout = 15
query_timeout = 30

[[tools]]
name = "execute_sql"
source = "primary_db"
readonly = true                 # Enforce read-only operations
max_rows = 1000                 # Result set limit
```

#### Environment Variables Method

Set in `.env` or system environment:

```env
DB_TYPE=sqlserver
DB_HOST=your-db-host.example.com
DB_PORT=1433
DB_USER=db_username
DB_PASSWORD=secure_password
DB_NAME=your_database
TRANSPORT=http
PORT=8080
```

#### SSH Tunnel Support

For databases behind bastion hosts:

```toml
[[sources]]
id = "remote_db"
type = "sqlserver"
host = "10.0.1.100"             # Internal network host
port = 1433
database = "your_database"
user = "db_username"
password = "secure_password"
ssh_host = "bastion.example.com"
ssh_port = 22
ssh_user = "deploy_user"
ssh_key = "~/.ssh/id_rsa"
```

## Transport Modes

### HTTP (Default for Production)

```bash
pnpm start --transport http --port 8080
```

Server exposes:
- MCP endpoint: `http://localhost:8080/mcp`
- Web workbench: `http://localhost:8080`
- Health check: `http://localhost:8080/health`

### Stdio (For Local Integration)

```bash
pnpm start --transport stdio
```

Direct stdin/stdout MCP protocol for local client integration.

## Integration

### Claude Desktop / Cursor

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "dbhub": {
      "command": "node",
      "args": ["/path/to/dbhub/dist/index.js"],
      "env": {
        "DB_TYPE": "sqlserver",
        "DB_HOST": "your-db-host.example.com",
        "DB_USER": "db_username",
        "DB_PASSWORD": "secure_password",
        "DB_NAME": "your_database",
        "TRANSPORT": "stdio"
      }
    }
  }
}
```

### HTTP Client

```bash
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/execute_sql",
    "params": {
      "query": "SELECT * FROM your_table LIMIT 10"
    }
  }'
```

## MCP Tools

### execute_sql

Execute SQL queries with parameterization and transaction support.

- **Parameters**: `query` (string), `params` (optional array)
- **Safety**: Read-only enforcement, row limiting, timeout protection
- **Returns**: Result set with column metadata

### search_objects

Discover and explore database schema objects.

- **Parameters**: `query` (string, optional), `type` (table|column|index|procedure)
- **Returns**: Paginated object listing with descriptions

### Custom Tools

Define reusable parameterized tools in `dbhub.toml`:

```toml
[[tools]]
name = "get_user_orders"
source = "primary_db"
sql = "SELECT * FROM orders WHERE user_id = ?"
```

## Security

### Best Practices

1. **Credential Management**
   - Never commit actual credentials to version control
   - Use `.gitignore` for `dbhub.toml` and `.env` files
   - Leverage environment variables for CI/CD environments
   - Rotate database passwords regularly

2. **Database User Isolation**
   ```sql
   -- SQL Server example
   CREATE LOGIN mcp_user WITH PASSWORD = 'StrongPassword123!';
   CREATE USER mcp_user FOR LOGIN mcp_user;
   EXEC sp_addrolemember 'db_datareader', 'mcp_user';  -- Read-only
   ```

3. **Network Access**
   - Restrict database host access to MCP server IP
   - Use VPC/security groups for internal networks
   - Enable TLS/SSL for all remote connections

4. **API Key Authentication** (HTTP mode)
   ```env
   API_KEY=your-secret-key
   ```
   Clients provide via `Authorization: Bearer <key>` header

## Development

```bash
# Install dependencies
pnpm install

# Development mode (watch + hot reload)
pnpm dev

# Build for production
pnpm build

# Start production build
pnpm start --transport http --port 8080

# Run tests
pnpm test

# Lint
pnpm lint
```

## Environment Variables Reference

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `DB_TYPE` | string | No | Database type: postgres, mysql, mariadb, sqlserver, sqlite |
| `DB_HOST` | string | No | Database hostname or IP |
| `DB_PORT` | number | No | Database port (auto-detected if omitted) |
| `DB_USER` | string | No | Database username |
| `DB_PASSWORD` | string | No | Database password (use .env for special characters) |
| `DB_NAME` | string | No | Database name or SQLite file path |
| `DSN` | string | No | Full connection string (alternative to individual params) |
| `TRANSPORT` | string | No | `stdio` (default) or `http` |
| `PORT` | number | No | HTTP server port (default: 8080) |
| `SSH_HOST` | string | No | SSH bastion hostname |
| `SSH_USER` | string | No | SSH username |
| `SSH_KEY` | string | No | Path to SSH private key |
| `SSH_PASSPHRASE` | string | No | SSH key passphrase (if encrypted) |
| `API_KEY` | string | No | API key for HTTP authentication |
| `READONLY` | boolean | No | Enforce read-only SQL execution |

## Compatibility

DBHub is compatible with all MCP-compliant AI systems:

- Claude (Claude Desktop, Claude Code)
- Cursor IDE
- VS Code with MCP extension
- Any MCP-compatible application

## License

See LICENSE file for details.

## Support

For issues, feature requests, or contributions, see the repository.

