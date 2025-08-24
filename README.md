# Docker Containers Description for Exalidraw Project

This document describes all Docker containers that are part of the Exalidraw project.

## Main Application Containers

### 1. Client Container
- **Image**: Custom built from `./client/Dockerfile`
- **Port**: `6969:80` (host:container)
- **Purpose**: Frontend React application
- **Network**: `Exalidraw`
- **Dependencies**: Depends on server container

### 2. Server Container
- **Image**: Custom built from `./server/Dockerfile`
- **Port**: `8080:8080` (host:container)
- **Purpose**: Backend Go application
- **Network**: `Exalidraw`
- **Volumes**: `./server/logs:/app/logs`

## Observability Stack Containers (Server Directory)

### 3. Loki Read Container
- **Image**: `grafana/loki:latest`
- **Ports**: `3101:3100`, `7946`, `9095`
- **Purpose**: Loki log aggregation read component
- **Network**: `loki`
- **Dependencies**: Depends on minio container

### 4. Loki Write Container
- **Image**: `grafana/loki:latest`
- **Ports**: `3102:3100`, `7946`, `9095`
- **Purpose**: Loki log aggregation write component
- **Network**: `loki`
- **Dependencies**: Depends on minio container

### 5. Alloy Container
- **Image**: `grafana/alloy:latest`
- **Port**: `12345:12345`
- **Purpose**: Grafana Alloy for observability pipeline
- **Network**: `loki`
- **Dependencies**: Depends on gateway container

### 6. MinIO Container
- **Image**: `minio/minio`
- **Port**: `9000` (internal)
- **Purpose**: Object storage for Loki data
- **Network**: `loki`
- **Environment Variables**:
  - `MINIO_ROOT_USER=loki`
  - `MINIO_ROOT_PASSWORD=supersecret`
  - `MINIO_PROMETHEUS_AUTH_TYPE=public`
  - `MINIO_UPDATE=off`

### 7. Grafana Container
- **Image**: `grafana/grafana:latest`
- **Port**: `3000:3000`
- **Purpose**: Web-based analytics and monitoring platform
- **Network**: `loki`
- **Dependencies**: Depends on gateway container
- **Environment Variables**:
  - `GF_PATHS_PROVISIONING=/etc/grafana/provisioning`
  - `GF_AUTH_ANONYMOUS_ENABLED=true`
  - `GF_AUTH_ANONYMOUS_ORG_ROLE=Admin`

### 8. Loki Backend Container
- **Image**: `grafana/loki:latest`
- **Ports**: `3100`, `7946` (internal)
- **Purpose**: Loki backend component
- **Network**: `loki`
- **Dependencies**: Depends on gateway container

### 9. Gateway Container
- **Image**: `nginx:latest`
- **Port**: `3100:3100`
- **Purpose**: Nginx reverse proxy for Loki services
- **Network**: `loki`
- **Dependencies**: Depends on read and write containers

### 10. Prometheus Container
- **Image**: `prom/prometheus:latest`
- **Port**: `9090:9090`
- **Purpose**: Metrics collection and monitoring
- **Network**: `loki`
- **Volumes**: 
  - `./prometheus.yml:/etc/prometheus/prometheus.yml`
  - `./data/prometheus:/prometheus`

### 11. Flog Container
- **Image**: `mingrammer/flog`
- **Purpose**: Log generation for testing
- **Network**: `loki`
- **Command**: `-f json -d 200ms -l`

## Summary

**Total Containers**: 11 containers

**Networks**:
- `Exalidraw`: Main application network
- `loki`: Observability stack network

**Ports Exposed**:
- `6969`: Client frontend
- `8080`: Server backend
- `3000`: Grafana dashboard
- `3100`: Loki gateway
- `3101`: Loki read
- `3102`: Loki write
- `9090`: Prometheus
- `12345`: Alloy

**Images Used**:
- Custom built images for client and server
- `grafana/loki:latest` (3 instances)
- `grafana/alloy:latest`
- `minio/minio`
- `grafana/grafana:latest`
- `nginx:latest`
- `prom/prometheus:latest`
- `mingrammer/flog`

## Startup Commands

The containers are started using:
1. `docker compose up -d` (main application)
2. `cd server && docker compose up -d` (observability stack)
