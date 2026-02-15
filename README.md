# Boone Gifts

Gift list and wishlist platform. Users create gift lists, share them with connections, and claim gifts from shared lists.

## Sub-Projects

| Directory | Stack | URL |
|---|---|---|
| `boone-gifts-backend/` | Python 3.14, FastAPI, SQLAlchemy, MySQL | `https://boone-gifts-api.localhost` |
| `boone-gifts-frontend/` | React 19, TypeScript, Vite, Tailwind CSS | `https://boone-gifts.localhost` |

Both sub-projects are included as git submodules.

## Prerequisites

- Docker Desktop (with "Allow the default Docker socket to be used" enabled)
- [go-task](https://taskfile.dev/)
- [tbc-localdev-infra](https://github.com/tomboone/tbc-localdev-infra) cloned as a sibling directory (provides Traefik, MySQL, PostgreSQL, Mailpit)

## Setup

1. Clone the repo with submodules:

   ```
   git clone --recurse-submodules git@github.com:tomboone/boone-gifts.git
   ```

   If already cloned without submodules:

   ```
   git submodule update --init
   ```

2. Start shared infrastructure:

   ```
   cd ../tbc-localdev-infra
   task up
   ```

3. Copy environment templates and configure each sub-project:

   ```
   cp boone-gifts-backend/.env.example boone-gifts-backend/.env
   cp boone-gifts-frontend/.env.example boone-gifts-frontend/.env
   ```

4. Start both services:

   ```
   task api:up
   task web:up
   ```

5. Run backend database migrations and create the first admin user:

   ```
   task api:migrate
   task api:create-admin
   ```

## Task Commands

A root Taskfile proxies commands to each sub-project with a namespace prefix. Run `task --list` to see all available commands.

**Backend** (`api:`)
```
task api:up              # Build and start (runs uvicorn)
task api:logs            # Follow container logs
task api:restart         # Restart the container
task api:test            # Run test suite
task api:migrate         # Apply database migrations
```

**Frontend** (`web:`)
```
task web:up              # Build and start (runs Vite)
task web:logs            # Follow container logs
task web:restart         # Restart the container
task web:test            # Run test suite
```

**Infrastructure** (`infra:`) — requires [tbc-localdev-infra](https://github.com/tomboone/tbc-localdev-infra) cloned at `../tbc-localdev-infra`
```
task infra:up            # Start Traefik, MySQL, PostgreSQL, Mailpit
task infra:down          # Stop all services
task infra:status        # Show service status
task infra:logs          # Follow logs
```
