# **Architecture Principles**
# **Data Platform Architecture Documentation**

## **Overview**
A modern, production-ready data platform built with microservices architecture and GitOps principles. This platform provides end-to-end data capabilities from ingestion to visualization, all managed through infrastructure-as-code practices.

*Architecture Principles*

Core Design Principles
1. GitOps First: All infrastructure and application configurations are stored in Git

2. Microservices Isolation: Each component runs in its own container with clear boundaries

3. Environment Parity: Identical configurations across development, staging, and production

4. Declarative Infrastructure: Everything defined as code (YAML manifests)

5. Observability First: Built-in health checks, logging, and monitoring

6. Data Pipeline as Code: Transformations (dbt) and visualizations (Superset) version-controlled

![System Architecture](https://raw.githubusercontent.com/Gech87/data-platform/main/docs/images/data-platform-architecture.png "Complete Data Pipeline Flow.")

## **Component Details**
## **1. PostgreSQL (Data Warehouse)**
Purpose: Central data storage implementing medallion architecture

* **Image:** postgres:latest (PostgreSQL 18+)

* **Port:** 5432

* **Storage:** Persistent Volume Claims (modular by layer)

* **Architecture:**

  * **Bronze Layer:** Raw ingested data from Airbyte

  * **Silver Layer:** Cleaned, validated data from dbt

  * **Gold Layer:** Business-ready aggregates (future)

* **Features:**

  * Schema-per-layer isolation

  * Row-level security policies

  * Connection pooling and optimization

  * Point-in-time recovery capabilities

## **2. pgAdmin (Database Management)**
* **Purpose:** Web-based PostgreSQL administration and SQL development

* **Image:** dpage/pgadmin4:8.0 (Stable version)

* **Port:** 80 (Service), 30080 (NodePort)

* **Storage:** PVC for configuration persistence

* **Features:**

  * Multi-database management across layers 

  * Query tool with explain plans

  * Schema comparison and migration tools

  * User role management across environments

## **3. Airbyte (Data Integration)**
Purpose: ETL/ELT data pipeline orchestration and extraction

* **Components:**

  * **Server:** API and coordination (airbyte/server:0.50.4)
  
  * **Worker:** Job execution with connector isolation (airbyte/worker:0.50.4)
  
  * **Webapp:** User interface (airbyte/webapp:0.50.4)
  
  * **Database:** Metadata storage (postgres:13-alpine)

* **Ports:**

  * **Web UI:** 30081 (NodePort)

  * **API:** 8001 (internal)

* **Connector Strategy:**

  * Source connectors for APIs, databases, files

  * Destination: PostgreSQL Bronze Layer

  * Custom connector development support

* Storage: Isolated PVCs for config, workspace, and temp data

## **4. dbt (Data Build Tool)**
**Purpose:** Transformations, testing, and documentation

* **Components:**

  * **dbt Core:** Transformation engine (fishtownanalytics/dbt-core:1.5+)

  * **dbt Scheduler:** Automated job runner (CronJob)

  * **dbt Docs Server:** Documentation site (optional)

* **Port:** 8080 (Documentation server)

* **Workflow:**

  * Pulls from Bronze Layer (raw data)

  * Transforms to Silver Layer (clean, modeled data)

  * Generates documentation and lineage graphs

  * Runs data quality tests

* **Features:**

  * SQL-based transformations

  * Modular, reusable models

  * Automated testing

  * Data lineage and documentation

  * Incremental model builds

## **5. Apache Superset (Business Intelligence)**
***Purpose:** Data visualization, dashboarding, and exploration

* **Components:**

  * **Superset Webapp:** UI and API (apache/superset:latest)

  * **Superset Worker:** Async queries and caching

  * **Redis:** Cache and message broker (redis:7-alpine)

  * **Database:** Metadata storage (postgres:13-alpine)

* **Ports:**

  * **Web UI:** 30082 (NodePort)

  * **API:** 8088 (internal)

* **Features:**

  * Connect to PostgreSQL Silver/Gold Layers

  * SQL Lab for ad-hoc queries
  
  * Interactive dashboards
  
  * Row-level security
  
  * Caching and performance optimization
  
  * Custom visualization plugins

## **6. ArgoCD (GitOps)**
**Purpose:** Continuous deployment and synchronization

**Image:** Official ArgoCD charts

**Port:** 9090 (Web UI via port-forward)

**Features:**

  * Multi-application management

  * Health status monitoring

  * Automated sync policies

  * Rollback capabilities

  * Resource hooks for pre/post deployment

## **Deployment Wave Plan**
```  
Wave -10 to 0: Infrastructure Foundation
├── Namespace creation
├── Network policies
├── Storage classes
└── Service accounts

Wave 1-5: Core Databases
├── PostgreSQL (data warehouse) - Wave 1
├── Airbyte metadata DB - Wave 2
├── Superset metadata DB - Wave 3
└── Redis cache - Wave 4

Wave 10-15: Data Ingestion & Management
├── Airbyte server - Wave 10
├── Airbyte worker - Wave 11
├── Airbyte webapp - Wave 12
└── pgAdmin - Wave 13

Wave 20-25: Data Transformation
├── dbt core service - Wave 20
├── dbt scheduler (CronJob) - Wave 21
├── dbt docs server - Wave 22 (optional)
└── Data quality monitor - Wave 23

Wave 30-35: Visualization & Consumption
├── Superset initialization job - Wave 30
├── Superset webapp - Wave 31
├── Superset worker - Wave 32
└── Superset flower (monitoring) - Wave 33

Wave 40+: Monitoring & Observability
├── Prometheus exporter - Wave 40
├── Grafana dashboards - Wave 41
└── Alert manager - Wave 42
```



## **Environmental Structure**

```
data-platform/
├── docker-compose.yml           # Local development
├── kubernetes/
│   ├── base/                   # Common configurations
│   │   ├── postgres/
│   │   ├── pgadmin/
│   │   ├── airbyte/
│   │   ├── dbt/
│   │   ├── superset/
│   │   └── monitoring/
│   └── overlays/              # Environment-specific
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   ├── patches/
│       │   ├── secrets/
│       │   └── wave-annotations.yaml
│       ├── staging/
│       └── prod/
├── dbt/
│   ├── models/
│   │   ├── staging/           # Bronze → Silver
│   │   ├── marts/            # Business models
│   │   └── intermediate/     # Reusable transformations
│   ├── seeds/                # Reference data
│   ├── tests/               # Data quality tests
│   ├── macros/              # Reusable SQL
│   └── docs/               # Documentation
├── superset/
│   ├── config/
│   ├── dashboards/          # Exported dashboards
│   ├── charts/             # Saved charts
│   └── assets/            # Custom visuals
└── scripts/
    ├── deployment/
    ├── backup/
    └── migration/
```

# **Networking Architecture**

Service Mesh
```
Kubernetes Services:
- postgres-bronze:5432     (Raw data)
- postgres-silver:5432     (Transformed data)
- postgres-gold:5432       (Aggregated data - future)
- airbyte-server:8001      (Airbyte API)
- airbyte-webapp:80        (Airbyte UI)
- superset:8088            (Superset API)
- superset-web:80          (Superset UI)
- dbt-docs:8080           (dbt documentation)

External Access (NodePort):
- pgAdmin:      http://localhost:30080
- Airbyte:      http://localhost:30081  
- Superset:     http://localhost:30082
- dbt Docs:     http://localhost:30083
- ArgoCD:       https://localhost:9090
```

# **Data Flow Security**
1. Network Policies: Restrict traffic between layers

2. Database Roles: Separate users per service

3. Secret Management: Environment-specific credentials

4. TLS/SSL: Encrypted connections between services

# **Storage Architecture**
# Multi-Layer Storage Strategy

```
PostgreSQL Storage (Tiered):
├── Bronze Layer (Raw): 50GB - High compression, frequent backups
├── Silver Layer (Cleaned): 100GB - Optimized for queries
└── Gold Layer (Aggregated): 20GB - SSD optimized, frequent access

Application Storage:
├── Airbyte:
│   ├── Config: 1GB (PVC)
│   ├── Workspace: 50GB (PVC) - Temporary processing
│   └── Temp: 10GB (emptyDir) - Ephemeral
├── dbt:
│   ├── Project: 500MB (ConfigMap) - Code
│   └── Profiles: 10MB (Secret) - Credentials
└── Superset:
    ├── Metadata: 5GB (PVC) - Dashboard definitions
    └── Cache: 2GB (emptyDir) - Redis data
```
