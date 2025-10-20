#  Plug & Play Real-Time Chat Server

A **production-grade, plug-and-play real-time chat backend** built with **Go**, **Gorilla WebSocket**, and **Redis Pub/Sub** — ready to integrate into any application stack.
This service provides **real-time bidirectional messaging**, **horizontal scalability**, **optional JWT authentication**, and **PostgreSQL persistence** — all out of the box.

---

##  Overview

The **Chat Server** is designed as a **modular, independent microservice** that can plug seamlessly into any system needing live, instant communication — whether a web app, mobile app, or IoT dashboard.

It handles:

* Persistent **WebSocket connections**
* **Concurrent message broadcasting**
* **Cluster-wide synchronization** through Redis Pub/Sub
* **Secure authentication** using JWT
* **Optional message storage** in PostgreSQL

All components are isolated, configurable, and can run independently — making it **deployable in seconds**.

---

##  Plug & Play Architecture

### 🔹 Key Design Principle

> “Drop it in, configure environment variables, and start chatting.”

### 🔹 Core Components

| Component                         | Purpose                                                                                                   |
| --------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Hub**                           | Central orchestrator that manages all active WebSocket clients and broadcasts messages using Go channels. |
| **Client**                        | Represents each WebSocket connection, running independent read/write goroutines.                          |
| **Redis Broker**                  | Enables real-time message delivery across distributed instances using Pub/Sub.                            |
| **JWT Auth Module (Optional)**    | Validates users and authorizes their WebSocket connections.                                               |
| **PostgreSQL Storage (Optional)** | Persists user sessions, chat history, and message metadata.                                               |
| **Config Layer**                  | Loads environment variables for flexible deployment without code changes.                                 |
| **Logger**                        | Structured JSON logs for tracing, debugging, and observability.                                           |

---

##  Functional Requirements

1. **Connection Management**

   * Accepts persistent WebSocket connections.
   * Registers, tracks, and disconnects clients safely.
   * Manages connection lifecycle automatically.

2. **Message Handling**

   * Accepts JSON messages and routes them through the Hub.
   * Broadcasts messages to all or specific groups.
   * Uses Redis Pub/Sub for inter-node message propagation.

3. **Concurrency & Safety**

   * Each connection runs on lightweight goroutines.
   * Go channels manage all internal communication.
   * No manual locking required — concurrency-safe by design.

4. **Authentication (Optional)**

   * Validates JWTs before WebSocket handshake.
   * Associates user claims with active sessions.
   * Supports external authentication providers (via JWT issuer).

5. **Persistence (Optional)**

   * Stores messages and users in PostgreSQL.
   * Fetches chat history on reconnect.
   * Provides audit logs for compliance.

6. **Configuration**

   * Controlled entirely via environment variables.
   * No manual setup beyond configuring `.env` or system variables.

---

##  Non-Functional Requirements

| Category            | Description                                                            |
| ------------------- | ---------------------------------------------------------------------- |
| **Performance**     | Low latency (<50ms) message propagation even under high concurrency.   |
| **Scalability**     | Horizontally scalable through Redis Pub/Sub across multiple instances. |
| **Reliability**     | Graceful error handling and recovery for network failures.             |
| **Security**        | JWT validation, secure configuration, and controlled access.           |
| **Maintainability** | Modular architecture with clear package boundaries.                    |
| **Portability**     | Works with Docker, Kubernetes, or bare metal setups.                   |
| **Observability**   | Structured logging and metrics integration ready.                      |

---

##  Base Logic Flow

### **1️ Connection Flow**

1. A client connects via WebSocket endpoint (`/ws`).
2. (Optional) The server validates the JWT in the query or header.
3. The connection is upgraded and registered in the Hub.
4. Two goroutines start for each client:

   * **Reader** → Reads messages from the client.
   * **Writer** → Sends messages to the client.

---

### **2️ Message Flow**

