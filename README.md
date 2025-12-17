# VidCensor

![License: Proprietary](https://img.shields.io/badge/License-Proprietary-red.svg)
![Python](https://img.shields.io/badge/python-3.11-blue.svg)
![React](https://img.shields.io/badge/react-%2320232a.svg?style=flat&logo=react&logoColor=%2361DAFB)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=flat&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=flat&logo=redis&logoColor=white)
![Celery](https://img.shields.io/badge/celery-%2337814A.svg?style=flat&logo=celery&logoColor=white)


VidCensor is a full-stack, automated video sanitization platform designed to automatically detect and censor (mute or beep) profanity in the audio track of video files **without re-encoding the video stream**. This preserves video quality and significantly speeds up the process compared to full video re-rendering.

## Features

* **Full-Stack Web App:** Drag-and-drop UI for uploading videos and tracking status.
* **GPU Acceleration:** Uses NVIDIA CUDA to detect and beep/mute/silence profane words at lightning speeds.
* **No Re-Encoding:** Preserves original video quality by only manipulating the audio stream.
* **Background Processing:** Asynchronous Task Queue (Celery) handles heavy ML workloads.
* **Cloud-Native Storage:** Built-in MinIO (S3 compatible) for file management, ready for AWS deployment.
* **Auto-Cleanup:** Automated retention policies delete old files to save storage space.


## Demo (Turn on the volume from the options below to listen the censored words)

update link here

## System Architecture

The application runs as a set of Docker containers orchestrated by `docker-compose`:

| Service | Technology | Description |
| :--- | :--- | :--- |
| **Frontend** | React + Vite | Modern web UI for uploads and monitoring. |
| **Backend** | FastAPI | REST API managing tasks, uploads, and downloads. |
| **Worker** | Celery + PyTorch | GPU-enabled worker for AI tasks. |
| **Beat** | Celery Beat | Scheduler for periodic tasks (e.g., file cleanup). |
| **Broker** | Redis | Message broker for task queues. |
| **Database** | PostgreSQL | Persists task metadata and status. |
| **Storage** | MinIO | S3-compatible object storage for video files. |


## Prerequisites

To run the GPU-accelerated stack, your host machine needs the following:

### 1. Hardware
* **NVIDIA GPU:** Compute Capability 3.5+ (Recommended: 6GB+ VRAM).
* **RAM:** 16GB+ recommended.

### 2. Software
* **Docker Engine** & **Docker Compose (v2)**
* **NVIDIA Drivers:** Install the latest drivers for your GPU.
* **NVIDIA Container Toolkit:** This is **required** for Docker to access your GPU.

#### Installing NVIDIA Container Toolkit (Linux/WSL2)
```bash
curl -fsSL [https://nvidia.github.io/libnvidia-container/gpgkey](https://nvidia.github.io/libnvidia-container/gpgkey) | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L [https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list](https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list) | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## Directory Structure

```text
VidCensorPy/
├── backend/                # FastAPI & Celery Python Code
│   ├── app/
│   │   ├── api/            # API Route endpoints
│   │   ├── worker/         # Celery tasks (ML processing logic)
│   │   ├── core/           # Configuration & Config.py
│   │   └── s3.py           # S3 Client Logic
│   ├── vidcensorpy/        # Core censoring library
│   └── tests/              # Backend tests
├── frontend/               # React Application (Source code)
├── docker-compose.yml      # Orchestration for all services
├── Dockerfile.backend      # Lean API image
├── Dockerfile.worker       # Heavy GPU image (PyTorch/Whisper)
├── requirements-backend.txt
├── requirements-worker.txt
└── minio-init.sh           # Script to provision local S3 buckets
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
* **API Framework:** [FastAPI](https://fastapi.tiangolo.com/) (High-performance Python web framework)
* **Task Queue:** [Celery](https://docs.celeryq.dev/) (Distributed task execution)
* **Message Broker:** [Redis](https://redis.io/) (In-memory data structure store)
* **Database:** [PostgreSQL](https://www.postgresql.org/) (Persistent task metadata storage)
* **Object Storage:** [MinIO](https://min.io/) (S3-compatible local cloud storage simulation)
* **ML Engine:**

### Key Features
* **Asynchronous Processing:** Long-running tasks (transcription, remuxing) are offloaded to background workers, preventing request timeouts.
* **GPU Acceleration:** Dedicated workers utilize NVIDIA CUDA cores for lightning-fast processing.
* **Cloud-Native Storage:** "Dual-Host" S3 architecture allows the backend to communicate internally (Docker-to-Docker) while serving Presigned URLs externally (Browser-to-Docker).
* **Audio Fidelity Protection:** Uses an intermediate lossless WAV format to prevent generational loss and dynamically matches the source audio bitrate during final export.
* **Auto-Maintenance:** Integrated Celery Beat scheduler automatically enforces retention policies, cleaning up stale raw/processed files to optimize storage costs.

### Component Structure
The backend follows a **12-Factor App** design, separating configuration, API routes, and worker logic into distinct modules.

```text
backend/app/
├── api/
│   └── v1.py                 # REST Endpoints (Upload, Status Polling)
├── db/
│   └── crud.py               # Database Access Layer (SQLAlchemy)
├── worker/
│   ├── tasks.py              # Celery Task Definitions (The Logic Orchestrator)
│   ├── cleanup.py            # Periodic Maintenance Tasks (S3 Retention)
│   └── celery_app.py         # Celery & Beat Configuration
└── s3.py                     # Smart S3 Client (Handles Internal vs External URLs)
```


## License & Copyright

**Copyright (c) 2025 Rishabh Dixit. All Rights Reserved.**

VidCensor is proprietary software.

The source code for this project is hosted in a private repository to protect intellectual property.
* **Recruiters & Hiring Managers:** If you are reviewing my application and wish to examine the source code, please contact me directly. I can provide temporary read-access to the private repository.

* **Licensing:** The software is not currently available for public use or distribution.

Please contact: [rdrishabh38@gmail.com] if you would like to connect for further discussion about your use case.
