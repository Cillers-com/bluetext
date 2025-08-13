# Documentation For The PostgreSQL Blueprint

This Blueprint specifies how to add and update a PostgreSQL module for providing a PostgreSQL database server as well as how to access the PostgreSQL database.

Parent module: `polytope/postgres`. Read the bluetext documentation resource for this standard module to understand how to specify the `args` attributes.

## Example of a Polytope PostgreSQL Module

```yaml
  - id: postgres
    info: "Runs the PostgreSQL database server (blueprint: postgres)"
    module: polytope/postgres
    args:
      image: postgres:16.2
      data-volume: { id: postgres-data, type: volume, scope: project }
      env:
        - { name: POSTGRES_HOST_AUTH_METHOD, value: trust }
        - { name: POSTGRES_DB, value: postgres }
      restart: { policy: on-failure }
      service-id: postgres
```

**IMPORTANT: Always create a wrapper module for PostgreSQL with a persistent volume!** Without a volume, all data will be lost when the container restarts.

## Accessing PostgreSQL

### Connection Parameters
PostgreSQL runs on port 5432 by default. Use the service name as the hostname when connecting from other containers in the same stack.

### Connection Testing
Test the database connection using standard PostgreSQL tools:
- **psql**: `psql -h postgres -U postgres -d postgres`
- **Python**: Use `psycopg2` or `asyncpg` libraries
- **Node.js**: Use `pg` library

## Environment Variables

Common PostgreSQL environment variables for configuration:

- **POSTGRES_DB**: Default database name to create
- **POSTGRES_USER**: Default user name (if not specified, defaults to 'postgres')
- **POSTGRES_PASSWORD**: Password for the default user
- **POSTGRES_HOST_AUTH_METHOD**: Authentication method (`trust`, `md5`, `scram-sha-256`)
- **POSTGRES_INITDB_ARGS**: Additional arguments for database initialization
- **POSTGRES_INITDB_WALDIR**: Custom location for write-ahead log

### Security Considerations
- **Never use `trust` authentication in production environments**
- **Always set strong passwords using secrets**
- **Consider using `scram-sha-256` for password authentication**

Example with proper authentication:
```yaml
env:
  - { name: POSTGRES_DB, value: myapp }
  - { name: POSTGRES_USER, value: appuser }
  - { name: POSTGRES_PASSWORD, value: pt.secret postgres-password }
  - { name: POSTGRES_HOST_AUTH_METHOD, value: scram-sha-256 }
```

## Initialization Scripts

PostgreSQL supports running SQL scripts during database initialization. Place SQL files in the container's `/docker-entrypoint-initdb.d/` directory:

```yaml
args:
  scripts:
    - type: host
      path: ./sql/init.sql
    - type: host
      path: ./sql/schema.sql
```

Scripts are executed in alphabetical order during the first container startup.

## Values And Secrets

The following values should be defined: `postgres-host`, `postgres-port`, and `postgres-database`. The following secrets should be defined: `postgres-username` and `postgres-password`. Do not hardcode these values in `polytope.yml`.

Example values:
```bash
pt values set postgres-host postgres
pt values set postgres-port 5432
pt values set postgres-database myapp
pt secrets set postgres-username appuser
pt secrets set postgres-password your-secure-password
```

## Database Management

### Creating Databases and Users
Use initialization scripts or connect to the running container to create additional databases and users:

```sql
-- Create a new database
CREATE DATABASE myapp;

-- Create a new user
CREATE USER appuser WITH PASSWORD 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE myapp TO appuser;
```

### Backup and Restore
- **Backup**: `pg_dump -h postgres -U postgres myapp > backup.sql`
- **Restore**: `psql -h postgres -U postgres myapp < backup.sql`

### Performance Tuning
Consider adjusting PostgreSQL configuration for your workload:
- `shared_buffers`: Memory for caching data
- `work_mem`: Memory for query operations
- `max_connections`: Maximum concurrent connections
- `checkpoint_segments`: Write-ahead log segments

## Common Connection Patterns

### Python (psycopg2)
```python
import psycopg2

conn = psycopg2.connect(
    host="postgres",
    port=5432,
    database="myapp",
    user="appuser",
    password="secure_password"
)
```

### Python (asyncpg)
```python
import asyncpg

conn = await asyncpg.connect(
    host="postgres",
    port=5432,
    database="myapp",
    user="appuser",
    password="secure_password"
)
```

### Node.js (pg)
```javascript
const { Client } = require('pg');

const client = new Client({
  host: 'postgres',
  port: 5432,
  database: 'myapp',
  user: 'appuser',
  password: 'secure_password'
});

await client.connect();
```

## Monitoring and Health Checks

PostgreSQL provides several ways to monitor database health:
- **pg_stat_activity**: Current database activity
- **pg_stat_database**: Database-level statistics
- **pg_stat_user_tables**: Table-level statistics

Example health check query:
```sql
SELECT 1; -- Simple connectivity test
SELECT version(); -- PostgreSQL version info
