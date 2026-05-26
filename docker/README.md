# Local NDEx Docker Container

Run a local NDEx server for testing agent communities, analytics, and MCP tools.

## Prerequisites

- Docker Desktop installed and running.
- On **Apple Silicon (M-series)**: enable Rosetta in Docker Desktop →
  Settings → General → *"Use Rosetta for x86/amd64 emulation"*. The
  `ndexbio/ndex-rest` image is amd64-only and the launcher forces
  `--platform linux/amd64`; without Rosetta the container crashes
  early with `exec format error` and the 120s readiness probe times
  out with no obvious cause.
- **Port 8080 must be free.** Check before launching:
  `lsof -i :8080`. If anything responds, stop it first (a different
  NDEx, Tomcat, Jenkins, etc.). A *successful but wrong* bind is the
  worst case — the launcher's readiness probe will pass while
  `127.0.0.1:8080` is serving the wrong server.

## Quick Start

```bash
cd docker/
./monolithic_ndex.sh --config config.toml
```

This will:
1. Pull the `ndexbio/ndex-rest` image (first run only)
2. Start all services (NDEx, PostgreSQL, Keycloak, Solr, MailHog) in one container
3. Wait for the server to be ready (~30-40 seconds)
4. Seed test users defined in `config.toml`

The NDEx REST API is available at `http://localhost:8080`.

### Verify the launcher succeeded

```bash
# Server responds:
curl -s http://127.0.0.1:8080/v2/admin/status | head
# expected: a JSON status object, not an error

# Users seeded:
docker logs ndex 2>&1 | grep "user '"
# expected: one "user '<name>': created" or "user '<name>': exists" line
# per entry in config.toml
```

### Recovering from a stuck launch

If the launcher times out at 120s (`Container did not reach Ready
state within 120s`):

```bash
docker logs ndex 2>&1 | tail -40
```

- `Cannot connect to the Docker daemon` → Docker Desktop isn't
  running. Open it, wait for the whale icon to stop animating, retry.
- `exec format error` on Apple Silicon → enable Rosetta per
  Prerequisites and retry.
- Postgres init errors (schema/permission complaints) → bind-mount
  data from a prior partial run is incompatible. Stop and wipe:
  ```bash
  docker rm -f ndex
  rm -rf ndex-data ndex-config postgres-data postgres-config \
         keycloak-data keycloak-config solr-data solr-config mailhog-config
  ./monolithic_ndex.sh --config config.toml
  ```

## Test Users

Defined in `config.toml`:

| Username | Password |
|---|---|
| rdaneel | `RFB5azn3jvn4qxz@tbh` |
| janetexample | `RCavesofsteel#70` |
| drh | `Foundation#70` |

Edit `config.toml` to add or change users.

## Using MCP Tools with the Local Server

The ndex2 Python library hardcodes a path override when it sees "localhost" in the URL. **Use `127.0.0.1` instead of `localhost`** in your config.

Add profiles to `~/.ndex/config.json`:

```json
"local-rdaneel": {
  "server": "http://127.0.0.1:8080",
  "username": "rdaneel",
  "password": "RFB5azn3jvn4qxz@tbh"
}
```

Then use `--profile local-rdaneel` when starting the MCP server, or pass `profile="local-rdaneel"` in MCP tool calls.

## Container Management

```bash
# Stop the container
docker stop ndex

# Start it again (preserves data)
docker start ndex

# Remove and start fresh
docker rm -f ndex
./monolithic_ndex.sh --config config.toml

# View logs
docker logs ndex
docker logs -f ndex  # follow
```

## Data Persistence

Runtime data is stored in bind-mounted directories (`*-config/`, `*-data/`) created alongside this script. These are git-ignored. To reset to a clean state, stop the container and delete these directories — they'll be recreated on next launch.

## Verified Operations

The following have been tested against this container:
- REST API: status, auth, create network, set visibility, search, delete
- MCP tools: `get_connection_status`, `create_network`, `get_network_summary`, `search_networks`, `set_network_visibility`, `download_network`, `delete_network`
