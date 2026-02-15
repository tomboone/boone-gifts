# Boone Gifts

## Overview
Gift list and wishlist platform. Users create gift lists, share them with connections, and claim gifts from shared lists. Organized as two sub-projects in this directory.

## Sub-Projects

| Directory | Stack | URL |
|---|---|---|
| `boone-gifts-backend/` | Python 3.14, FastAPI, SQLAlchemy, MySQL | `https://boone-gifts-api.localhost` |
| `boone-gifts-frontend/` | React 19, TypeScript, Vite, Tailwind CSS | `https://boone-gifts.localhost` |

Each has its own git repo, Dockerfile, docker-compose.yml, Taskfile.yaml, and `.claude/CLAUDE.md` with sub-project-specific details.

## Shared Infrastructure

### Docker & Traefik
- Both services run on the external `proxy` Docker network
- Traefik runs separately from `/Users/trb74/Sites/traefik/docker-compose.yml`
- **Traefik v3.6.1+ is required** — Docker Engine 29 (Docker Desktop 4.52+) raised the minimum API version to 1.44, breaking older versions
- TLS is terminated at Traefik using its default self-signed certificate
- Docker Desktop must have "Allow the default Docker socket to be used" enabled
- If Traefik loses the socket connection after a Docker Desktop restart, do a full `docker compose down && up` on the Traefik stack

### MySQL
- Shared container `mysql_db` at `/Users/trb74/Sites/mysql/mysql/docker-compose.yml` on the `proxy` network
- Dev database: `boone_gifts`, Test database: `boone_gifts_test`

### Conventions
- Task commands have no namespace prefix (e.g., `task up`, `task test`)
- Both containers run their dev servers directly on `task up` — use `task logs` to follow output
- Source is bind-mounted in both containers for live development
- `docs/plans/` in each sub-project contains design and implementation documents (gitignored, local-only)

## Features

### Auth
- Invite-only registration — admin creates invites, invitees register with token
- JWT access tokens (30 min) + refresh tokens (7 days)
- Frontend stores tokens in memory only (XSS protection), Axios interceptors handle silent refresh

### Gift Lists
- Users create lists and add gifts with name, description, URL, price
- Lists are shared with specific connected users
- Shared users can claim gifts; claim info is hidden from the list owner

### User Connections
- Symmetric friend-style connections with request/accept flow
- Connections gate list sharing — must be connected to share a list
- Removing a connection revokes all shares, claims, and collection items between both users

### Collections
- Personal organizational layer — users group gift lists into named collections
- Can contain owned lists and lists shared by connections
- Owner-only access, automatic cleanup on unshare/disconnect

## API Endpoints

### Auth (`/auth`)
- `POST /auth/login` — Login with email and password
- `POST /auth/register` — Register with an invite token
- `POST /auth/refresh` — Refresh an access token

### Users (`/users`) — admin only
- `GET /users`, `GET /users/{id}`, `PUT /users/{id}`, `DELETE /users/{id}`

### Invites (`/invites`) — admin only
- `POST /invites`, `GET /invites`, `DELETE /invites/{id}`

### Connections (`/connections`)
- `POST /connections` — Send request (by user_id or email)
- `GET /connections` — List accepted connections
- `GET /connections/requests` — List pending incoming requests
- `POST /connections/{id}/accept` — Accept a request
- `DELETE /connections/{id}` — Remove/reject/cancel

### Lists (`/lists`)
- `POST /lists`, `GET /lists`, `GET /lists/{id}`, `PUT /lists/{id}`, `DELETE /lists/{id}`

### Gifts (`/lists/{list_id}/gifts`)
- `POST /lists/{id}/gifts`, `PUT /lists/{id}/gifts/{gift_id}`, `DELETE /lists/{id}/gifts/{gift_id}`
- `POST /lists/{id}/gifts/{gift_id}/claim`, `DELETE /lists/{id}/gifts/{gift_id}/claim`

### Shares (`/lists/{list_id}/shares`)
- `POST /lists/{id}/shares`, `GET /lists/{id}/shares`, `DELETE /lists/{id}/shares/{user_id}`

### Collections (`/collections`)
- `POST /collections`, `GET /collections`, `GET /collections/{id}`, `PUT /collections/{id}`, `DELETE /collections/{id}`
- `POST /collections/{id}/items`, `DELETE /collections/{id}/items/{list_id}`

## CORS
The backend's `APP_CORS_ORIGINS` env var must include the frontend URL (`https://boone-gifts.localhost`).
