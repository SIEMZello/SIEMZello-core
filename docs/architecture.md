# High-Level System Architecture

## Objective
Design a modular, cloud-native SIEM system capable of ingesting logs, detecting threats via AI, and visualizing real-time security incidents.

---

## Components Overview

### 1. **Log Collection and Processing**
- **OpenTelemetry** collects logs from multiple endpoints
- **Kafka** serves as the message broker to decouple ingestion and processing
- **Logstash + Elasticsearch (ELK Stack)** indexes and stores log data for querying

### 2. **AI-Powered Anomaly Detection**
- Logs are parsed and structured
- AI models (Isolation Forest, LSTM) analyze logs for anomalies
- Models are hosted behind FastAPI inference endpoints

### 3. **Backend API**
- FastAPI provides REST endpoints to:
  - Query logs and anomalies
  - Trigger and manage alerts
  - Serve AI model predictions
- Manages PostgreSQL storage for events and incidents

### 4. **Frontend Dashboard**
- Built with Next.js for real-time visualization
- Dashboards for:
  - Alerts overview
  - Incident timelines
  - Source filtering (IP, date, severity)

### 5. **Deployment**
- All services containerized with Docker
- Kubernetes used for orchestration and scaling
- Supports AWS/GCP cloud deployment

---

## Data Flow
```yaml
Endpoint Logs --> OpenTelemetry --> Kafka --> [ ELK + AI Inference ] --> FastAPI Backend --> PostgreSQL --> Next.js Dashboard
```

## Security Considerations
- Use HTTPS for all services
- Access tokens for API endpoints
- Secrets managed with environment files or secret stores

---

## Roadmap
- Week 1: Skeleton, docs, setup
- Week 2: Log pipeline (OpenTelemetry, Kafka)
- Week 3: AI model training and deployment
- Week 4: Backend-Frontend integration
- Week 5: Final alerts, polish, testing