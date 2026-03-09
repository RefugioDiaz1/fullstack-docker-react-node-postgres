# 🐳 Fullstack Dockerized — React + Node.js + PostgreSQL

> Fullstack application containerized with Docker, featuring a React TypeScript frontend, Node.js REST API with Express and Sequelize, and PostgreSQL database — all orchestrated with Docker Compose following production-grade security practices.

---

## 🧱 Tech Stack

### Frontend
- **React 18** with **TypeScript**
- **Vite** — build tool and dev server
- **Nginx** — serving the production build inside Docker

### Backend
- **Node.js** with **TypeScript**
- **Express 5** — REST API framework
- **Sequelize + sequelize-typescript** — ORM for PostgreSQL
- **Morgan** — HTTP request logger
- **CORS** — controlled cross-origin access
- **Swagger / OpenAPI** — auto-generated API documentation
- **dotenv** — environment variable management

### Database
- **PostgreSQL 16** (Alpine) — relational database

### DevOps / Infrastructure
- **Docker** — containerization
- **Docker Compose** — multi-container orchestration
- **Multi-stage builds** — optimized production images
- **Docker Networks** — service isolation by security layer
- **Bind Mounts** — persistent local data storage

---

## 📁 Project Structure

```
fullstack-docker-react-node-postgres/
├── client/                         # React + TypeScript frontend
│   ├── src/
│   ├── Dockerfile                  # Multi-stage: Vite build + Nginx
│   └── package.json
├── server/                         # Node.js + TypeScript backend
│   ├── src/
│   │   ├── config/
│   │   │   ├── db.ts               # Sequelize connection
│   │   │   └── swagger.ts          # Swagger config
│   │   ├── models/                 # Sequelize models
│   │   ├── router.ts               # API routes
│   │   └── server.ts               # Express app setup
│   ├── Dockerfile                  # Node build + tsc compile
│   └── package.json
├── volumeData/                     # PostgreSQL data (local bind mount)
├── docker-compose.yml              # Full orchestration
├── .env                            # Environment variables (not committed)
├── .env.example                    # Environment variable template
└── .gitignore
```

---

## 🔐 Architecture & Security

The project uses **two isolated Docker networks** to prevent unauthorized access between services:

```
Internet
    │
    ├── frontNode  (public-netFront)   → localhost:1254
    └── backNode   (public-netFront + private-netBD) → localhost:3005
                            │
                            └── BDPostgres (private-netBD) → localhost:5440
```

- The **frontend cannot reach the database directly** — enforced at the network layer by Docker
- The **backend acts as the only bridge** between the public network and the database
- PostgreSQL has **no external exposure** beyond what's needed for local development

---

## ⚙️ Environment Variables

Create a `.env` file in the project root based on `.env.example`:

```bash
# Backend
DATABASE_URL=postgresql://admin:admin123@postgres:5432/your_db_name
FRONTEND_URL=http://localhost:1254
PORT=3500

# PostgreSQL
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB=your_db_name
```

> ⚠️ Never commit your `.env` file. Only `.env.example` should be in version control.

---

## 🚀 Getting Started

### Prerequisites
- [Docker](https://www.docker.com/get-started) installed
- [Docker Compose](https://docs.docker.com/compose/) available

### 1. Clone the repository
```bash
git clone https://github.com/your-username/fullstack-docker-react-node-postgres.git
cd fullstack-docker-react-node-postgres
```

### 2. Set up environment variables
```bash
cp .env.example .env
# Edit .env with your values
```

### 3. Build and start all services
```bash
docker compose up -d --build
```

### 4. Access the application

| Service | URL |
|---|---|
| Frontend (React) | http://localhost:1254 |
| Backend (API) | http://localhost:3005 |
| API Docs (Swagger) | http://localhost:3005/docs |
| PostgreSQL | localhost:5440 |

---

## 🐳 Docker Details

### Services

| Service | Image | Port |
|---|---|---|
| `frontNode` | Custom (nginx:alpine) | 1254:80 |
| `backNode` | Custom (node:24-alpine) | 3005:3500 |
| `BDPostgres` | postgres:16-alpine | 5440:5432 |

### Useful commands

```bash
# Start all services
docker compose up -d

# Rebuild and start (after code changes)
docker compose up -d --build

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# View logs of a specific service
docker logs backNode
docker logs frontNode
docker logs BDPostgres

# Enter a container
docker exec -it backNode sh
docker exec -it BDPostgres sh
```

### Verify network isolation
```bash
# Enter the frontend container
docker exec -it frontNode sh

# This should FAIL (correct behavior — frontend cannot reach DB)
ping BDPostgres

# This should WORK (correct behavior — frontend can reach backend)
ping backNode
```

---

## 🛠️ Multi-stage Docker Builds

### Frontend (React + Vite → Nginx)
```
Stage 1 (builder): node:24-alpine
  └── npm install + npm run build → /app/dist

Stage 2 (production): nginx:alpine
  └── Copy /dist → serve with Nginx (~20MB final image)
```

### Backend (TypeScript → JavaScript)
```
Stage 1: node:24-alpine
  └── npm install + tsc → /app/dist/index.js
  └── node dist/index.js
```

---

## 📊 Database

- PostgreSQL runs in an isolated container with **persistent data** stored in `./volumeData/`
- Tables are created automatically on startup via **Sequelize `sync()`**
- The ORM handles schema creation based on TypeScript models with decorators

---

## 📖 API Documentation

Swagger UI is available at `http://localhost:3005/docs` when the backend is running.

---

## 🧪 Testing

```bash
# Run tests
docker exec -it backNode npm test

# Run with coverage
docker exec -it backNode npm run test:coverage
```

---

## 🔄 Switching Environments

To point the backend to a different database (e.g., cloud/staging), just update the `.env`:

```bash
# Local Docker
DATABASE_URL=postgresql://admin:admin123@postgres:5432/your_db

# Cloud (e.g., Render, Supabase)
DATABASE_URL=postgresql://user:password@your-cloud-host/your_db?ssl=true
```

Then rebuild:
```bash
docker compose up -d --build
```

No changes needed in `Dockerfile` or `docker-compose.yml`. ✅

---

## 📌 Key Concepts Applied

- **Multi-stage builds** — minimal production images
- **Docker Networks** — security isolation between layers
- **Bind Mounts** — persistent database data on local machine
- **Build-time vs Runtime variables** — `ARG` for Vite, `environment` for Node
- **Restart policies** — `unless-stopped` for development resilience
- **depends_on** — proper service startup order
- **`.dockerignore`** — prevents `.env` and `node_modules` from entering images

---

## 👤 Author

**Refugio Diaz**  
Full Stack Developer  
[GitHub](https://github.com/your-username) · [LinkedIn](https://linkedin.com/in/your-profile)

---

## 📄 License

MIT License — feel free to use this project as a template.
