# 🔧 ForgeGit — Git-as-a-Service Platform

> A self-hosted Git repository service powered by **Gitea** and **PostgreSQL**, containerized with Docker Compose for easy deployment.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Services](#services)
- [Ports](#ports)
- [Volumes](#volumes)
- [Stopping & Resetting](#stopping--resetting)
- [Security Notes](#security-notes)

---

## Overview

**ForgeGit** is a lightweight, self-hosted Git-as-a-Service (GaaS) platform built on top of [Gitea](https://gitea.io/) — a fast, open-source Git service written in Go. It uses **PostgreSQL 15** as its database backend and is fully containerized using **Docker Compose**, making it easy to spin up locally or on any server.

---

## Architecture

```
┌──────────────────────────────────────────────┐
│                Docker Network                 │
│                (gaas-network)                 │
│                                              │
│   ┌─────────────┐      ┌──────────────────┐  │
│   │  PostgreSQL  │◄────►│      Gitea       │  │
│   │  (port 5432) │      │  (port 3000/22)  │  │
│   └─────────────┘      └──────────────────┘  │
│         │                       │             │
│   postgres-data           gitea-data          │
│     (volume)               (volume)           │
└──────────────────────────────────────────────┘
```

- **Gitea** handles all Git hosting, user management, and web UI.
- **PostgreSQL** serves as the persistent database backend for Gitea.
- Both services communicate internally over a dedicated Docker bridge network (`gaas-network`).

---

## Project Structure

```
forgegit-lingaa/
│
├── docker-compose.yml      # Main orchestration file for all services
│
└── gitea/
    ├── app.ini             # Gitea application configuration (pre-configured)
    └── custom/             # (Optional) Custom themes, templates, and assets
```

---

## Prerequisites

Make sure the following are installed on your machine:

| Tool | Version | Description |
|------|---------|-------------|
| [Docker](https://www.docker.com/get-started) | 20.x+ | Container runtime |
| [Docker Compose](https://docs.docker.com/compose/) | v2.x+ | Multi-container orchestration |

---

## Getting Started

### 1. Clone the Repository

```bash
git clone <your-repo-url> forgegit-lingaa
cd forgegit-lingaa
```

### 2. Start the Services

```bash
docker compose up -d
```

This will:
- Pull the `postgres:15` and `gitea/gitea:latest` images (if not already cached)
- Start both services in detached mode

### 3. Access Gitea

Open your browser and navigate to:

```
http://localhost:3000
```

Gitea is pre-configured and ready to use — no initial setup wizard is required (`INSTALL_LOCK = true` in `app.ini`).

### 4. Register Your First User

On the Gitea web interface:
1. Click **Sign Up**
2. The **first registered user** is automatically assigned as the **Admin**.

---

## Configuration

The Gitea configuration is managed via `./gitea/app.ini`, which is mounted into the container at runtime.

### Key Settings

| Section | Key | Value | Description |
|---------|-----|-------|-------------|
| `[server]` | `DOMAIN` | `localhost` | Gitea server domain |
| `[server]` | `ROOT_URL` | `http://localhost:3000/` | Public URL for Gitea |
| `[server]` | `HTTP_PORT` | `3000` | HTTP listening port |
| `[database]` | `DB_TYPE` | `postgres` | Database engine |
| `[database]` | `HOST` | `postgres:5432` | Internal DB hostname |
| `[repository]` | `MAX_CREATION_LIMIT` | `10` | Max repos per user |
| `[service]` | `REQUIRE_SIGNIN_VIEW` | `false` | Public repo browsing allowed |
| `[security]` | `INSTALL_LOCK` | `true` | Skips the setup wizard |

> **To change the domain for production**, update `DOMAIN` and `ROOT_URL` in `gitea/app.ini` and restart the services.

---

## Services

### `gitea` — Gitea Git Service

| Property | Value |
|----------|-------|
| Image | `gitea/gitea:latest` |
| Container | `gaas-gitea` |
| Web UI | `http://localhost:3000` |
| SSH | `ssh://localhost:2222` |
| Config | `./gitea/app.ini` |

### `postgres` — PostgreSQL Database

| Property | Value |
|----------|-------|
| Image | `postgres:15` |
| Container | `gaas-postgres` |
| Database | `gitea` |
| User | `gitea` |
| Internal Port | `5432` |

---

## Ports

| Host Port | Container Port | Service | Protocol |
|-----------|----------------|---------|----------|
| `3000` | `3000` | Gitea Web UI | HTTP |
| `2222` | `22` | Gitea SSH | SSH |

> **SSH Git Clone Example:**
> ```bash
> git clone ssh://git@localhost:2222/<username>/<repo>.git
> ```

---

## Volumes

Persistent data is stored in named Docker volumes to survive container restarts:

| Volume | Used By | Contents |
|--------|---------|----------|
| `postgres-data` | PostgreSQL | All database records |
| `gitea-data` | Gitea | Repositories, attachments, avatars |

---

## Stopping & Resetting

### Stop services (keep data)

```bash
docker compose down
```

### Stop and remove all data (full reset)

```bash
docker compose down -v
```

> ⚠️ The `-v` flag removes all named volumes — **this will permanently delete all repositories and database records.**

### Restart services

```bash
docker compose restart
```

### View logs

```bash
# All services
docker compose logs -f

# Gitea only
docker compose logs -f gitea

# PostgreSQL only
docker compose logs -f postgres
```

---

## Security Notes

> ⚠️ **This setup is intended for local development and internal use.** For production deployments, consider the following:

- **Change the default credentials**: Update `POSTGRES_PASSWORD` and `SECRET_KEY` in `docker-compose.yml` and `app.ini`.
- **Use environment variables or Docker secrets** instead of hardcoding credentials.
- **Enable HTTPS**: Configure a reverse proxy (e.g., Nginx, Traefik) with TLS certificates in front of Gitea.
- **Restrict SSH access**: Limit which IPs can reach port `2222`.
- **Update `ROOT_URL`**: Set it to your actual domain name in production.

---

## Authors

- [Kabilesh P](https://github.com/KabileshP)
- [Kamalesh S P](https://github.com/Kamalesh-Suresh-Kumar)
- [Kasilingam M](https://github.com/lingaa005)
