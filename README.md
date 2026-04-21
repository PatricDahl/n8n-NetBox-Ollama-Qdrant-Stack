# n8n + NetBox + Ollama + Qdrant Stack

[![Docker Compose](https://img.shields.io/badge/Docker%20Compose-1.0-blue.svg)](https://docs.docker.com/compose/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A production-ready **Docker Compose** setup combining:
- **n8n**: Open-source workflow automation with PostgreSQL backend and demo workflows/credentials.
- **Ollama**: Local LLM inference (pulls Llama 3.2 by default), with support for **CPU**, **NVIDIA GPU**, or **AMD GPU (ROCm)**.
- **Qdrant**: Vector database for semantic search and AI embeddings.
- **NetBox**: Open-source IPAM/DCIM tool (v4.4) with PostgreSQL, Valkey (Redis), worker, and persistent storage.

All services share a `demo` network for seamless communication. Includes healthchecks, auto-imports for n8n demo data, and GPU profiles.

## 🎯 Features
- **Modular profiles**: Run Ollama on CPU (`cpu`), NVIDIA GPU (`gpu-nvidia`), or AMD ROCm (`gpu-amd`).
- **Persistent volumes** for data, models, configs, and media.
- **Demo data**: n8n workflows and credentials auto-imported on startup.
- **Healthchecks & dependencies**: Ensures reliable startup order.
- **Exposed ports**:
  | Service  | Port  |
  |----------|-------|
  | n8n     | 5678 |
  | Ollama  | 11434 |
  | Qdrant  | 6333 |
- **NetBox internals**: Accessible via internal network (expose ports manually if needed, e.g., add `ports: - "8000:8080"` to `netbox` service).

## 🛠 Prerequisites
- **Docker** (20+) and **Docker Compose** (v2+).
- **GPU Support** (optional):
  - NVIDIA: NVIDIA Container Toolkit (`nvidia-docker2`).
  - AMD ROCm: Host with ROCm drivers (`/dev/kfd`, `/dev/dri`).
- **Hardware**: 8GB+ RAM recommended; GPU for Ollama acceleration.

## 🚀 Quick Start
1. **Clone & Setup**:
   ```bash
   git clone <your-repo>  # Or download files
   cd <project-dir>
   ```

2. **Create Environment Files**:
   - Copy `.env.example` to `.env` and fill in:
     ```
     POSTGRES_USER=n8nuser
     POSTGRES_PASSWORD=your_secure_password
     POSTGRES_DB=n8n
     N8N_ENCRYPTION_KEY=your_32char_encryption_key
     N8N_USER_MANAGEMENT_JWT_SECRET=your_jwt_secret
     VERSION=1  # For NetBox image tag (e.g., docker.io/netboxcommunity/netbox:1-v4.4-3.4.1)
     ```
   - Create `env/` directory with NetBox env files (samples below):
     - `env/netbox.env`: `SECRET_KEY=your_django_secret`
     - `env/postgres.env`: `POSTGRES_DB=netbox`, `POSTGRES_USER=netbox`, `POSTGRES_PASSWORD=netpass`
     - `env/redis.env`: `REDIS_PASSWORD=redpass`
     - `env/redis-cache.env`: `REDIS_PASSWORD=redpass` (same as above)
   - Create directories:
     ```
     mkdir -p n8n/demo-data/{credentials,workflows}
     mkdir -p configuration shared env
     ```
     - Add n8n demo JSON files to `n8n/demo-data/`.
     - NetBox: Configure `configuration/settings.py` (mounts to `/etc/netbox/config`).

3. **Start Stack**:
   ```bash
   # CPU-only (default)
   docker compose up -d

   # NVIDIA GPU
   docker compose --profile gpu-nvidia up -d

   # AMD GPU
   docker compose --profile gpu-amd up -d
   ```

4. **Verify**:
   ```bash
   docker compose logs -f
   docker compose ps  # Check health
   ```

5. **Access Services**:
   - n8n: http://localhost:5678
   - Ollama: http://localhost:11434
   - Qdrant: http://localhost:6333/dashboard
   - NetBox: Internal (add port mapping: `ports: - "8000:8080"` to `netbox` service, then http://localhost:8000)

6. **Stop & Cleanup**:
   ```bash
   docker compose down  # Keep volumes
   docker compose down -v  # Remove volumes
   ```

## 📁 Directory Structure
```
.
├── docker-compose.yml      # Main compose file
├── .env                    # Common secrets
├── .env.example           # Template
├── env/
│   ├── netbox.env
│   ├── postgres.env
│   ├── redis.env
│   └── redis-cache.env
├── n8n/
│   └── demo-data/         # Workflows & credentials JSON
├── configuration/         # NetBox settings.py, local_requirements.txt, etc.
├── shared/                # Shared data (e.g., /data/shared in n8n)
└── volumes/               # (Managed by Docker)
```

## 🔧 Customization
- **NetBox Version**: Set `VERSION` in `.env` (e.g., `2` for `2-v4.4-3.4.1`).
- **Ollama Models**: Edit `ollama-pull-*.yaml` commands (pulls `llama3.2` by default).
- **Expose NetBox**: Edit `netbox` service:
  ```yaml
  ports:
    - "8000:8080"
  ```
- **n8n Data**: Place `credentials/` and `workflows/` JSON in `n8n/demo-data/`.

## 🐛 Troubleshooting
- **Ollama GPU**: Verify `nvidia-smi` (NVIDIA) or `rocm-smi` (AMD). Check logs: `docker compose logs ollama`.
- **NetBox Health**: Superuser via `docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser`.
- **Import Failures**: Ensure n8n demo JSON is valid.
- **Permissions**: Run `docker compose up` as non-root if issues.
- **Logs**: `docker compose logs <service>`.

## 📄 Sample Env Files
**env/netbox.env**:
```
SUPERUSER_API_TOKEN=your_token
REDIS_PASSWORD=$$REDIS_PASSWORD
DATABASE_PASSWORD=$$POSTGRES_PASSWORD
```

(Adapt from NetBox docs.)

## 🤝 Contributing
Fork, PR, or issues welcome!

## 📄 License
MIT License - see [LICENSE](LICENSE) file.
