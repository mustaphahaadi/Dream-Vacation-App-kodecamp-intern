# Dream Vacation App

> A full-stack containerised application that lets users build and manage their dream vacation destination list — now with automated CI/CD powered by GitHub Actions.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [Local Setup](#local-setup)
- [Docker Setup](#docker-setup)
- [CI/CD Pipeline](#cicd-pipeline)
- [Docker Architecture](#docker-architecture)
- [External API](#external-api)
- [Best Practices](#best-practices)

---

## Project Overview

The Dream Vacation App is a full-stack containerised application that allows users to:

- Add countries to a dream vacation list
- Retrieve country details (capital, population, region) via the REST Countries API
- Store and manage destinations in a PostgreSQL database
- Remove unwanted destinations

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + Nginx |
| Backend | Node.js 20 + Express |
| Database | PostgreSQL 16 |
| External API | REST Countries API |
| Containerisation | Docker + Docker Compose |
| CI/CD | GitHub Actions |

---

## Local Setup

### Backend

```bash
cd backend
npm install
# Configure your .env (see backend/.env.example)
npm start
```

### Frontend

```bash
cd frontend
npm install
# Set REACT_APP_API_URL in frontend/.env
npm start
```

---

## Docker Setup

Run the entire stack (frontend + backend + database) with a single command:

```bash
docker-compose up --build
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:3001/api/destinations |
| PostgreSQL | localhost:5432 |

---

## CI/CD Pipeline

The project uses **GitHub Actions** for fully automated CI/CD. Two separate workflow files handle the frontend and backend independently, so a change in one service does not unnecessarily trigger a build in the other.

### Workflow Files

| File | Triggers on changes to |
|---|---|
| `.github/workflows/backend.yml` | `backend/**` |
| `.github/workflows/frontend.yml` | `frontend/**` |

### Pipeline Stages

Both workflows share the same two-job structure:

```
Push / PR to main or dev
         │
         ▼
  ┌─────────────┐
  │  Job 1: CI  │  Lint · Audit · (Test + Build for frontend)
  └──────┬──────┘
         │ on success
         ▼
  ┌─────────────┐
  │  Job 2: CD  │  Build Docker image → Push to Docker Hub
  └─────────────┘  (runs on push only, not on PRs)
```

#### Backend pipeline (`backend.yml`)

1. **Lint & Audit** — installs Node 20 deps, runs `npm audit` for known vulnerabilities.
2. **Build & Push** — builds the backend Docker image and pushes it to Docker Hub tagged with:
   - `sha-<short-commit-sha>` (every push)
   - `latest` (main branch only)
   - `<branch-name>`

#### Frontend pipeline (`frontend.yml`)

1. **Lint, Test & Build** — installs Node 20 deps, runs `npm run build` (catches ESLint errors at compile time) and `npm test` (Jest unit tests).
2. **Build & Push** — builds the multi-stage Nginx Docker image and pushes it to Docker Hub with the same tagging strategy.

### Required GitHub Secrets

Before the workflows can push images you must add the following secrets in your GitHub repo:
**Settings → Secrets and variables → Actions → New repository secret**

| Secret name | Value |
|---|---|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_TOKEN` | A Docker Hub access token (not your password) |

> **Tip:** Generate a Docker Hub access token at https://hub.docker.com/settings/security

### Image Tags

Every successful push produces images with consistent, traceable tags:

```
<DOCKER_USERNAME>/dream-vacation-backend:sha-a1b2c3d   ← commit SHA
<DOCKER_USERNAME>/dream-vacation-backend:main           ← branch
<DOCKER_USERNAME>/dream-vacation-backend:latest         ← main only

<DOCKER_USERNAME>/dream-vacation-frontend:sha-a1b2c3d
<DOCKER_USERNAME>/dream-vacation-frontend:main
<DOCKER_USERNAME>/dream-vacation-frontend:latest
```

### Pulling the Latest Images

```bash
docker pull <DOCKER_USERNAME>/dream-vacation-backend:latest
docker pull <DOCKER_USERNAME>/dream-vacation-frontend:latest
```

---

## Docker Architecture

```
                 ┌──────────────────┐
                 │   Docker Network  │
                 │  vacation-network │
   Port 80  ───▶ │    frontend       │
                 │  (React + Nginx)  │
                 └────────┬─────────┘
                          │ HTTP
                 ┌────────▼─────────┐
   Port 3001 ───▶│    backend        │
                 │  (Node + Express) │
                 └────────┬─────────┘
                          │ TCP 5432
                 ┌────────▼─────────┐
                 │       db          │
                 │   (PostgreSQL)    │
                 │ Volume: pg-data   │
                 └──────────────────┘
```

All services communicate over a custom Docker bridge network (`vacation-network`). PostgreSQL data is persisted through a named Docker volume (`postgres-data`).

---

## External API

This project integrates with the [REST Countries API](https://restcountries.com) to fetch country metadata.

```
GET https://restcountries.com/v3.1/name/{countryName}
```

> **Note:** The API v3 endpoint has been deprecated. A migration to v5 is tracked in the roadmap.

---

## Best Practices

- ✅ Version control with Git
- ✅ Environment-based configuration (`.env` files, GitHub Secrets)
- ✅ Separation of frontend/backend services
- ✅ Multi-stage Docker builds (frontend) for minimal image size
- ✅ Path-filtered workflows — only rebuild what changed
- ✅ Docker layer caching (`cache-from/cache-to: type=gha`) for fast CI runs
- ✅ Images tagged with commit SHA for full traceability
- ✅ PRs never push images — secrets are safe