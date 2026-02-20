# Setup Guide

This guide will walk you through setting up DBHub MCP Server with your SQL Server database.

## Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Configure Your Database

#### Option A: Create `dbhub.toml` (Recommended for TOML Configuration)

Copy the example configuration and customize it for your environment:

```bash
cp dbhub.toml.example dbhub.toml
```

Edit `dbhub.toml` with your database credentials:

```toml
[[sources]]
id = "your_db"
description = "Your Database"
type = "sqlserver"
host = "your-db-host.example.com"        # Your SQL Server hostname or IP
port = 1433                               # SQL Server port (default: 1433)
database = "YourDatabaseName"             # Database name
user = "sa"                               # Database username
password = "your-secure-password"         # Database password
sslmode = "require"                       # Use "require" for encrypted connection
connection_timeout = 15
query_timeout = 15
```

#### Option B: Use Environment Variables (.env)

If your database password contains special characters (@, :, /, #), use the .env file approach:

```bash
cp dbhub/.env.example dbhub/.env
```

Edit `dbhub/.env`:

```env
DB_TYPE=sqlserver
DB_HOST=your-db-host.example.com
DB_PORT=1433
DB_USER=sa
DB_PASSWORD=your-secure-password
DB_NAME=YourDatabaseName
TRANSPORT=stdio
PORT=8080
```

### 4. (Optional) Generate SSL Certificates for HTTP Transport

If you plan to use HTTP transport, generate self-signed certificates:

```bash
mkdir -p dbhub/certs
openssl req -x509 -newkey rsa:4096 -keyout dbhub/certs/key.pem -out dbhub/certs/cert.pem -days 365 -nodes
```

### 5. Start the Development Server

```bash
pnpm run dev
```

The server will start with your configured database connection.

## Configuration Options

### Database Connection Methods

#### TOML Configuration (Recommended)

Create a `dbhub.toml` file in the project root:

```toml
[[sources]]
id = "my_database"             # Unique identifier for this database
description = "My Database"    # Human-readable description
type = "sqlserver"             # Database type
host = "localhost"             # Database host
port = 1433                    # Database port
database = "mydb"              # Database name
user = "sa"                    # Username
password = "password"          # Password
sslmode = "require"            # "require" for encryption, "disable" for no encryption
```

#### DSN (Connection String)

Alternatively, use a DSN in your `.env` file:

```env
DSN=sqlserver://user:password@localhost:1433/mydb
```

> **Note**: If your password contains special characters (@, :, /, #), use the individual parameter method instead.

### SSH Tunnel Configuration (for Remote Databases)

If your database is behind a bastion host, add SSH configuration:

```toml
[[sources]]
id = "prod_database"
type = "sqlserver"
host = "10.0.1.100"           # Internal network host
port = 1433
database = "mydb"
user = "sa"
password = "password"
ssh_host = "bastion.example.com"
ssh_port = 22
ssh_user = "deploy_user"
ssh_key = "~/.ssh/id_rsa"
# ssh_passphrase = "key_password"  # Only if your key is encrypted
```

### Tool Configuration

Configure which tools are available and their restrictions:

```toml
[[tools]]
name = "execute_sql"
source = "my_database"
readonly = true                # Prevent exec of INSERT/UPDATE/DELETE
max_rows = 1000                # Maximum rows to return per query
```

## Security Considerations

### Credentials Management

- **Never commit** `dbhub.toml` or `.env` files to version control
- Use `.gitignore` to prevent accidental commits
- For production, consider using secrets management systems (AWS Secrets Manager, HashiCorp Vault, etc.)

### SSL/TLS Certificates

- Generated certificates for HTTP transport are self-signed
- For production, use certificates from a trusted CA
- Never commit certificate files to version control
- Keep private keys safe and never share them

### Database User Permissions

For the MCP server, consider creating a dedicated database user with limited permissions:

```sql
-- Example: Create read-only user for SQL Server
CREATE LOGIN mcp_server WITH PASSWORD = 'StrongPassword123!';
CREATE USER mcp_server FOR LOGIN mcp_server;

-- Grant only SELECT permissions
EXEC sp_addrolemember 'db_datareader', 'mcp_server';
```

## Environment-Specific Configurations

You can run the server with different configuration files:

```bash
# Use default dbhub.toml
pnpm run dev

# Use a specific configuration file
pnpm run dev -- --config=./configs/production.toml
```

Create different configuration files for different environments:
- `dbhub.development.toml` - Local development
- `dbhub.staging.toml` - Staging environment
- `dbhub.production.toml` - Production environment

## Troubleshooting

### Connection Timeout

If you get connection timeouts:
1. Verify the database host and port are correct
2. Check if your firewall allows connections to the database
3. Increase `connection_timeout` value in configuration
4. If using SSH tunnel, verify SSH credentials and key permissions

### Special Characters in Password

If your password contains special characters (@, :, /, #), use the environment variable method (Option B) instead of TOML DSN format.

### SSL/TLS Errors

If you get SSL/TLS errors:
1. Ensure `sslmode = "require"` is set if your database requires encryption
2. For self-signed certificates in development, set `sslmode = "disable"` (not recommended for production)

## Further Documentation

- [Configuration Reference](./docs/config/toml.mdx)
- [Installation Guide](./docs/installation.mdx)
- [Quick Start](./docs/quickstart.mdx)
- [Custom Tools](./docs/tools/custom-tools.mdx)

