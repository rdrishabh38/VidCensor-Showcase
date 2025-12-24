# VidCensor

![License: Proprietary](https://img.shields.io/badge/License-Proprietary-red.svg)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Tests](https://img.shields.io/badge/tests-74%20passed-success)
![Coverage](https://img.shields.io/badge/coverage-96%25-success)

![Python](https://img.shields.io/badge/python-3.11-blue.svg)
![CUDA](https://img.shields.io/badge/CUDA-12.1-76B900?logo=nvidia&logoColor=white)
![FFmpeg](https://img.shields.io/badge/FFmpeg-007808?style=flat&logo=ffmpeg&logoColor=white)

![Vue.js](https://img.shields.io/badge/vuejs-%2335495e.svg?style=flat&logo=vuedotjs&logoColor=%234FC08D)
![Vite](https://img.shields.io/badge/vite-%23646CFF.svg?style=flat&logo=vite&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=flat&logo=tailwind-css&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=flat&logo=nginx&logoColor=white)

![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi)
![Pydantic](https://img.shields.io/badge/Pydantic-E92063?style=flat&logo=pydantic&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat&logo=sqlalchemy&logoColor=white)
![Celery](https://img.shields.io/badge/celery-%2337814A.svg?style=flat&logo=celery&logoColor=white)

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=flat&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=flat&logo=redis&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-C72E49?style=flat&logo=minio&logoColor=white)

![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-%23000000.svg?style=flat&logo=opentelemetry&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=flat&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=Prometheus&logoColor=white)
![Jaeger](https://img.shields.io/badge/Jaeger-60D0E4?style=flat&logo=jaegertracing&logoColor=white)

![Poetry](https://img.shields.io/badge/poetry-package_manager-blue)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)
![Type Checker: Mypy](https://img.shields.io/badge/type_checker-mypy-blue)
![Security: Bandit](https://img.shields.io/badge/security-bandit-yellow.svg)
![Testcontainers](https://img.shields.io/badge/testcontainers-enabled-1f2937?logo=testcontainers&logoColor=white)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://github.com/pre-commit/pre-commit)
![Git Cliff](https://img.shields.io/badge/changelog-git--cliff-ff69b4?logo=git&logoColor=white)


VidCensor is a full-stack, automated video sanitization platform designed to automatically detect and censor (mute or beep) profanity in the audio track of video files **without re-encoding the video stream**. This preserves video quality and significantly speeds up the process compared to full video re-rendering.


## Features

* **Full-Stack Web App:** Drag-and-drop UI for uploading videos and tracking status.
* **GPU Acceleration:** Uses NVIDIA CUDA to transcribe audio at lightning speeds.
* **High-Concurrency Backend:** **FastAPI Async** architecture handles multiple concurrent uploads without blocking.
* **No Re-Encoding:** Preserves original video quality by only manipulating the audio stream.
* **Background Processing:** Asynchronous Task Queue (Celery) handles heavy ML workloads.
* **Cloud-Native Storage:** Built-in MinIO (S3 compatible) for file management, ready for AWS deployment.
* **Auto-Cleanup:** Automated retention policies delete old files to save storage space.
* **Security Guardrails:**
    * **Frontend (UX):** "Fail-Fast" validation instantly rejects files > 300MB or > 5 minutes long.
    * **Backend (Security):** Strict content-length checks and FFmpeg probing enforce hard limits.
* **Full Observability Stack:**
    * **Application:** Distributed Tracing (Jaeger) and Request Metrics (Prometheus).
    * **Infrastructure:** **cAdvisor** integration monitors raw Container CPU/RAM usage to detect bottlenecks.


## Demo (Turn on the volume from the options below to listen the censored words)

https://github.com/user-attachments/assets/57611c05-e25a-43e7-8022-354394ccd971

## System Architecture

The application runs as a set of Docker containers orchestrated by `docker-compose`:

| Service | Technology | Description |
| :--- | :--- | :--- |
| **Frontend** | Vue.js + Vite | Modern web UI for uploads and monitoring. |
| **Backend** | FastAPI | REST API managing tasks, uploads, and downloads. |
| **Worker** | Celery + PyTorch | GPU-enabled worker running AI tasks & FFmpeg. |
| **Beat** | Celery Beat | Scheduler for periodic tasks (e.g., file cleanup). |
| **Broker** | Redis | Message broker for task queues. |
| **Database** | PostgreSQL | Persists task metadata and status. |
| **Storage** | MinIO | S3-compatible object storage for video files. |
| **Telemetry** | OTel, Jaeger, Prometheus, Grafana | Comprehensive monitoring: Logs, Traces, and Metrics. |
|**Infra Monitor**| cAdvisor | Real-time container resource usage (CPU/RAM/Net). |


## Prerequisites

To run the GPU-accelerated stack, your host machine needs the following:

### 1. Hardware
* **NVIDIA GPU:** Compute Capability 3.5+ (Recommended: 6GB+ VRAM).
* **RAM:** 16GB+ recommended.

### 2. Software
* **Docker Engine** & **Docker Compose (v2)**
* **NVIDIA Drivers:** Install the latest drivers for your GPU.
* **NVIDIA Container Toolkit:** This is **required** for Docker to access your GPU.


## Directory Structure

```text
VidCensor/
├── backend/                            # FastAPI & Celery Python Code
│   ├── app/
│   │   ├── api/                        # API Route endpoints
│   │   ├── worker/                     # Celery tasks (ML processing logic)
│   │   ├── core/                       # Configuration & Config.py
│   │   └── s3.py                       # S3 Client Logic
│   ├── vidcensor/                      # Core censoring library (FFmpeg/Pydub logic)
│   └── tests/                          # Comprehensive Test Suite (96% Coverage)
│       ├── integration/                # Integration Tests (API/DB/Worker with Testcontainers)
│       └── vidcensor/                  # Unit Tests (Pure Logic)
│   ├── pyproject.toml                  # Poetry Dependency Management
│   └── poetry.lock                     # Locked dependencies
├── frontend/                           # Vue.js Application (Source code)
├── telemetry/                          # Infrastructure as Code (Observability)
│   ├── dashboards/                     # JSON definitions for Grafana
│   ├── dashboards.yaml                 # Dashboard provisioning
│   ├── datasources.yaml                # Prometheus connection config
│   ├── otel-collector-config.yaml      # OpenTelemetry Pipeline config
│   └── prometheus.yaml                 # Metrics scraping config
├── docker-compose.yml                  # Orchestration for all services
├── Dockerfile.backend                  # Lean API image
├── Dockerfile.worker                   # Heavy GPU image
├── requirements-backend.txt
├── requirements-worker.txt
└── minio-init.sh                       # Script to provision local S3 buckets
```

## Frontend Architecture

The user interface is a responsive Single Page Application (SPA) built to interact seamlessly with the VidCensor API. It allows users to upload videos, track the GPU-accelerated processing pipeline in real-time, and preview/download the sanitized results.

### Tech Stack
* **Framework:** [Vue.js 3](https://vuejs.org/) (Composition API)
* **Build Tool:** [Vite](https://vitejs.dev/) (Fast hot-module replacement)
* **Styling:** [Tailwind CSS v4.0](https://tailwindcss.com/) (Utility-first CSS)
* **HTTP Client:** [Axios](https://axios-http.com/) (Centralized API service)
* **Deployment:** Nginx (Alpine Linux)

### Key Features
* **Drag-and-Drop Upload:** Intuitive file selection zone supporting MP4 files.
* **Real-Time Polling:** The UI polls the backend every 2 seconds to fetch granular task updates (e.g., *Extracting Audio* → *AI Transcribing* → *Remuxing*).
* **Granular Status Mapping:** Automatically maps backend Enum statuses (e.g., `EXTRACTING_AUDIO`) to user-friendly messages using Vue computed properties.
* **In-Browser Preview:** Integrated HTML5 video player allows users to verify the censorship (beeps/silence) before downloading the final file.
* **Error Handling:** Robust error management for file size limits, connection timeouts, and backend failures.
* **Synchronous Size Check:** Rejects files exceeding the configured limit (Default: 300MB) instantly upon selection.
* **Asynchronous Duration Check:** Uses a hidden HTML5 video element to probe metadata and reject videos longer than the limit (Default: 5 mins) *without* uploading or playing the file.
* **Cancellable Uploads:** Utilizes `AbortController` signals to allow users to terminate large uploads mid-stream via a prominent "Cancel" button.

### Component Structure
The frontend is refactored into a modular architecture to separate logic (`App.vue`), presentation (`components/`), and data fetching (`services/`).

```text
frontend/src/
├── components/
│   ├── UploadZone.vue        # Handles file drag-and-drop & validation
│   ├── ProcessingStatus.vue  # Visualizes the pipeline steps & progress bar
│   └── DownloadReady.vue     # Success state with Video Player & Download button
├── services/
│   └── api.js                # Centralized Axios client for API communication
└── App.vue                   # Main Controller (Manages State: Idle -> Processing -> Completed)
```


## Backend Architecture

The backend is a high-performance, asynchronous microservice designed to handle heavy ML workloads without blocking the user interface. It decouples the API layer (FastAPI) from the heavy lifting (Celery/GPU), ensuring the server remains responsive even while transcoding large video files.

### Tech Stack
* **API Framework:** [FastAPI](https://fastapi.tiangolo.com/) (**Fully Async** implementation with `aiofiles` & `asyncpg`)
* **Task Queue:** [Celery](https://docs.celeryq.dev/) (Distributed task execution)
* **Message Broker:** [Redis](https://redis.io/) (In-memory data structure store)
* **Database:** [PostgreSQL](https://www.postgresql.org/) (Persistent task metadata storage)
* **Object Storage:** [MinIO](https://min.io/) (S3-compatible local cloud storage simulation)

### Key Features
* **Asynchronous Processing:** Long-running tasks (transcription, remuxing) are offloaded to background workers, preventing request timeouts.
* **GPU Acceleration:** Dedicated workers utilize NVIDIA CUDA cores for lightning-fast processing.
* **Cloud-Native Storage:** "Dual-Host" S3 architecture allows the backend to communicate internally (Docker-to-Docker) while serving Presigned URLs externally (Browser-to-Docker).
* **Audio Fidelity Protection:** Uses an intermediate lossless WAV format to prevent generational loss and dynamically matches the source audio bitrate during final export.
* **Auto-Maintenance:** Integrated Celery Beat scheduler automatically enforces retention policies, cleaning up stale raw/processed files to optimize storage costs.


### Observability & Resilience
* **Structured Logging:** All services output machine-readable JSON logs via `structlog`, correlated with Trace IDs.
* **Distributed Tracing:** A single Trace ID follows a request from the API (`POST /upload`) -> Redis Queue -> GPU Worker (`process_video_task`), visible as a unified waterfall in Jaeger.
* **Health Monitoring:** A dedicated `/health` endpoint runs background checks on DB, Redis, and S3 connectivity, feeding real-time status to the dashboard.
* **Infrastructure Monitoring:** **cAdvisor** runs as a sidecar to scrape raw Docker statistics, allowing Grafana to correlate Application Load (Requests/sec) with Infrastructure Stress (Container CPU spikes).
* **Infrastructure as Code:** Grafana Dashboards and Data Sources are auto-provisioned via YAML.

### Component Structure
The backend follows a **12-Factor App** design, separating configuration, API routes, and worker logic into distinct modules.

```text
backend
├── alembic/                # Database migration scripts (schema versioning).
├── app/
│   ├── api/
│   │   ├── health/         # System health checks (DB, Redis, S3) & background polling.
│   │   ├── v1/
│   │   │   ├── tasks/      # Task status polling & download link generation.
│   │   │   └── upload/     # Video upload handling & validation.
│   │   └── deps.py         # Async database dependency injection.
│   ├── core/
│   │   ├── config.py       # Pydantic settings (Env vars: DB URL, S3 credentials).
│   │   ├── logging.py      # Structlog configuration for JSON logging.
│   │   └── telemetry.py    # OpenTelemetry setup (Tracing & Metrics).
│   ├── db/
│   │   ├── models.py       # SQLAlchemy ORM models (Task table definition).
│   │   └── session.py      # Dual-engine setup (Async for API, Sync for Worker).
│   ├── schemas/            # Pydantic models for request/response validation.
│   ├── worker/
│   │   ├── celery_app.py   # Celery app & Beat scheduler configuration.
│   │   ├── tasks.py        # Main GPU processing pipeline.
│   │   └── cleanup.py      # Periodic maintenance tasks (S3 retention).
│   ├── crud.py             # Synchronous DB operations (for Worker).
│   ├── crud_async.py       # Asynchronous DB operations (for API).
│   ├── main.py             # FastAPI entrypoint & middleware configuration.
│   └── s3.py               # MinIO/S3 client wrapper (Dual-host support).
├── tests/                  # Hybrid test suite (Integration + Unit).
│   ├── integration/        # Testcontainers-based tests (Real DB/MinIO).
│   └── vidcensor/          # Unit tests for core business logic.
│   └── unit/               # Unit tests for other API logic.
├── vidcensor/              # Core logic library.
├── poetry.lock             # Exact dependency versions (Reproducibility).
├── pyproject.toml          # Project metadata & dependency definitions.
└── profanity_list.txt      # Censorship dictionary.
```

## Quick Start

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/rdrishabh38/[REDACTED].git
    cd REDACTED
    ```

2.  **Setup Environment Variables:**
    modify the properties from `backend/app/core/config.py` as required or leave them be.

3.  **Launch the Stack:**
    ```bash
    docker compose up -d --build
    ```

4.  **Access the Application:**
    * **Frontend UI:** [http://localhost:5173](http://localhost:5173)
    * **API Documentation:** [http://localhost:8000/docs](http://localhost:8000/docs)
    * **Grafana Dashboards:** [http://localhost:3001](http://localhost:3001) (User/Pass: `admin`/`admin`)
    * **Jaeger Traces:** [http://localhost:16686](http://localhost:16686)
    * **MinIO Console:** [http://localhost:9001](http://localhost:9001) (User/Pass: `minioadmin`)


## Frontend Deployment: Dev vs. Prod

The project includes two Docker configurations for the frontend:

### 1. Development Mode (Hot-Reload)
* **Port:** `3000` (Mapped to internal 5173)
* **Description:** Runs Vite Dev Server. Changes to `.vue` files are reflected instantly.
* **Command:** Standard `docker compose up` uses this mode by default.

### 2. Production Mode (Static Nginx)
* **Port:** `8080`
* **Description:** Uses a Multi-Stage Docker build. Node.js compiles the assets, and a lightweight Nginx container serves them with Gzip compression and caching.
* **To Run:**
    ```bash
    docker build -t vidcensor-frontend-prod -f frontend/Dockerfile.prod ./frontend
    docker run -p 8080:80 vidcensor-frontend-prod
    ```


## Configuration

VidCensor follows the **12-Factor App** methodology. Configuration is managed via environment variables in the `config.py` file.

| Variable | Default | Description |
| :--- | :--- | :--- |
| `CENSOR_MODE` | `silence_and_beep` | How to censor: `beep`, `silence`, or `silence_and_beep`. |
| `RETENTION_MINUTES` | `60` | Time before raw/processed videos are auto-deleted. |
| `S3_ENDPOINT_URL` | `http://minio:9000` | Leave set for local dev, unset for AWS S3. |
| `FFMPEG_PATH` | `ffmpeg` | Usually just `"ffmpeg"`, but provide the full path if it's not in your system PATH. |

* Adjust other settings like padding, bitrate, etc., as needed.

**Edit `profanity_list.txt`:**
    Add the words you want to censor (one word per line, case-insensitive).

more configurations are present in `backend/app/core/config.py`, please refer it for more parameter tweaking.

### Frontend Configuration (.env)
Create a `.env` file in the `frontend/` directory to control UI-level constraints. These are "baked in" at build time.

| Variable | Default | Description |
| :--- | :--- | :--- |
| `VITE_MAX_FILE_SIZE_MB` | `300` | Maximum file size in Megabytes allowed for upload. |
| `VITE_MAX_DURATION_SEC` | `300` | Maximum video duration in Seconds allowed. |


## Development Workflow

* **View Logs:**
    ```bash
    docker compose logs -f backend worker cadvisor
    ```

* **Check System Health:**
    ```bash
    curl http://localhost:8000/health
    # Returns: {"status": "healthy", "components": {"postgres": 1, "redis": 1, "minio": 1}}
    ```

* **Rebuild specific service (e.g., after changing requirements):**
    ```bash
    docker compose up -d --build backend
    ```

* **Clean Reset (Ephemeral):**
    The DB and Storage are configured to be ephemeral by default. To wipe all data and start fresh:
    ```bash
    docker compose down
    docker compose up -d
    ```

## Testing & Quality Assurance

This project maintains a high standard of quality with a **95% Code Coverage** enforcement policy.

### Test Architecture
We use a hybrid testing strategy:
* **Unit Tests (`tests/vidcensor/`):** Verify pure business logic (FFmpeg commands, censoring math) in isolation.
* **Integration Tests (`tests/integration/`):** Use **Testcontainers** to spin up real, ephemeral Docker containers for **Postgres** and **MinIO**. This ensures tests run against actual infrastructure logic, not just mocks.

### Running Tests Locally

1.  **Ensure Docker is running** (Required for Testcontainers).
2.  **Run the full suite:**

    ```bash
    cd backend
    poetry run pytest tests/
    ```

    *This command will auto-provision the necessary DB/S3 containers, run all tests, and generate a coverage report. The command will fail if coverage drops below 95%.*

### Continuous Integration (CI)
Every Pull Request is automatically vetted by GitHub Actions:
* **System Dependencies:** Installs `ffmpeg` and `libmagic` on the runner.
* **Quality Gates:** Fails the pipeline if tests fail or if Code Coverage drops below **95%**.

## Contributing & Development Setup

This project adheres to strict coding standards (Strict Typing, JSON Logging, Security Checks) to ensure robustness.

### 1. Local Python Setup (Backend)

We use **[Poetry](https://python-poetry.org/)** for dependency management.

1.  **Install Poetry:**
    ```bash
    curl -sSL [https://install.python-poetry.org](https://install.python-poetry.org) | python3 -
    ```
2.  **Install Dependencies:**
    Navigate to the backend directory and install the environment.
    ```bash
    cd backend
    poetry install --with dev,worker
    ```
3.  **Activate Shell:**
    ```bash
    poetry shell
    ```

### 2. Pre-Commit Hooks (Crucial)

This project uses `pre-commit` to enforce standards **before** code is committed. This pipeline includes:
* **Mypy:** Strict static type checking.
* **Ruff:** Fast linting.
* **Black/Isort:** Code formatting.
* **Bandit:** Security vulnerability scanning.

**Setup:**
Run this once after cloning:
```bash
pre-commit install
```

**Manual Run: To check all files without committing:**

```
pre-commit run --all-files
```

### 3. Managing Dependencies (Docker Sync)

We maintain a strict separation between **Development** (Poetry) and **Production** (Docker) dependencies to keep images optimized.

* **`pyproject.toml`**: The Single Source of Truth.
* **`requirements-backend.txt`**: Auto-generated lean dependencies for the API.
* **`requirements-worker.txt`**: Auto-generated heavy AI dependencies for the GPU Worker.

**Workflow to add a new library:**
1.  Add package via Poetry: `cd backend && poetry add <package_name>`
2.  Regenerate the Docker requirements files:
    ```bash
    # Run from root directory
    make requirements
    ```
    *This runs `poetry export` to strictly pin versions and sync all files.*


## Release Process

We use **Semantic Versioning** and automate our `CHANGELOG.md` using **Git-Cliff**.

**To cut a new release:**
1.  Commit all changes.
2.  Push a new tag:
    ```bash
    git tag v2.1.0
    git push origin v2.1.0
    ```
3.  **Automation:** A GitHub Action will trigger, generating the changelog for the new commits, prepending it to `CHANGELOG.md`, and pushing the update back to the `main` branch.


## License & Copyright

**Copyright (c) 2025 Rishabh Dixit. All Rights Reserved.**

VidCensor is proprietary software.

The source code for this project is hosted in a private repository to protect intellectual property.
* **Recruiters & Hiring Managers:** If you are reviewing my application and wish to examine the source code, please contact me directly. I can provide temporary read-access to the private repository.

* **Licensing:** The software is not currently available for public use or distribution.

Please contact: [rdrishabh38@gmail.com] if you would like to connect for further discussion about your use case.
