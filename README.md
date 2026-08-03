# Evaluer Evaluation System

**Evaluer** is an automated student evaluation and grading system designed to integrate seamlessly with the **Hive** course platform. It automatically processes, calculates, and presents evaluation metrics, course progress, student dashboards, and distribution statistics.

---

## 🏗️ Architecture Overview

The system consists of two primary Dockerized services running on a shared bridge network:

1. **`evaluer` (Backend API)**
   - Core evaluation service that connects to the **Hive** course platform API.
   - Computes scores based on weights and rules configured in `evaluer-config.yaml`.
   - Exposes RESTful API endpoints at port `8000` (internal network).

2. **`evaluer-dashboard` (Frontend SPA & Reverse Proxy)**
   - React-based Web Dashboard providing student dashboards, analytics tables, and distribution views.
   - Built on Nginx, serving static frontend assets on port `80` (and `3000`).
   - Handles client-side SPA routing and reverse-proxies requests matching `/api/` and `/api-health` directly to the `evaluer` backend container.

---

## 📋 Prerequisites

Before running the system, ensure you have the following installed on your host system:

- [Docker Engine](https://docs.docker.com/get-docker/) (v20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2.0+)

---

## 🚀 Quick Start Guide

### Step 1: Load Docker Images from Tarballs

The pre-built container images are supplied as offline tar archives (`.tar`). Load both images into your local Docker engine:

```bash
docker load -i evaluer.tar
docker load -i dashboard.tar
```

Verify that the images were successfully loaded:

```bash
docker images
```

You should see `evaluer:v1` and `dashboard:v1` in the image list.

---

### Step 2: Configure Environment Variables (`.env`)

Copy the provided `.env.example` template to create your `.env` file:

```bash
# On Linux / macOS / Bash:
cp .env.example .env

# On Windows (PowerShell):
Copy-Item .env.example .env
```

Edit the `.env` file with your **Hive** platform credentials and configuration settings:

```env
HIVE_BASE_URL="https://your-hive-platform-url.com"
HIVE_USERNAME="your_hive_username"
HIVE_PASSWORD="your_hive_password"
CONFIG_PATH="/app/.local/config.yaml"
```

#### Environment Variables Reference

| Variable | Description | Example |
| :--- | :--- | :--- |
| `HIVE_BASE_URL` | Base URL of the target Hive course platform instance. | `https://hive.example.com` |
| `HIVE_USERNAME` | Username for authenticating with the Hive platform API. | `admin_evaluer` |
| `HIVE_PASSWORD` | Password for authenticating with the Hive platform API. | `SecretPassword123` |
| `CONFIG_PATH` | Path inside the container to the evaluation config file. | `/app/.local/config.yaml` |

---

### Step 3: Review Evaluation Configuration (`evaluer-config.yaml`)

The `evaluer-config.yaml` file defines subjects, modules, exercises, weight distributions, and grading scores. This file is mounted read-write into the container at `/app/.local/config.yaml`.

Example configuration:

```yaml
subjects:
  Sukmi:
    name: Sukmi
    weight: 1.0
    modules:
      dreams:
        name: dreams
        weight: 1.0
        exercises:
          shichrur:
            name: shichrur
            score: 100
```

---

### Step 4: Start the System with Docker Compose

Launch all containers in detached mode:

```bash
docker compose up -d
```

*(If using older Compose versions, run `docker-compose up -d`)*

To check the status of running containers:

```bash
docker compose ps
```

---

## 🌐 Accessing the Application

Once running, access the system endpoints in your browser or HTTP client:

- **Dashboard UI**: [http://localhost](http://localhost) or [http://localhost:3000](http://localhost:3000)
- **API Health Check**: [http://localhost/api-health](http://localhost/api-health)
- **Backend API Base**: `http://localhost/api/`

---

## 🛠️ Operational Commands

### View Logs

To view real-time log output from all services:

```bash
docker compose logs -f
```

To view logs for a specific service:

```bash
docker compose logs -f evaluer
docker compose logs -f evaluer-dashboard
```

### Stop & Remove Containers

To stop the running application:

```bash
docker compose down
```

To stop containers and remove unused network resources:

```bash
docker compose down --volumes --remove-orphans
```

---

## 🔒 Port Exposure & Security Notes

By default, the `evaluer` backend container is accessible only within the internal Docker bridge network (`internal-net`) for enhanced security. All API traffic passes through the Nginx reverse proxy on port `80`.

If you need to expose port `8000` directly for development or API testing, uncomment lines 18-19 in [`docker-compose.yml`](file:///C:/Users/Tomer/Documents/evaluer-compose/docker-compose.yml):

```yaml
  evaluer:
    image: evaluer:v1
    container_name: evaluer
    ports:
      - "8000:8000"
```

---

## 📂 Project Structure

```text
.
├── .env.example          # Environment variables template
├── evaluer-config.yaml   # Subject and grading configuration rules
├── docker-compose.yml    # Docker Compose service orchestration
├── nginx.conf            # Nginx server configuration & API proxy setup
├── dashboard.tar         # Pre-built Frontend Docker image archive
└── evaluer.tar           # Pre-built Backend Docker image archive
```
