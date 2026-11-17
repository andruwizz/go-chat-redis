# Go Chat Redis

A simple real-time, channel-based chat application written in Go using [Gorilla WebSocket](https://github.com/gorilla/websocket) and [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/), with a lightweight browser frontend.

## Features

*   **Real-time messaging** over WebSocket (`/ws/{channel}`).
*   **Redis Pub/Sub broadcasting**: messages published to a channel are fanned out to all clients subscribed to that channel.
*   **Dynamic channel routing**: connect to any channel name via the URL path (the frontend uses `general` by default).
*   **Static frontend** served directly by the Go server (`./web`).
*   **Dockerized Redis** setup with AOF persistence and health checks via `docker-compose.yml`.

## Tech Stack

*   **Language**: [Go](https://go.dev/) (1.25+)
*   **WebSocket**: [gorilla/websocket](https://github.com/gorilla/websocket) v1.5.3
*   **Router**: [gorilla/mux](https://github.com/gorilla/mux) v1.8.1
*   **Pub/Sub**: [go-redis/redis/v8](https://github.com/go-redis/redis) v8.11.5 with [Redis](https://redis.io/) (`redis:7-alpine`)
*   **Frontend**: Plain HTML/JS (`web/index.html`, `web/main.js`)

## Architecture Overview

```
  Browser ──WebSocket──> handleWebSocket (/ws/{channel})
                              │                   ▲
                              ▼                   │
                         rdb.Publish         rdb.Subscribe
                              │                   │
                              └───>  Redis  ──────┘
                                    (Pub/Sub)
```

Each WebSocket connection subscribes to the requested Redis `{channel}` and spawns a goroutine to read incoming WebSocket messages and publish them to Redis. The handler's main loop reads messages from the Redis subscription channel and writes them back to the WebSocket client.

## Project Structure

```
├── main.go              # HTTP server, Gorilla Mux router, WebSocket & Redis Pub/Sub handler
├── docker-compose.yml   # Redis 7 container configuration
├── web/
│   ├── index.html       # Chat UI
│   └── main.js          # WebSocket client logic
├── go.mod
└── go.sum
```

## Prerequisites

- [Go](https://go.dev/dl/) 1.25+
- [Docker](https://www.docker.com/) & Docker Compose (or a local Redis server running on `localhost:6379`)

## Getting Started

### 1. Start Redis

```bash
docker compose up -d
```

### 2. Run

```bash
git clone <repo-url>
cd go-chat-redis
go run .
```

Open <http://localhost:8080> in one or more browser tabs, enter a username, and start chatting.

### Build

```bash
go build -o go-chat-redis .
./go-chat-redis
```

> The server serves `./web` relative to the working directory, so run it from the project root.

## WebSocket Protocol

Connect to:

```
ws://localhost:8080/ws/{channel}
```

For example, the bundled frontend connects to `ws://localhost:8080/ws/general`.

Messages are sent and received as plain text frames. The frontend formats outgoing messages as:

```text
<username>:<message>
```

Any text message sent by a client is published to the corresponding Redis channel and broadcast to every WebSocket client subscribed to that channel.

## Notes

*   `CheckOrigin` allows all origins for development; restrict it in production.
*   The Redis address is currently configured as `localhost:6379` in `main.go`.
*   Messages are delivered via Redis Pub/Sub and are not stored as chat history.

## Reference

This repository is a hands-on from this post  [Medium - Building a Real-Time Chat Application with Redis Pub/Sub, Golang, and WebSocket](https://senowijayanto.medium.com/building-a-real-time-chat-application-with-redis-pub-sub-golang-and-websocket-1484e1a4f67a)