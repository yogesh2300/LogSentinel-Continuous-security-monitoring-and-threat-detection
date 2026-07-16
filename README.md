# LogSentinel — Continuous Security Monitoring and Threat Detection

LogSentinel is an intelligent security monitoring platform that continuously collects and analyzes Linux system logs to detect suspicious activities, assess security risks, and provide real-time threat detection through behavioral analytics and machine learning.

## Architecture

```
User Login → Dashboard → Servers → Add Server (Test → Save)
                                        ↓
                    PostgreSQL (encrypted SSH credentials)
                                        ↓
              Background Scheduler (all active servers)
                                        ↓
         SSH Collect → Parse → Normalize → Risk Engine → Store
                                        ↓
              Isolation Forest + Random Forest → Alerts
                                        ↓
                    Dashboard (per-server metrics)
```

**SSH credentials are never read from `.env` in production.** Every Linux server is securely registered through the LogSentinel web interface.

---

## Prerequisites

- Python 3.11+
- Node.js 18+
- PostgreSQL
- One or more Linux VMs with SSH access (CentOS/RHEL/Ubuntu)

---

# Quick Start

## 1. Backend

```powershell
cd LogSentinel
copy .env.example .env
# Set DB_* credentials and SECRET_KEY

.\.venv\Scripts\Activate.ps1
pip install -r backend\requirements.txt
python scripts\migrate_schema.py
python -m backend.database.init_db
python -m backend.main
```

API Documentation:

```
http://localhost:8000/docs
```

---

## 2. Frontend

```powershell
cd frontend
npm install
npm run dev
```

Frontend:

```
http://localhost:5173
```

---

## 3. Register Linux Servers

1. Open **http://localhost:5173**
2. Register or Login.
3. Promote your account to Administrator.

```sql
UPDATE users
SET role = 'admin'
WHERE username = 'your_username';
```

4. Navigate to **Servers → Add Server**
5. Enter:
   - Server Name
   - Host/IP
   - Port
   - Username
   - Password or SSH Key
6. Click **Test Connection**
7. Save the server.
8. Start log collection manually or enable the scheduler.

---

## 4. Enable Automatic Collection

Configure `.env`

```env
SCHEDULER_ENABLED=true
COLLECTION_INTERVAL_MINUTES=15
COLLECTION_TAIL_LINES=500
```

The scheduler periodically connects to every active Linux server, collects logs, processes them through the behavioral analysis pipeline, and performs machine learning–based threat detection.

---

## 5. Run ML Detection

Navigate to:

```
Detection → Run Detection
```

Requires at least **10 events** in the database.

For demonstration:

```powershell
python scripts\seed_demo_data.py 120
```

---

# API Endpoints

| Endpoint | Auth | Description |
|----------|------|-------------|
| `GET /health` | No | Health check |
| `POST /api/v1/auth/register` | No | Register user |
| `POST /api/v1/auth/token` | No | User login |
| `GET /api/v1/servers` | JWT | List registered servers |
| `POST /api/v1/servers/test-connection` | Admin | Test SSH connection |
| `POST /api/v1/servers` | Admin | Register Linux server |
| `POST /api/v1/servers/{id}/test` | JWT | Test saved server |
| `POST /api/v1/servers/{id}/collect` | JWT | Collect logs |
| `POST /api/v1/collection/run` | Admin | Execute collection pipeline |
| `GET /api/v1/events` | JWT | Retrieve security events |
| `GET /api/v1/dashboard/summary` | JWT | Dashboard statistics |
| `POST /api/v1/detection/run` | JWT | Execute ML threat detection |

---

# Project Structure

```
LogSentinel/
├── backend/          # FastAPI backend, SSH collector, risk engine, ML pipeline
├── frontend/         # React + Vite dashboard
├── scripts/          # Database migration & demo data scripts
└── .env              # Database, JWT and scheduler configuration
```

---

# Security Analytics Pipeline

```
Linux Server
      │
      ▼
SSH Collector
      │
      ▼
Log Parser
      │
      ▼
Event Normalizer
      │
      ▼
Risk Engine
      │
      ▼
PostgreSQL
      │
      ▼
Machine Learning Engine
      │
      ▼
Dashboard & Alerts
```

---

# Machine Learning Detection

LogSentinel uses a hybrid detection engine located in:

```
backend/services/detection_service.py
```

Detection components include:

- **Rule-Based Risk Engine** — Detects high-risk events using predefined security rules.
- **Isolation Forest** — Identifies anomalous user and system behavior.
- **Random Forest** — Classifies events as suspicious or normal.

Behavioral features include:

- Server ID
- Login time
- Session duration
- Failed login attempts
- CPU, Memory and Disk utilization
- Commands executed
- Network connections
- User activity patterns

---

# Key Features

- Continuous Linux security log monitoring
- Multi-server SSH log collection
- Automated log parsing and normalization
- Risk score calculation
- Behavioral anomaly detection
- Hybrid Machine Learning (Isolation Forest + Random Forest)
- Real-time dashboard and security analytics
- Secure authentication using JWT
- PostgreSQL-based event storage
- Scalable modular architecture

---

# License

**MCA Final Year Project**

**LogSentinel: Continuous Security Monitoring and Threat Detection**
