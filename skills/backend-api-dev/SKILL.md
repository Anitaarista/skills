---
name: backend-api-dev
description: "Backend API development with multiple language and framework support. Build REST APIs, GraphQL servers, WebSocket services, microservices, CLI tools, and server-side applications using Express.js, FastAPI, Django, Flask, Go, Rust, or any backend technology. Use this skill whenever the user wants to create an API, server, backend service, REST endpoint, GraphQL schema, WebSocket server, cron job, scraper, CLI tool, database schema, authentication system, or any server-side code. Also triggers for requests involving server deployment, nginx configuration, database queries, or middleware development."
argument-hint: "Describe the API or backend service you want to build"
version: 1.0.0
---

# Backend API Development Skill

Build server-side applications and APIs with the framework and language of the user's choice.

## Core Philosophy

Backend development is about reliability, security, and performance. Whether it's a simple REST API or a complex microservice architecture, the fundamentals matter: clean API design, proper error handling, authentication, input validation, and good documentation.

## Environment

- **Node.js**: v24.14.1 with npm 11.11.1
- **Python**: 3.12.13 with pip
- **GCC**: Available for native modules (C/C++/Go/Rust compilation possible)
- **Working directory**: `/home/z/my-project/`
- **Persistent storage**: `/home/z/my-project/upload/` (OSS mount, survives restart)
- **Available ports**: Any port can be used; Caddy on port 81 proxies externally
- **AI SDK**: `z-ai-web-dev-sdk` installed — for AI-powered API endpoints
- **No Docker** — All services run directly in the environment

## Tech Stack Selection

| Framework | Language | Best For | Performance | DX |
|---|---|---|---|---|
| **FastAPI** (recommended) | Python | Rapid API dev, data apps, ML APIs | High | Excellent |
| **Express.js** | JavaScript | Quick prototyping, JSON APIs | Medium | Excellent |
| **Django + DRF** | Python | Full-featured apps with admin panel | Medium | Good |
| **Flask** | Python | Lightweight, microservices | Medium | Good |
| **Gin** | Go | High-performance APIs, microservices | Very High | Good |
| **Actix-web** | Rust | Maximum performance, systems-level | Highest | Medium |
| **NestJS** | TypeScript | Enterprise-grade, modular architecture | Medium | Excellent |

**Default recommendation**: FastAPI for new projects — it has auto-generated documentation, async support, type validation, and is the fastest Python framework. But always ask the user if they have a preference.

## Framework-Specific Guides

Read only the reference file for the chosen framework:
- `references/fastapi.md` — FastAPI with Python (recommended default)
- `references/express.md` — Express.js with Node.js
- `references/django.md` — Django + DRF
- `references/go-gin.md` — Go with Gin framework
- `references/rust-actix.md` — Rust with Actix-web

## API Design Principles (All Frameworks)

### RESTful API Design

- Use **plural nouns** for resources: `/users`, `/posts`, `/orders`
- Use **HTTP methods** correctly:
  - `GET` — Read (never modify data)
  - `POST` — Create
  - `PUT` — Full update
  - `PATCH` — Partial update
  - `DELETE` — Remove
- Use **proper status codes**:
  - `200` — Success
  - `201` — Created
  - `204` — No Content (successful delete)
  - `400` — Bad Request (validation error)
  - `401` — Unauthorized (not logged in)
  - `403` — Forbidden (no permission)
  - `404` — Not Found
  - `409` — Conflict (duplicate)
  - `422` — Unprocessable Entity
  - `429` — Too Many Requests
  - `500` — Internal Server Error
- Use **consistent error format**:
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Email is required",
      "details": [{ "field": "email", "message": "This field is required" }]
    }
  }
  ```
- Support **pagination**: `?page=1&limit=20`
- Support **filtering**: `?status=active&sort=created_at:desc`
- Version your API: `/api/v1/users`

### Authentication Patterns

- **JWT tokens** — Stateless, scalable, good for SPAs and mobile
- **API keys** — Simple, good for server-to-server
- **OAuth 2.0** — For third-party integrations
- **Session cookies** — For traditional web apps
- Always use HTTPS in production
- Hash passwords with bcrypt/argon2
- Never store plain text passwords

### Database Patterns

- Use **connection pooling**
- Always use **parameterized queries** (never string interpolation for SQL)
- Implement **migrations** for schema changes
- Use **transactions** for multi-step operations
- Add **indexes** for frequently queried fields
- Implement **soft deletes** where appropriate

## Common Backend Features

For detailed implementation patterns, read the framework-specific reference. Common features include:
- CRUD operations
- Authentication & authorization
- File upload/download
- Rate limiting
- Caching (Redis, in-memory)
- Background jobs/queues
- WebSocket real-time communication
- Scheduled tasks (cron)
- Email sending
- Webhook handling
- Health check endpoints
- Request logging & monitoring

## AI-Powered API Endpoints

Use `z-ai-web-dev-sdk` for AI features in backend APIs:

```javascript
import ZAI from 'z-ai-web-dev-sdk';
// Use in Express routes, FastAPI endpoints, etc.
```

**IMPORTANT**: Only use in server-side code, never expose the SDK to clients.

## Project Structure Template (Framework-Agnostic)

```
my-api/
├── src/
│   ├── main.ts              # Entry point
│   ├── routes/              # Route definitions
│   ├── controllers/         # Request handlers
│   ├── services/            # Business logic
│   ├── models/              # Data models
│   ├── middleware/           # Custom middleware
│   ├── utils/               # Utilities
│   ├── config/              # Configuration
│   └── types/               # Type definitions
├── tests/                   # Test files
├── .env                     # Environment variables
├── .env.example             # Template for env vars
└── README.md                # Setup & usage docs
```

## Development Workflow

1. **Set up project** — Initialize with the chosen framework's tooling
2. **Define models** — Data models/schemas first
3. **Create routes** — API endpoints
4. **Implement business logic** — Services layer
5. **Add middleware** — Auth, validation, error handling
6. **Write tests** — At minimum, test critical paths
7. **Document** — API docs, setup instructions
8. **Save to upload/** — Copy to persistent storage for download

## Important Rules

1. **Always validate input** — Never trust client data
2. **Use environment variables** — Never hardcode secrets
3. **Handle errors gracefully** — Consistent error format, proper status codes
4. **Add health check endpoint** — `GET /api/health` for monitoring
5. **Use CORS appropriately** — Don't use `*` in production
6. **Implement rate limiting** — Protect against abuse
7. **Log requests** — At minimum, log errors and important events
8. **Save to upload/** — Copy final project to `/home/z/my-project/upload/`
