# Checkbox Grid - Real-time Collaboration

A real-time collaborative checkbox grid application that synchronizes 500 checkboxes across multiple connected clients using WebSockets, Redis pub/sub, and JWT authentication.

Users open the page, toggle checkboxes, and see updates broadcast instantly to every connected client. The app uses Redis for distributed state management and supports horizontal scaling.

## Features

- ✨ **Real-time Synchronization** - 500 checkboxes sync instantly across all connected clients via Socket.io
- 🔐 **JWT Authentication** - Secure token-based authentication with JWKS validation
- 🎨 **Dark/Light Mode** - Beautiful glassmorphic UI with theme toggle (persisted)
- 📱 **Responsive Design** - Optimized for desktop and mobile
- 🚀 **Scalable Architecture** - Redis pub/sub for distributed state management
- 🔄 **Live Connection Counter** - Real-time user count display

## Tech Stack

- **Backend**: Node.js + Express.js
- **Real-time**: Socket.io with WebSocket support
- **State Management**: Redis/Valkey with pub/sub
- **Authentication**: JWT with RS256 algorithm (JWKS validation)
- **Frontend**: Vanilla HTML, CSS, and JavaScript
- **Containerization**: Docker Compose for easy setup

## Project Structure

```
.
├── index.js                    # Main server with Socket.io & authentication
├── redis-connection.js         # Redis pub/sub configuration
├── package.json               # Dependencies
├── docker-compose.yml         # Valkey/Redis service
├── README.md                  # This file
└── public/
    ├── index.html             # Main app with 500 checkboxes & theme toggle
    ├── login.html             # Login interface
    └── auth.html              # Authentication handler
```

## How It Works

1. **Authentication**: Users authenticate via JWT tokens validated against JWKS endpoints
2. **Connection**: Socket.io establishes WebSocket connections with JWT verification
3. **State Management**: Checkbox states are stored in Redis
4. **Synchronization**: 
   - When a checkbox changes, the client emits `client:checkbox:change` with the access token
   - Server validates the token and publishes update to Redis pub/sub
   - All subscribed instances receive the update and broadcast to connected clients
5. **Real-time Updates**: Connected clients receive instant updates via Socket.io events
6. **UI Experience**: Glassmorphic design with smooth animations and instant feedback

## Getting Started

### Prerequisites

- Node.js (v16 or higher)
- Docker & Docker Compose (for Redis/Valkey)

### 1. Install dependencies

```bash
npm install
```

### 2. Start Redis/Valkey

```bash
docker-compose up -d
```

### 3. Set environment variables (optional)

The app includes sensible defaults, but you can override:

```bash
# .env or export in your shell
AUTH_ORIGIN=http://localhost:8000
AUTH_CLIENT_ID=8f6763b7-bbe8-4c21-938d-e940a80264b1
AUTH_CLIENT_SECRET=146c4f36-6d5d-4ec4-948d-29d680ac5204
```

### 4. Run the server

**Development** (with auto-reload via nodemon):
```bash
npm run dev
```

**Production**:
```bash
npm start
```

The app will start on:

```
http://localhost:3000
```

## Socket.io Events

### Client → Server

| Event | Payload | Description |
|-------|---------|-------------|
| `client:checkbox:change` | `{index, checked, accessToken}` | Sent when a checkbox is toggled |

### Server → Client

| Event | Payload | Description |
|-------|---------|-------------|
| `server:checkbox:status` | `Array<boolean>` | Full checkbox state on connect |
| `server:checkbox:change` | `{index, checked}` | Broadcasts checkbox change |
| `server:checkbox:user` | `{user}` | Notification of who changed a checkbox |
| `server:error` | `{message, data}` | Error events (e.g., invalid token) |

## Infrastructure

The application uses Redis/Valkey for distributed state management and pub/sub, enabling:

- ✅ Multiple server instances
- ✅ Reliable state persistence
- ✅ Horizontal scaling
- ✅ Cross-instance real-time sync
- ✅ JWT authentication validation

## Features in Detail

### Authentication Flow

1. Users log in via the `/login` page
2. The authentication service validates credentials and issues a JWT token
3. Token is stored in browser localStorage as `accessToken`
4. Socket.io connections include the token for verification
5. Server validates token signature using JWKS from `AUTH_ORIGIN`
6. Invalid tokens result in `server:error` event

### Theme Persistence

- 🌙 Dark mode toggle button in top-right corner
- ☀️ Shows appropriate icon based on current theme
- Theme preference saved to localStorage as `theme`
- Loads automatically on page refresh
- Smooth transitions between modes

### Performance Optimizations

- JWKS tokens cached for 5 minutes (reduces HTTP requests)
- Redis pub/sub for efficient distributed updates
- CSS transitions for smooth UI interactions
- Backdrop filter blur for glassmorphic effects
- Responsive grid layout (auto-fit checkboxes)

## Known Limitations

- Requires valid JWT token from AUTH_ORIGIN for socket connections
- Checkbox state is ephemeral (cleared on service restart)
- Single authentication provider (AUTH_ORIGIN)

## Future Improvements

- [ ] Persist checkbox state to database for recovery
- [ ] Add comprehensive test suite (unit, integration, e2e)
- [ ] Implement rate limiting per user
- [ ] Add telemetry and monitoring
- [ ] Support multiple authentication providers
- [ ] Add WebSocket fallback for browsers without WebSocket support
- [ ] Implement checkbox history/audit logging
- [ ] Add user presence indicators
- [ ] Performance optimization for 1000+ checkboxes

## License

ISC
