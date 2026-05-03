# One Million Checkbox

A real-time collaborative checkbox application built with Node.js, Express, Socket.IO, and Redis/Valkey.

Users authenticate, open the grid, and every checkbox change is synced live across connected clients. State is persisted in Redis so the grid survives server restarts.

## Demo Video

- YouTube: https://youtu.be/M376gYk6fWQ

Replace the link above with your final demo video URL.

## Features

- Real-time checkbox updates with Socket.IO
- Persistent checkbox state in Redis hash
- Cross-instance sync using Redis pub/sub
- Auth code exchange flow with access token validation
- JWT verification via JWKS (RS256)
- Basic per-user rate limiting for checkbox writes

## Tech Stack

- Node.js (ES Modules)
- Express
- Socket.IO
- ioredis
- jsonwebtoken
- Valkey/Redis (Docker)

## Project Structure

```text
.
├── docker-compose.yml
├── index.js
├── package.json
├── redis-connection.js
└── public/
    ├── auth.html
    ├── index.html
    └── login.html
```

## Prerequisites

- Node.js 18+ (recommended)
- npm
- Docker (for Valkey/Redis)

## Environment Variables

The server supports these environment variables:

- `PORT` (default: `8000`)
- `AUTH_ORIGIN` (default in code: `http://localhost:8000`)
- `AUTH_CLIENT_ID`
- `AUTH_CLIENT_SECRET`
- `AUTH_REDIRECT_URI` (optional, fallback: `${req.protocol}://${host}/auth`)
- `REDIS_URL` (optional, overrides host/port)
- `REDIS_HOST` (default: `localhost`)
- `REDIS_PORT` (default: `6379`)

## Setup and Run

### 1. Install dependencies

```bash
npm install
```

### 2. Start Valkey/Redis with Docker

```bash
docker compose up -d
```

### 3. Start the app

Use correct shell syntax with no spaces around `=`.

```bash
PORT=5000 AUTH_CLIENT_ID=your-client-id AUTH_CLIENT_SECRET=your-client-secret npm run start
```

PowerShell equivalent:

```powershell
$env:PORT="5000"; $env:AUTH_CLIENT_ID="your-client-id"; $env:AUTH_CLIENT_SECRET="your-client-secret"; npm run start
```

### 4. Open in browser

- App: `http://localhost:5000`
- Health: `http://localhost:5000/health`

## Authentication Flow

1. Client opens `/` and loads `public/login.html`.
2. If no valid token exists, client redirects to `/login`.
3. Server redirects to auth provider at `AUTH_ORIGIN/api/auth/signin` with `client_id` and `redirect_uri`.
4. Auth provider redirects back to `/auth?code=...`.
5. `public/auth.html` posts code to `/auth/exchange`.
6. Server exchanges code at `AUTH_ORIGIN/api/auth/token` and returns access token.
7. Client stores token and opens `/home`.

## Real-Time Update Flow

1. On connect, server emits full checkbox state with `server:checkbox:status`.
2. Client emits `client:checkbox:change` with checkbox index, value, and token.
3. Server validates token (JWKS + RS256).
4. Server applies rate limit and saves state to Redis hash (`checkbox:state`).
5. Server emits local update and publishes event to Redis channel (`checkbox:change`).
6. Other instances consume pub/sub event and broadcast to their connected clients.

## Socket Events

### Client to Server

- `client:checkbox:change`
  - Payload:
    - `index: number`
    - `checked: boolean`
    - `accessToken: string`

### Server to Client

- `server:checkbox:status` (full boolean array)
- `server:checkbox:change` (single checkbox change)
- `server:checkbox:user` (user name who changed a checkbox)
- `server:error` (validation/rate-limit errors)

## API Routes

- `GET /` -> login page
- `GET /home` -> checkbox grid
- `GET /auth` -> auth callback page
- `GET /login` -> redirect to auth provider
- `POST /auth/exchange` -> code-to-token exchange
- `GET /health` -> health check JSON

## Development

```bash
npm run dev
```

## Notes and Improvements

- Move all auth secrets to environment variables only (avoid hardcoded fallback values in production).
- Use distributed rate limiting via Redis for multi-instance deployments.
- Add integration tests for auth flow and socket events.
- Add structured logging and monitoring.

## License

ISC
