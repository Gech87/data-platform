# Data Platform - Microservices Architecture

## Overview
Professional data platform demonstrating modern microservices architecture with GitOps.

## Services
- PostgreSQL 18+ (Data Warehouse)
- pgAdmin 4 (Database Management)
- Portainer (Container Management)
- Airbyte (Data Integration)

## Quick Start
```bash
# 1. Copy environment template
cp .env.example .env
# Edit .env with your credentials

# 2. Start services
docker-compose up -d

# 3. Access services
- PostgreSQL: localhost:5432
- pgAdmin: http://localhost:8080
- Portainer: http://localhost:9001