1. The client sends a JSON message → `{ "type": "chat", "body": "Hello" }`.
2. The Reader passes it to the **Hub’s broadcast channel**.
3. The Hub sends it to all connected clients.
4. The message is published to **Redis Pub/Sub**.
5. Other server instances receive the message and rebroadcast locally.
6. (Optional) The message is persisted to PostgreSQL.

---

### **3️ Redis Pub/Sub Flow**

* **Publish:** Every outgoing message is published to Redis.
* **Subscribe:** Each instance listens for messages on the Redis channel.
* **Effect:** Global synchronization — every instance stays in sync without shared memory.

---

### **4️ Persistence Flow (Optional)**

* When a message is received, it’s stored in PostgreSQL.
* On reconnect, the client can request the recent message history.
* This enables durability and continuity across sessions.

---

### **5️ Authentication Flow (Optional)**

1. Clients connect using a token (`/ws?token=<JWT>`).
2. The server validates the token using a shared secret.
3. If valid → connection is accepted.
4. If invalid → connection is denied with an authentication error.

---

##  Configuration (Plug & Play Setup)

### Environment Variables

```
APP_ENV=production
PORT=8080

# Redis
REDIS_URL=redis://localhost:6379

# JWT (Optional)
JWT_SECRET=your-secure-secret
JWT_ISSUER=chat-server

# PostgreSQL (Optional)
POSTGRES_DSN=postgres://user:password@db:5432/chatdb
ENABLE_PERSISTENCE=true
```

>  **No code changes required.**
> Configure, run, and start using the chat server instantly.

---

##  Plug & Play Integration Scenarios

| Scenario                | How It Fits                                                                   |
| ----------------------- | ----------------------------------------------------------------------------- |
| **Web App Chat**        | Connect your frontend (React, Vue, etc.) to `/ws` — start chatting instantly. |
| **Mobile App Backend**  | Native or hybrid apps connect via WebSocket using JWT tokens.                 |
| **Notification System** | Broadcast system events or updates to all connected clients.                  |
| **Collaborative Tools** | Real-time document or dashboard sync using the same channel model.            |

---

##  Deployment

###  Using Docker

```bash
docker run -d \
  -p 8080:8080 \
  -e REDIS_URL=redis://host.docker.internal:6379 \
  -e JWT_SECRET=mysecret \
  yourorg/chat-server:latest
```

###  Using Kubernetes

Deploy as a stateless service with Redis as a shared Pub/Sub backend.

* Each pod represents one chat server instance.
* Redis acts as the message bus.
* Load Balancer distributes WebSocket connections.

---

##  System Design Diagram

```
                ┌────────────────────────┐
                │  WebSocket Clients     │
                │  (Browser / Mobile)    │
                └──────────┬─────────────┘
                           │
                           ▼
                 ┌──────────────────────┐
                 │   Chat Server (Go)   │
                 │ ──────────────────── │
                 │  - Hub               │
                 │  - Clients (WS)      │
                 │  - JWT Auth (opt)    │
                 │  - Storage (opt)     │
                 └─────────┬────────────┘
                           │
                  Redis Pub/Sub Channel
                           │
                           ▼
                 ┌──────────────────────┐
                 │  Other Chat Servers  │
                 │  (Distributed Nodes) │
                 └─────────┬────────────┘
                           │
                           ▼
                 ┌──────────────────────┐
                 │  PostgreSQL (opt)    │
                 │  Message Persistence │
                 └──────────────────────┘
```

---

##  Production Highlights

* **Plug-and-Play Deployment:** Run instantly with environment configuration only.
* **Scalable & Distributed:** Works seamlessly across multiple servers.
* **Secure:** JWT-based authentication and encrypted secrets.
* **Durable:** Optional persistence for message history and audit.
* **Observable:** Structured JSON logs for real-time insight.
* **Portable:** Fully containerized for Docker/Kubernetes.
* **Stateless Core:** Easy scaling without shared memory.
