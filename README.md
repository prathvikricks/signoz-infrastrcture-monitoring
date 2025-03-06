# signoz-infrastrcture-monitoring
# OpenTelemetry Collector Host Monitoring for SigNoz

Monitor your infrastructure with proper hostname identification in SigNoz using OpenTelemetry Collector in Docker.

## Features
- Infrastructure metrics collection (CPU, Memory, Disk, Network)
- Hostname identification (override cloud instance default names)
- Docker container monitoring
- SigNoz OTLP integration

## Prerequisites
- Docker 20.10+
- Docker Compose 2.12+
- SigNoz instance (v0.22+)
- Linux host (tested on Ubuntu 22.04)

## Configuration

### 1. Environment Setup
Create `.env` file:
```bash
echo "HOST_HOSTNAME=$(cat /etc/hostname)" > .env  # Use system hostname
```
# OR set custom hostname
```
echo "HOST_HOSTNAME=my-custom-hostname" > .env
```

## docker-compose.yml file for otel
```bash
version: '3'
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.96.0
    container_name: otel-collector
    command: ["--config=/etc/otel/config.yaml"]
    user: "0:0"
    volumes:
      - ./config.yaml:/etc/otel/config.yaml
      - /:/hostfs:ro
      - /proc:/hostfs/proc:ro
      - /sys:/hostfs/sys:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - /etc/hostname:/etc/hostname:ro
    environment:
      - OTEL_RESOURCE_ATTRIBUTES=service.name=hostmetrics,host.name=${HOST_HOSTNAME}
    restart: unless-stopped
    privileged: true
    cap_add:
      - SYS_PTRACE
      - DAC_READ_SEARCH
```
### config.yaml file for the otel
```bash
receivers:
  hostmetrics:
    collection_interval: 60s
    root_path: /hostfs
    scrapers:
      cpu: {}
      load: {}
      memory: {}
      disk: {}
      filesystem: {}
      network: {}
      processes: {}

processors:
  resourcedetection:
    detectors: [env, system]
    system:
      hostname_sources: [os]
    timeout: 2s

extensions:
  health_check: {}
  zpages: {}

exporters:
  otlp:
    endpoint: "YOUR_SIGNOZ_ENDPOINT:4317"
    tls:
      insecure: true

service:
  telemetry:
    metrics:
      address: 0.0.0.0:8888
  extensions: [health_check, zpages]
  pipelines:
    metrics/hostmetrics:
      receivers: [hostmetrics]
      processors: [resourcedetection]
      exporters: [otlp]
```
## Preview
<img width="1470" alt="Screenshot 2025-03-06 at 2 37 49 PM" src="https://github.com/user-attachments/assets/dc5811ad-0e80-42f5-a975-07afdf7d44ab"/>
<img width="1470" alt="Screenshot 2025-03-06 at 2 38 59 PM" src="https://github.com/user-attachments/assets/83bbdf53-dfe9-41dc-a502-806532c3cd33"/>
<img width="1470" alt="Screenshot 2025-03-06 at 2 39 41 PM" src="https://github.com/user-attachments/assets/e67dfdd2-5b88-4472-ac52-ee28ee420288" />





