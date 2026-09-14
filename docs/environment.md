# Environment & Dependencies

This document describes all runtime versions, database options, development setup, Docker deployment, toolchain, and system-level dependencies required to build and run memos.

> **Version sources**: All version numbers listed below are verified against the repository's configuration files (`go.mod`, `web/package.json`, `scripts/Dockerfile`, `scripts/compose.yaml`, `proto/buf.yaml`, `proto/buf.gen.yaml`, `.golangci.yaml`). When updating dependencies, update this document accordingly.

## Runtime Requirements

| Component | Version | Source |
|-----------|---------|--------|
| Go | 1.27.0 | `go.mod` |
| Node.js | >= 24 | `web/package.json` (`engines.node`) |
| pnpm | 11.0.1 | `web/package.json` (`packageManager`) |

## Database

Memos supports three database drivers, selected via the `--driver` flag (or `MEMOS_DRIVER` environment variable). The default is **SQLite**, which requires no external database server.

| Driver | Flag value | Go module | Version | Notes |
|--------|-----------|-----------|---------|-------|
| SQLite | `sqlite` | `modernc.org/sqlite` | v1.56.0 | Default. Pure Go implementation, no CGO required. |
| MySQL | `mysql` | `github.com/go-sql-driver/mysql` | v1.10.0 | Production use. Requires external MySQL server. |
| PostgreSQL | `postgres` | `github.com/lib/pq` | v1.12.3 | Production use. Requires external PostgreSQL server. |

Driver selection logic is in `store/db/db.go` — the `NewDBDriver` function switches on `profile.Driver`.

### Database DSN configuration

- **SQLite**: `--data` flag or `MEMOS_DATA` env (data directory path; default `/var/opt/memos`)
- **MySQL**: `--dsn` flag or `MEMOS_DSN` env (e.g. `mysql://user:password@tcp(host:3306)/memos`)
- **PostgreSQL**: `--dsn` flag or `MEMOS_DSN` env (e.g. `postgresql://user:password@host:5432/memos?sslmode=disable`)

See `docs/configuration-provisioning.md` for full configuration reference.

## Local Development Setup

### Prerequisites

1. **Go 1.27.0** — [Download](https://go.dev/dl/) or use a version manager (`gvm`, `asdf`).
2. **Node.js >= 24** — [Download](https://nodejs.org/) or use `nvm`/`fnm`.
3. **pnpm 11.0.1** — Install via Corepack:
   ```bash
   corepack enable
   corepack prepare pnpm@11.0.1 --activate
   ```

### Build the frontend

The frontend must be built before the backend, as the Go binary embeds the compiled static assets.

```bash
cd web
pnpm install
pnpm release    # builds to ../server/router/frontend/dist
```

For active frontend development with hot reload:

```bash
cd web
pnpm install
pnpm dev        # starts Vite dev server
```

### Build and run the backend

```bash
go build -o memos ./cmd/memos
./memos
```

Or run directly:

```bash
go run ./cmd/memos
```

The server starts on port **5230** by default. Open `http://localhost:5230` in your browser.

### Run tests

**Backend:**

```bash
go test ./...
```

**Frontend:**

```bash
cd web
pnpm test
```

## Docker Deployment

### Quick start with Docker

```bash
docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  neosmemo/memos:stable
```

### Docker Compose

The repository includes a `scripts/compose.yaml` file:

```yaml
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    volumes:
      - ~/.memos/:/var/opt/memos
    ports:
      - 5230:5230
```

Run with:

```bash
docker compose -f scripts/compose.yaml up -d
```

### Building the Docker image locally

The `scripts/Dockerfile` uses a multi-stage build:

| Stage | Base image | Purpose |
|-------|-----------|---------|
| `backend` | `golang:1.27.0-alpine` | Compiles the Go binary with `CGO_ENABLED=0` (static linking) |
| `monolithic` | `alpine:3.21` | Minimal runtime image with `tzdata`, `ca-certificates`, `su-exec` |

Key build details:

- **Static binary**: `CGO_ENABLED=0`, build tags `netgo,osusergo`, `-ldflags="-s -w ... -extldflags '-static'"`
- **Non-root user**: Runs as `nonroot` (UID 10001) via `su-exec`
- **Port**: 5230 (`MEMOS_PORT` env, `EXPOSE 5230`)
- **Data volume**: `/var/opt/memos`
- **Timezone**: `UTC` by default (`TZ` env)

Build command:

```bash
docker build -f scripts/Dockerfile -t memos:local .
```

## Toolchain

### Protobuf — buf

Protobuf code generation is managed by [buf](https://buf.build) v2:

- Configuration: `proto/buf.yaml` (module + lint + breaking-change rules)
- Code generation: `proto/buf.gen.yaml` (remote plugins)

Generated outputs:

| Plugin | Output | Language |
|--------|--------|----------|
| `buf.build/protocolbuffers/go` | `proto/gen` | Go (protobuf) |
| `buf.build/grpc/go` | `proto/gen` | Go (gRPC) |
| `buf.build/connectrpc/go` | `proto/gen` | Go (ConnectRPC) |
| `buf.build/grpc-ecosystem/gateway` | `proto/gen` | Go (gRPC-Gateway) |
| `buf.build/community/google-gnostic-openapi` | `proto/gen` | OpenAPI spec |
| `buf.build/bufbuild/es:v2.12.1` | `web/src/types/proto` | TypeScript |

Install buf and generate code:

```bash
buf generate proto
```

### Go linting — golangci-lint

Configuration: `.golangci.yaml` (golangci-lint v2 format).

Enabled linters: `revive`, `govet`, `staticcheck`, `misspell`, `gocritic`, `sqlclosecheck`, `rowserrcheck`, `nilerr`, `godot`, `forbidigo`, `mirror`, `bodyclose`.

```bash
golangci-lint run
```

### Frontend linting — biome + tsc

- **biome** (^2.4.14): Configuration in `web/biome.json` — formatter + linter for JS/TS/CSS/HTML.
- **TypeScript** (^7.0.2): Type checking via `tsc --noEmit`.

```bash
cd web
pnpm lint          # tsc --noEmit + biome check
pnpm lint:fix      # biome auto-fix
pnpm format        # biome format
```

### Frontend build — Vite

- **Vite** ^8.0.11: Dev server and production bundler.
- **Vitest** ^4.1.5: Unit test framework.

## System Dependencies

### Build-time

- **No C compiler required**: The Go binary is compiled with `CGO_ENABLED=0`, using the pure-Go SQLite driver (`modernc.org/sqlite`). No system-level `gcc` or `libc-dev` is needed.
- **Git**: Required by Go modules and `apk add` in the Docker build stage.
- **`ca-certificates`**: Needed during build for HTTPS module downloads.

### Runtime

- **`tzdata`**: Timezone database (Alpine does not include it by default).
- **`ca-certificates`**: TLS certificate bundle for outbound HTTPS.
- **`su-exec`**: Privilege dropping helper (Alpine alternative to `gosu`).

### Operating system compatibility

- **Linux**: Primary target. The Docker image is based on Alpine 3.21.
- **macOS**: Supported for local development.
- **Windows**: Supported for local development (use WSL2 or native Go).

### Ports and volumes

| Item | Default | Configurable via |
|------|---------|-----------------|
| HTTP port | 5230 | `--port` flag / `MEMOS_PORT` env |
| Data directory | `/var/opt/memos` (Docker) or `./var` (local) | `--data` flag / `MEMOS_DATA` env |
