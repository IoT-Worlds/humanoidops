# Implementation Plan - HumanoidOps v0.1
**Technical Architecture & Implementation Strategy**

---

## Document Information

| Field | Value |
|-------|-------|
| **Project** | HumanoidOps |
| **Version** | 0.1 (MVP) |
| **Document Type** | Implementation Plan |
| **Status** | Review |
| **Created** | 2026-02-12 |
| **Last Updated** | 2026-02-12 |

---

## Table of Contents

1. [Architecture Overview]
2. [Technology Stack Decisions]
3. [System Architecture (C4 Model)]
4. [Component Design]
5. [Data Architecture]
6. [API Design]
7. [Security Architecture]
8. [Testing Strategy]
9. [Deployment Architecture]
10. [Implementation Phases]
11. [Architecture Decision Records]

---

## Architecture Overview

### Design Principles

**1. Security First**
- All communications encrypted by default
- Defense in depth (multiple security layers)
- Principle of least privilege (RBAC)
- Audit everything

**2. Scalability by Design**
- Stateless API servers (horizontal scaling)
- Time-series optimized database
- Asynchronous processing where possible
- Connection pooling and caching

**3. Operational Simplicity**
- Single command deployment (`docker-compose up`)
- Self-contained services
- Clear logging and metrics
- Graceful degradation

**4. Developer Experience**
- Clean APIs (RESTful, well-documented)
- Comprehensive SDKs
- Local development == production
- Fast feedback loops

### Architectural Style

**Microservices-lite (Modular Monolith for v0.1)**

For the MVP, we'll use a **modular monolith** approach:
- Separate services (API, MQTT Handler) but deployed together
- Clear module boundaries within services
- Shared database (acceptable for v0.1 scale)
- Easy to split into true microservices later (v0.2+)

**Rationale**: 
- Simpler deployment and debugging for MVP
- Faster development (no distributed system complexity)
- Can migrate to microservices when scaling requires it
- Lower operational overhead for users

---

## Technology Stack Decisions

### Backend: Go

**Choice**: Go 1.21+ with Gin framework

**Rationale**:
- **Performance**: Compiled language, efficient concurrency (goroutines)
- **Deployment**: Single binary, no runtime dependencies
- **MQTT Support**: Excellent MQTT client libraries
- **Ecosystem**: Rich library ecosystem for APIs
- **Type Safety**: Static typing reduces bugs
- **Community**: Strong robotics and IoT community

**Alternatives Considered**:
- **Python**: Slower performance, but easier for robotics developers
  - Decision: Use Python for robot agent, Go for backend
- **Node.js**: Good for real-time, but less type-safe
- **Rust**: Excellent performance, but steeper learning curve

**Go Framework**: Gin vs Fiber
- **Choice**: Gin
- **Rationale**: More mature, larger community, better documentation
- Fiber is faster but less ecosystem support

---

### Database: PostgreSQL + TimescaleDB

**Choice**: PostgreSQL 14+ with TimescaleDB extension

**Rationale**:
- **PostgreSQL**: Industry-standard, ACID compliance, excellent JSON support (JSONB)
- **TimescaleDB**: Purpose-built for time-series data (telemetry)
- **Performance**: Automatic partitioning, compression, continuous aggregates
- **Familiarity**: Most teams know PostgreSQL
- **Cost**: Open-source, no licensing fees

**TimescaleDB Benefits**:
- Hypertables for automatic time partitioning
- 10-100x faster queries on time-series data
- Automatic data retention policies
- Compression reduces storage by 90%

**Alternatives Considered**:
- **InfluxDB**: Purpose-built time-series DB, but adds complexity (two databases)
- **MongoDB**: Flexible schema, but less ACID guarantees
- **Cassandra**: Massive scale, but overkill for MVP

---

### Message Broker: EMQX (MQTT)

**Choice**: EMQX 5.0+ (open-source edition)

**Rationale**:
- **Scalability**: Supports 4M+ concurrent connections (single node)
- **MQTT 5.0**: Modern protocol with improved features
- **Clustering**: Easy horizontal scaling (future)
- **Security**: Built-in TLS, authentication, authorization
- **Dashboard**: Web UI for monitoring
- **Open Source**: Apache 2.0 license

**Alternatives Considered**:
- **Mosquitto**: Lightweight and simple, but limited scalability
  - Decision: Provide Mosquitto as alternative in docker-compose
- **RabbitMQ**: Excellent broker, but complex for MQTT use case
- **Kafka**: Overkill for real-time telemetry, more for event streaming

---

### Frontend: Next.js + TypeScript

**Choice**: Next.js 14+ (App Router) with TypeScript

**Rationale**:
- **React**: Industry standard for dashboards
- **TypeScript**: Type safety catches bugs early
- **Next.js**: Server-side rendering (SSR), routing, optimization
- **App Router**: Modern approach, better performance
- **Ecosystem**: Rich component libraries (shadcn/ui)
- **Developer Experience**: Hot reload, fast refresh

**Alternatives Considered**:
- **Vue.js**: Simpler learning curve, but smaller ecosystem
- **Svelte**: Excellent performance, but less mature
- **Plain React (Vite)**: More control, but need to configure everything

**UI Library**: shadcn/ui
- Radix UI primitives + Tailwind CSS
- Copy-paste components (not npm dependency)
- Full customization control

---

### Robot Agent: Python

**Choice**: Python 3.10+ with paho-mqtt

**Rationale**:
- **Robotics Standard**: Most robot software uses Python (ROS, etc.)
- **Ease of Use**: Simple integration for robotics engineers
- **Libraries**: Excellent MQTT, HTTP, data validation libraries
- **Cross-platform**: Works on embedded Linux, Raspberry Pi, etc.

**Key Libraries**:
- `paho-mqtt`: Industry standard MQTT client
- `pydantic`: Data validation and serialization
- `requests`: Simple HTTP client (fallback)

---

### Monitoring: Prometheus + Grafana

**Choice**: Prometheus for metrics, Grafana for visualization

**Rationale**:
- **Industry Standard**: De facto monitoring stack
- **Pull-based**: Metrics scraped from services
- **Time-series Native**: Ideal for operational metrics
- **Alerting**: Built-in alert manager (future)
- **Open Source**: No licensing costs

---

### Deployment: Docker + Docker Compose

**Choice**: Docker for containerization, Docker Compose for orchestration

**Rationale**:
- **Simplicity**: Single command to start entire stack
- **Consistency**: Dev == Staging == Prod environments
- **Portability**: Run anywhere Docker runs
- **Isolation**: Services in separate containers
- **No Kubernetes**: Overkill for MVP, adds complexity

Future: Kubernetes manifests for production scale (v0.2+)

---

## System Architecture (C4 Model)

### Level 1: System Context

```mermaid
C4Context
    title System Context - HumanoidOps Platform

    Person(operator, "Fleet Operator", "Monitors robot fleet health and responds to alerts")
    Person(admin, "System Admin", "Manages robots, users, and system configuration")
    Person(developer, "Robot Developer", "Integrates robots with platform")
    
    System(humanoidops, "HumanoidOps Platform", "Fleet management and monitoring platform")
    
    System_Ext(robot, "Humanoid Robot", "Physical robot sending telemetry")
    System_Ext(prometheus, "Prometheus", "Metrics collection and alerting")
    System_Ext(grafana, "Grafana", "Operational dashboards")
    
    Rel(operator, humanoidops, "Views fleet status, acknowledges alerts", "HTTPS")
    Rel(admin, humanoidops, "Manages robots and users", "HTTPS")
    Rel(developer, humanoidops, "Integrates robots via API", "HTTPS/MQTT")
    Rel(robot, humanoidops, "Sends telemetry data", "MQTT/HTTPS")
    Rel(prometheus, humanoidops, "Scrapes metrics", "HTTPS")
    Rel(grafana, prometheus, "Queries metrics", "PromQL")
```

**Key Interactions**:
1. **Robots → Platform**: Telemetry via MQTT (primary) or HTTPS (fallback)
2. **Users → Platform**: Dashboard access via web browser (HTTPS)
3. **Developers → Platform**: Robot registration and management via REST API
4. **Monitoring Systems → Platform**: Metrics scraping for operational visibility

---

### Level 2: Container Diagram

```mermaid
C4Container
    title Container Diagram - HumanoidOps Internal Architecture

    Person(user, "User", "Fleet operator or admin")
    System_Ext(robot, "Robot", "Sends telemetry")
    
    Container_Boundary(platform, "HumanoidOps Platform") {
        Container(dashboard, "Dashboard", "Next.js", "Web-based UI for fleet monitoring")
        Container(api, "API Service", "Go/Gin", "REST API for robot management")
        Container(mqtt_handler, "MQTT Handler", "Go", "Processes incoming telemetry")
        Container(emqx, "MQTT Broker", "EMQX", "Message broker for telemetry")
        ContainerDb(db, "Database", "PostgreSQL+TimescaleDB", "Stores robots, telemetry, alerts")
        Container(prometheus, "Prometheus", "Prometheus", "Metrics collection")
    }
    
    Rel(user, dashboard, "Views fleet", "HTTPS")
    Rel(dashboard, api, "Fetches data", "HTTPS/WebSocket")
    Rel(robot, emqx, "Publishes telemetry", "MQTT/TLS")
    Rel(robot, api, "Sends telemetry (fallback)", "HTTPS")
    Rel(emqx, mqtt_handler, "Delivers messages", "MQTT")
    Rel(mqtt_handler, db, "Writes telemetry", "SQL")
    Rel(api, db, "Reads/writes data", "SQL")
    Rel(prometheus, api, "Scrapes metrics", "HTTP")
    Rel(prometheus, mqtt_handler, "Scrapes metrics", "HTTP")
```

**Service Responsibilities**:

| Service | Responsibility | Technology | Port |
|---------|---------------|------------|------|
| **Dashboard** | User interface | Next.js | 3000 |
| **API Service** | REST API, WebSocket, business logic | Go/Gin | 8080 |
| **MQTT Handler** | Telemetry ingestion, alert evaluation | Go | (internal) |
| **MQTT Broker** | Message routing | EMQX | 1883/8883 |
| **Database** | Data persistence | PostgreSQL+TimescaleDB | 5432 |
| **Prometheus** | Metrics storage | Prometheus | 9090 |

---

### Level 3: Component Diagram (API Service)

```mermaid
C4Component
    title Component Diagram - API Service

    Container(dashboard, "Dashboard", "Next.js")
    ContainerDb(db, "Database", "PostgreSQL")
    
    Container_Boundary(api, "API Service") {
        Component(router, "HTTP Router", "Gin", "Routes requests to handlers")
        Component(auth_middleware, "Auth Middleware", "Go", "JWT validation, RBAC")
        Component(handlers, "HTTP Handlers", "Go", "Request/response logic")
        Component(services, "Business Logic", "Go", "Core domain logic")
        Component(repositories, "Data Access", "GORM", "Database queries")
        Component(ws_manager, "WebSocket Manager", "Go", "Real-time updates")
        Component(metrics, "Metrics Exporter", "Prometheus", "Exposes /metrics")
    }
    
    Rel(dashboard, router, "HTTP Requests")
    Rel(router, auth_middleware, "All requests")
    Rel(auth_middleware, handlers, "Authenticated requests")
    Rel(handlers, services, "Calls business logic")
    Rel(services, repositories, "Data operations")
    Rel(repositories, db, "SQL queries")
    Rel(ws_manager, dashboard, "Push updates")
    Rel(metrics, handlers, "Records metrics")
```

**Component Responsibilities**:

| Component | Responsibility | Files |
|-----------|---------------|-------|
| **HTTP Router** | Route matching, middleware chain | `main.go`, `routes.go` |
| **Auth Middleware** | JWT validation, RBAC enforcement | `middleware/auth.go` |
| **HTTP Handlers** | Request parsing, response formatting | `handlers/*.go` |
| **Business Logic** | Domain logic, validation | `services/*.go` |
| **Data Access** | Database queries, ORM | `repositories/*.go` |
| **WebSocket Manager** | Connection management, broadcasting | `websocket/manager.go` |
| **Metrics Exporter** | Prometheus metrics | `metrics/metrics.go` |

---

## Component Design

### Backend Components (Go)

#### 1. API Service Architecture

```
backend/
├── cmd/
│   └── api/
│       └── main.go                 # Entry point
├── internal/
│   ├── config/
│   │   └── config.go              # Configuration management
│   ├── middleware/
│   │   ├── auth.go                # Authentication middleware
│   │   ├── cors.go                # CORS middleware
│   │   ├── logger.go              # Request logging
│   │   └── ratelimit.go           # Rate limiting
│   ├── handlers/
│   │   ├── auth.go                # Authentication endpoints
│   │   ├── robots.go              # Robot management
│   │   ├── telemetry.go           # Telemetry endpoints
│   │   ├── alerts.go              # Alert management
│   │   └── audit.go               # Audit log queries
│   ├── services/
│   │   ├── auth_service.go        # Authentication logic
│   │   ├── robot_service.go       # Robot business logic
│   │   ├── alert_service.go       # Alert evaluation
│   │   └── audit_service.go       # Audit logging
│   ├── repositories/
│   │   ├── robot_repo.go          # Robot data access
│   │   ├── telemetry_repo.go      # Telemetry queries
│   │   ├── alert_repo.go          # Alert data access
│   │   └── user_repo.go           # User management
│   ├── models/
│   │   ├── robot.go               # Robot model
│   │   ├── telemetry.go           # Telemetry model
│   │   ├── alert.go               # Alert model
│   │   ├── user.go                # User model
│   │   └── audit.go               # Audit log model
│   ├── websocket/
│   │   ├── manager.go             # Connection manager
│   │   ├── client.go              # Client connection
│   │   └── message.go             # Message types
│   ├── database/
│   │   ├── database.go            # Connection setup
│   │   └── migrations.go          # Migration runner
│   └── metrics/
│       └── metrics.go             # Prometheus metrics
├── pkg/
│   ├── jwt/
│   │   └── jwt.go                 # JWT utilities
│   └── validator/
│       └── validator.go           # Input validation
├── migrations/
│   ├── 001_initial_schema.up.sql
│   ├── 001_initial_schema.down.sql
│   ├── 002_add_indexes.up.sql
│   └── 002_add_indexes.down.sql
└── go.mod
```

**Layered Architecture**:
```
┌─────────────────────────────────────┐
│      HTTP Layer (Handlers)          │  ← Request/Response
├─────────────────────────────────────┤
│   Business Logic (Services)         │  ← Domain Logic
├─────────────────────────────────────┤
│   Data Access (Repositories)        │  ← Database Queries
├─────────────────────────────────────┤
│      Database (PostgreSQL)          │  ← Persistence
└─────────────────────────────────────┘
```

---

#### 2. MQTT Handler Service Architecture

```
backend/
├── cmd/
│   └── mqtt-handler/
│       └── main.go                 # Entry point
├── internal/
│   ├── mqtt/
│   │   ├── client.go              # MQTT client wrapper
│   │   ├── handler.go             # Message handler
│   │   └── config.go              # MQTT configuration
│   ├── telemetry/
│   │   ├── processor.go           # Telemetry processing
│   │   └── validator.go           # Schema validation
│   ├── alert/
│   │   ├── engine.go              # Alert rule engine
│   │   ├── rules.go               # Alert rule definitions
│   │   └── notifier.go            # Alert notifications
│   └── database/
│       └── database.go            # Shared DB connection
```

**Processing Pipeline**:
```
MQTT Message
    ↓
Parse JSON
    ↓
Validate Schema
    ↓
Transform Data
    ↓
Write to Database
    ↓
Evaluate Alert Rules
    ↓
Trigger Alerts (if conditions met)
    ↓
Broadcast via WebSocket
```

---

### Frontend Components (Next.js)

#### Application Structure

```
dashboard/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx           # Login page
│   │   └── layout.tsx             # Auth layout
│   ├── (dashboard)/
│   │   ├── fleet/
│   │   │   └── page.tsx           # Fleet overview
│   │   ├── robots/
│   │   │   └── [id]/
│   │   │       └── page.tsx       # Robot detail
│   │   ├── alerts/
│   │   │   └── page.tsx           # Alerts page
│   │   ├── audit/
│   │   │   └── page.tsx           # Audit logs
│   │   └── layout.tsx             # Dashboard layout
│   ├── api/
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.ts       # Auth API routes
│   └── layout.tsx                 # Root layout
├── components/
│   ├── ui/                        # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── table.tsx
│   │   └── ...
│   ├── fleet/
│   │   ├── RobotCard.tsx          # Robot status card
│   │   ├── FleetSummary.tsx       # Summary stats
│   │   └── StatusFilter.tsx       # Filter controls
│   ├── charts/
│   │   ├── BatteryChart.tsx       # Battery line chart
│   │   ├── TemperatureChart.tsx   # Temperature chart
│   │   └── StatusTimeline.tsx     # Status swimlane
│   ├── alerts/
│   │   ├── AlertList.tsx          # Alert table
│   │   └── AlertCard.tsx          # Alert item
│   └── layout/
│       ├── Header.tsx             # Top navigation
│       ├── Sidebar.tsx            # Side navigation
│       └── Footer.tsx             # Footer
├── lib/
│   ├── api.ts                     # API client
│   ├── websocket.ts               # WebSocket hook
│   ├── auth.ts                    # Auth utilities
│   └── utils.ts                   # Helper functions
├── hooks/
│   ├── useRobots.ts               # Robot data hook
│   ├── useAlerts.ts               # Alerts data hook
│   └── useWebSocket.ts            # WebSocket hook
├── types/
│   ├── robot.ts                   # Robot types
│   ├── telemetry.ts               # Telemetry types
│   └── alert.ts                   # Alert types
└── styles/
    └── globals.css                # Global styles
```

**Component Hierarchy**:
```
App Layout
├── Auth Layout (login)
│   └── Login Page
└── Dashboard Layout
    ├── Header (nav, user menu)
    ├── Sidebar (navigation)
    ├── Main Content
    │   ├── Fleet Page
    │   │   ├── Fleet Summary
    │   │   ├── Status Filter
    │   │   └── Robot Grid
    │   │       └── Robot Card (×N)
    │   ├── Robot Detail Page
    │   │   ├── Robot Header
    │   │   ├── Tabs
    │   │   ├── Telemetry Charts
    │   │   └── Alert List
    │   └── Alerts Page
    │       └── Alert List
    └── Footer
```

---

### Robot Agent (Python)

#### SDK Structure

```
agent/
├── humanoidops_agent/
│   ├── __init__.py
│   ├── client.py                  # Main SDK client
│   ├── mqtt_transport.py          # MQTT transport
│   ├── http_transport.py          # HTTP fallback
│   ├── queue.py                   # Offline message queue
│   ├── models.py                  # Pydantic models
│   ├── simulator.py               # Robot simulator
│   ├── cli.py                     # CLI tool
│   ├── config.py                  # Configuration
│   └── exceptions.py              # Custom exceptions
├── examples/
│   ├── basic_integration.py       # Simple example
│   ├── with_config.py             # Using config file
│   └── simulator_fleet.py         # Fleet simulation
├── tests/
│   ├── test_client.py
│   ├── test_mqtt.py
│   └── test_simulator.py
├── pyproject.toml
├── README.md
└── LICENSE
```

**Client Usage**:
```python
from humanoidops_agent import HumanoidOpsClient

# Initialize client
client = HumanoidOpsClient(
    robot_id="550e8400-e29b-41d4-a716-446655440000",
    api_key="secret_key",
    mqtt_broker="mqtt://broker.example.com:1883",
    # Falls back to HTTPS if MQTT unavailable
    api_url="https://api.example.com/v1"
)

# Send telemetry (auto-retries, queues if offline)
client.send_telemetry(
    battery_level=85.5,
    status="working",
    location={"x": 12.5, "y": 8.3, "z": 0.0},
    metrics={
        "cpu_temp": 65.2,
        "cpu_usage": 45.8,
        "memory_usage": 60.3,
        "joint_health": [98, 97, 99, 100, 95, 98],
        "error_codes": []
    }
)
```

---

## Data Architecture

### Database Schema

#### Entity Relationship Diagram

```mermaid
erDiagram
    USERS ||--o{ ROBOTS : registers
    USERS ||--o{ AUDIT_LOGS : performs
    USERS ||--o{ ALERTS : acknowledges
    ROBOTS ||--o{ TELEMETRY : generates
    ROBOTS ||--o{ ALERTS : triggers
    
    USERS {
        uuid id PK
        varchar username UK
        varchar email UK
        varchar password_hash
        varchar role
        timestamp created_at
        timestamp updated_at
    }
    
    ROBOTS {
        uuid id PK
        varchar serial_number UK
        varchar model
        varchar manufacturer
        varchar firmware_version
        varchar location_name
        jsonb location_coordinates
        varchar status
        timestamp registered_at
        uuid registered_by FK
        timestamp last_seen
    }
    
    TELEMETRY {
        uuid robot_id FK
        timestamptz timestamp
        float battery_level
        varchar status
        jsonb location
        jsonb metrics
        timestamptz server_timestamp
    }
    
    ALERTS {
        uuid id PK
        uuid robot_id FK
        varchar type
        varchar severity
        text message
        timestamp triggered_at
        boolean acknowledged
        uuid acknowledged_by FK
        timestamp acknowledged_at
    }
    
    AUDIT_LOGS {
        uuid id PK
        timestamptz timestamp
        uuid user_id FK
        varchar action
        varchar resource_type
        uuid resource_id
        varchar result
        inet ip_address
        text user_agent
        jsonb details
    }
```

---

#### Table Definitions

**1. Users Table**

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'viewer',
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT users_role_check CHECK (role IN ('admin', 'operator', 'viewer'))
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
```

**Purpose**: Store user accounts with authentication and authorization data.

**Columns**:
- `id`: Primary key (UUID for distributed systems)
- `username`: Login identifier (unique)
- `email`: Email address (unique, for password reset future)
- `password_hash`: bcrypt hash (cost factor 12)
- `role`: RBAC role (admin | operator | viewer)
- `created_at`, `updated_at`: Audit timestamps

---

**2. Robots Table**

```sql
CREATE TABLE robots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    serial_number VARCHAR(100) UNIQUE NOT NULL,
    model VARCHAR(100) NOT NULL,
    manufacturer VARCHAR(100),
    firmware_version VARCHAR(50),
    location_name VARCHAR(255),
    location_coordinates JSONB,
    status VARCHAR(50) NOT NULL DEFAULT 'active',
    registered_at TIMESTAMP NOT NULL DEFAULT NOW(),
    registered_by UUID REFERENCES users(id) ON DELETE SET NULL,
    last_seen TIMESTAMP,
    
    CONSTRAINT robots_status_check CHECK (status IN ('active', 'inactive', 'maintenance'))
);

CREATE INDEX idx_robots_serial ON robots(serial_number);
CREATE INDEX idx_robots_status ON robots(status);
CREATE INDEX idx_robots_model ON robots(model);
CREATE INDEX idx_robots_last_seen ON robots(last_seen DESC);

-- GiST index for location queries (future)
CREATE INDEX idx_robots_location ON robots USING GiST (location_coordinates);
```

**Purpose**: Store robot metadata and registration information.

**Columns**:
- `serial_number`: Manufacturer serial number (unique identifier)
- `model`: Robot model/type (for model-specific alert thresholds)
- `location_coordinates`: JSONB `{"lat": 37.7749, "lon": -122.4194}`
- `status`: Registration status (not operational status)
- `last_seen`: Most recent telemetry timestamp (updated on every message)

---

**3. Telemetry Table (Hypertable)**

```sql
CREATE TABLE telemetry (
    robot_id UUID NOT NULL,
    timestamp TIMESTAMPTZ NOT NULL,
    battery_level DOUBLE PRECISION,
    status VARCHAR(50),
    location JSONB,
    metrics JSONB,
    server_timestamp TIMESTAMPTZ DEFAULT NOW(),
    
    CONSTRAINT fk_robot FOREIGN KEY (robot_id) REFERENCES robots(id) ON DELETE CASCADE
);

-- Convert to TimescaleDB hypertable
SELECT create_hypertable('telemetry', 'timestamp');

-- Indexes
CREATE INDEX idx_telemetry_robot_time ON telemetry(robot_id, timestamp DESC);
CREATE INDEX idx_telemetry_status ON telemetry(status) WHERE status IN ('error', 'offline');

-- Retention policy (30 days)
SELECT add_retention_policy('telemetry', INTERVAL '30 days');

-- Compression policy (compress after 7 days)
ALTER TABLE telemetry SET (
    timescaledb.compress,
    timescaledb.compress_segmentby = 'robot_id'
);

SELECT add_compression_policy('telemetry', INTERVAL '7 days');
```

**Purpose**: Store time-series telemetry data from robots.

**TimescaleDB Features**:
- **Hypertable**: Automatically partitions by time (1-week chunks)
- **Compression**: 90% storage reduction after 7 days
- **Retention**: Auto-delete data older than 30 days
- **Fast queries**: Optimized for time-range queries

**JSONB Columns**:
- `location`: `{"x": 12.5, "y": 8.3, "z": 0.0}`
- `metrics`: Flexible schema for manufacturer-specific data
  ```json
  {
    "cpu_temp": 65.2,
    "cpu_usage": 45.8,
    "memory_usage": 60.3,
    "joint_health": [98, 97, 99, 100, 95, 98],
    "error_codes": ["ERR_101"],
    "network_latency": 45,
    "custom": { /* manufacturer-specific */ }
  }
  ```

---

**4. Alerts Table**

```sql
CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    robot_id UUID NOT NULL REFERENCES robots(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL,
    severity VARCHAR(20) NOT NULL,
    message TEXT NOT NULL,
    triggered_at TIMESTAMP NOT NULL DEFAULT NOW(),
    acknowledged BOOLEAN DEFAULT FALSE,
    acknowledged_by UUID REFERENCES users(id) ON DELETE SET NULL,
    acknowledged_at TIMESTAMP,
    
    CONSTRAINT alerts_severity_check CHECK (severity IN ('info', 'warning', 'critical')),
    CONSTRAINT alerts_type_check CHECK (type IN ('robot_offline', 'low_battery', 'error_status', 'high_temperature', 'high_latency'))
);

CREATE INDEX idx_alerts_robot ON alerts(robot_id);
CREATE INDEX idx_alerts_triggered ON alerts(triggered_at DESC);
CREATE INDEX idx_alerts_acknowledged ON alerts(acknowledged) WHERE NOT acknowledged;
CREATE INDEX idx_alerts_severity ON alerts(severity) WHERE severity = 'critical';
```

**Purpose**: Store triggered alerts with acknowledgment tracking.

**Alert Types**:
- `robot_offline`: No telemetry for >30s
- `low_battery`: Battery <20% (warning) or <10% (critical)
- `error_status`: Robot reports status='error'
- `high_temperature`: CPU >80°C
- `high_latency`: Network latency >1000ms

---

**5. Audit Logs Table**

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    result VARCHAR(20) NOT NULL,
    ip_address INET,
    user_agent TEXT,
    details JSONB,
    
    CONSTRAINT audit_result_check CHECK (result IN ('success', 'failure'))
);

CREATE INDEX idx_audit_timestamp ON audit_logs(timestamp DESC);
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
CREATE INDEX idx_audit_result ON audit_logs(result) WHERE result = 'failure';

-- Retention: Keep for 90 days
-- Note: Implement as application logic or cron job
```

**Purpose**: Immutable audit trail for security and compliance.

**Actions Logged**:
- `auth.login`, `auth.logout`, `auth.refresh`
- `robot.create`, `robot.update`, `robot.delete`
- `alert.acknowledge`
- `user.create`, `user.update`, `user.delete` (admin only)

---

#### Data Access Patterns

**Common Queries**:

1. **Get Robot List** (Dashboard)
```sql
SELECT id, serial_number, model, status, last_seen, 
       CASE 
           WHEN last_seen IS NULL THEN 'unknown'
           WHEN last_seen < NOW() - INTERVAL '30 seconds' THEN 'offline'
           ELSE 'online'
       END as operational_status
FROM robots
WHERE status = 'active'
ORDER BY last_seen DESC
LIMIT 20 OFFSET ?;
```

2. **Get Recent Telemetry** (Robot Detail)
```sql
SELECT timestamp, battery_level, status, metrics
FROM telemetry
WHERE robot_id = ?
  AND timestamp >= NOW() - INTERVAL '24 hours'
ORDER BY timestamp DESC;
```

3. **Get Active Alerts** (Dashboard)
```sql
SELECT a.*, r.serial_number, r.model
FROM alerts a
JOIN robots r ON a.robot_id = r.id
WHERE a.acknowledged = false
ORDER BY a.severity DESC, a.triggered_at DESC;
```

4. **Audit Log Query** (Security Review)
```sql
SELECT a.*, u.username
FROM audit_logs a
LEFT JOIN users u ON a.user_id = u.id
WHERE a.timestamp >= ?
  AND a.action LIKE 'robot.%'
ORDER BY a.timestamp DESC;
```

---

### Data Retention & Archival

**Telemetry Data**:
- **Hot data** (7 days): Uncompressed, fast queries
- **Warm data** (8-30 days): Compressed, slightly slower queries
- **Cold data** (30+ days): Deleted (or archived to S3 in future)

**Audit Logs**:
- **Retention**: 90 days minimum
- **Compression**: After 30 days (TimescaleDB)
- **Archival**: Export to S3 before deletion (future)

**Implementation**:
```sql
-- TimescaleDB policies (already shown above)
SELECT add_retention_policy('telemetry', INTERVAL '30 days');
SELECT add_compression_policy('telemetry', INTERVAL '7 days');

-- For audit_logs (future: convert to hypertable)
-- Manual cleanup for now:
DELETE FROM audit_logs WHERE timestamp < NOW() - INTERVAL '90 days';
```

---

## API Design

### REST API Specification

#### Base URL
```
Production:  https://api.humanoidops.com/v1
Development: http://localhost:8080/api/v1
```

#### Authentication
All endpoints (except `/health`, `/metrics`, `/openapi`) require JWT authentication:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Standard Response Format

**Success Response**:
```json
{
  "data": { ... },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req-abc-123"
  }
}
```

**Error Response**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid robot serial number format",
    "details": {
      "field": "serial_number",
      "constraint": "alphanumeric"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req-abc-123"
  }
}
```

**HTTP Status Codes**:
- `200 OK`: Success
- `201 Created`: Resource created
- `204 No Content`: Success with no response body
- `400 Bad Request`: Validation error
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource doesn't exist
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

---

### API Endpoints

#### Authentication Endpoints

**1. Login**

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "securepassword123"
}

Response 200 OK:
{
  "data": {
    "token": "eyJhbGc...",
    "refresh_token": "refresh_xyz...",
    "expires_at": "2024-01-16T10:30:00Z",
    "user": {
      "id": "user-uuid",
      "username": "admin",
      "email": "admin@example.com",
      "role": "admin"
    }
  }
}
```

**2. Refresh Token**

```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "refresh_xyz..."
}

Response 200 OK:
{
  "data": {
    "token": "eyJhbGc...",
    "expires_at": "2024-01-16T10:30:00Z"
  }
}
```

**3. Logout**

```http
POST /api/v1/auth/logout
Authorization: Bearer {token}

Response 204 No Content
```

---

#### Robot Management Endpoints

**1. Register Robot**

```http
POST /api/v1/robots
Authorization: Bearer {token}
Content-Type: application/json

{
  "serial_number": "HV1-00123",
  "model": "HumanoidV1",
  "manufacturer": "RoboCorp",
  "firmware_version": "1.2.3",
  "location_name": "Warehouse A",
  "location_coordinates": {
    "lat": 37.7749,
    "lon": -122.4194
  }
}

Response 201 Created:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "serial_number": "HV1-00123",
    "model": "HumanoidV1",
    "manufacturer": "RoboCorp",
    "firmware_version": "1.2.3",
    "location_name": "Warehouse A",
    "location_coordinates": {
      "lat": 37.7749,
      "lon": -122.4194
    },
    "status": "active",
    "registered_at": "2024-01-15T10:30:00Z",
    "registered_by": "admin-user-id",
    "api_key": "secret_key_abc123..."
  }
}
```

**Authorization**: Admin only

---

**2. List Robots**

```http
GET /api/v1/robots?status=active&model=HumanoidV1&page=1&limit=20
Authorization: Bearer {token}

Response 200 OK:
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "serial_number": "HV1-00123",
      "model": "HumanoidV1",
      "status": "active",
      "last_seen": "2024-01-15T10:29:55Z",
      "operational_status": "online"
    },
    // ... more robots
  ],
  "meta": {
    "total": 125,
    "page": 1,
    "limit": 20,
    "total_pages": 7
  }
}
```

**Query Parameters**:
- `status`: Filter by registration status (active, inactive, maintenance)
- `model`: Filter by model name
- `location`: Filter by location name
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)

**Authorization**: All authenticated users

---

**3. Get Robot Details**

```http
GET /api/v1/robots/{robot_id}
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "serial_number": "HV1-00123",
    "model": "HumanoidV1",
    "manufacturer": "RoboCorp",
    "firmware_version": "1.2.3",
    "location_name": "Warehouse A",
    "location_coordinates": {
      "lat": 37.7749,
      "lon": -122.4194
    },
    "status": "active",
    "registered_at": "2024-01-15T10:30:00Z",
    "registered_by": "admin-user-id",
    "last_seen": "2024-01-15T10:29:55Z",
    "latest_telemetry": {
      "battery_level": 85.5,
      "status": "working",
      "timestamp": "2024-01-15T10:29:55Z"
    }
  }
}
```

**Authorization**: All authenticated users

---

**4. Update Robot**

```http
PATCH /api/v1/robots/{robot_id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "location_name": "Warehouse B",
  "firmware_version": "1.2.4"
}

Response 200 OK:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    // ... updated robot data
  }
}
```

**Authorization**: Admin only

---

**5. Deactivate Robot**

```http
DELETE /api/v1/robots/{robot_id}
Authorization: Bearer {token}

Response 204 No Content
```

**Note**: This is a soft delete (status → inactive). Telemetry history preserved.

**Authorization**: Admin only

---

#### Telemetry Endpoints

**1. Submit Telemetry (HTTPS Fallback)**

```http
POST /api/v1/telemetry
Authorization: Bearer {robot_api_key}
Content-Type: application/json

{
  "robot_id": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-15T10:35:45Z",
  "battery_level": 85.5,
  "status": "working",
  "location": {
    "x": 12.5,
    "y": 8.3,
    "z": 0.0
  },
  "metrics": {
    "cpu_temp": 65.2,
    "cpu_usage": 45.8,
    "memory_usage": 60.3,
    "joint_health": [98, 97, 99, 100, 95, 98],
    "error_codes": [],
    "network_latency": 45
  }
}

Response 201 Created:
{
  "data": {
    "received_at": "2024-01-15T10:35:45.123Z",
    "status": "processed"
  }
}
```

**Rate Limit**: 10 req/s per robot

**Authorization**: Robot API key

---

**2. Get Robot Telemetry History**

```http
GET /api/v1/robots/{robot_id}/telemetry?start=2024-01-14T00:00:00Z&end=2024-01-15T00:00:00Z&limit=1000
Authorization: Bearer {token}

Response 200 OK:
{
  "data": [
    {
      "timestamp": "2024-01-15T10:35:45Z",
      "battery_level": 85.5,
      "status": "working",
      "location": {"x": 12.5, "y": 8.3, "z": 0.0},
      "metrics": { ... }
    },
    // ... more telemetry points
  ],
  "meta": {
    "total": 86400,  // 24h at 1Hz
    "returned": 1000,
    "start": "2024-01-14T00:00:00Z",
    "end": "2024-01-15T00:00:00Z"
  }
}
```

**Query Parameters**:
- `start`: Start timestamp (ISO 8601)
- `end`: End timestamp (ISO 8601)
- `limit`: Max records (default: 1000, max: 10000)

**Authorization**: All authenticated users

---

#### Alert Endpoints

**1. List Alerts**

```http
GET /api/v1/alerts?acknowledged=false&severity=critical&robot_id={uuid}&page=1&limit=20
Authorization: Bearer {token}

Response 200 OK:
{
  "data": [
    {
      "id": "alert-uuid",
      "robot_id": "robot-uuid",
      "robot_serial": "HV1-00123",
      "type": "robot_offline",
      "severity": "critical",
      "message": "Robot has been offline for 45 seconds",
      "triggered_at": "2024-01-15T10:30:00Z",
      "acknowledged": false,
      "acknowledged_by": null,
      "acknowledged_at": null
    },
    // ... more alerts
  ],
  "meta": {
    "total": 7,
    "page": 1,
    "limit": 20
  }
}
```

**Query Parameters**:
- `acknowledged`: Filter by acknowledgment status (true, false, all)
- `severity`: Filter by severity (critical, warning, info, all)
- `robot_id`: Filter by robot
- `start`, `end`: Time range filter
- `page`, `limit`: Pagination

**Authorization**: All authenticated users

---

**2. Get Alert Details**

```http
GET /api/v1/alerts/{alert_id}
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "id": "alert-uuid",
    "robot_id": "robot-uuid",
    "robot": {
      "serial_number": "HV1-00123",
      "model": "HumanoidV1"
    },
    "type": "robot_offline",
    "severity": "critical",
    "message": "Robot has been offline for 45 seconds",
    "triggered_at": "2024-01-15T10:30:00Z",
    "acknowledged": false,
    "acknowledged_by": null,
    "acknowledged_at": null
  }
}
```

**Authorization**: All authenticated users

---

**3. Acknowledge Alert**

```http
POST /api/v1/alerts/{alert_id}/acknowledge
Authorization: Bearer {token}
Content-Type: application/json

{
  "notes": "Investigating connectivity issue"  // optional
}

Response 200 OK:
{
  "data": {
    "id": "alert-uuid",
    "acknowledged": true,
    "acknowledged_by": "user-uuid",
    "acknowledged_at": "2024-01-15T10:35:00Z"
  }
}
```

**Authorization**: Operator or Admin

---

#### Audit Log Endpoints

**1. Query Audit Logs**

```http
GET /api/v1/audit-logs?start=2024-01-14T00:00:00Z&action=robot.create&user_id={uuid}&result=success&page=1&limit=50
Authorization: Bearer {token}

Response 200 OK:
{
  "data": [
    {
      "id": "audit-uuid",
      "timestamp": "2024-01-15T10:30:00Z",
      "user_id": "user-uuid",
      "username": "admin",
      "action": "robot.create",
      "resource_type": "robot",
      "resource_id": "robot-uuid",
      "result": "success",
      "ip_address": "192.168.1.100",
      "user_agent": "curl/7.68.0",
      "details": {
        "serial_number": "HV1-00123",
        "model": "HumanoidV1"
      }
    },
    // ... more logs
  ],
  "meta": {
    "total": 1523,
    "page": 1,
    "limit": 50
  }
}
```

**Query Parameters**:
- `start`, `end`: Time range
- `user_id`: Filter by user
- `action`: Filter by action (e.g., `robot.create`, `alert.acknowledge`)
- `result`: Filter by result (success, failure)
- `page`, `limit`: Pagination

**Authorization**: Admin or Operator

---

#### System Endpoints

**1. Health Check**

```http
GET /api/v1/health

Response 200 OK:
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z",
  "version": "0.1.0",
  "checks": {
    "database": "healthy",
    "mqtt": "healthy"
  }
}
```

**No authentication required**

---

**2. Prometheus Metrics**

```http
GET /api/v1/metrics

Response 200 OK (Prometheus format):
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",endpoint="/api/v1/robots",status="200"} 1523

# HELP telemetry_messages_received_total Total telemetry messages
# TYPE telemetry_messages_received_total counter
telemetry_messages_received_total{robot_id="..."} 86400

# ... more metrics
```

**No authentication required** (but should be behind firewall in production)

---

**3. OpenAPI Specification**

```http
GET /api/v1/openapi

Response 200 OK:
openapi: 3.0.3
info:
  title: HumanoidOps API
  version: 0.1.0
  description: Fleet management API for humanoid robots
...
```

**No authentication required**

---

### WebSocket Protocol

**Connection**:
```javascript
const ws = new WebSocket('ws://localhost:8080/api/v1/ws?token={jwt}');
```

**Message Types**:

1. **Telemetry Update**
```json
{
  "type": "telemetry_update",
  "robot_id": "550e8400-e29b-41d4-a716-446655440000",
  "data": {
    "battery_level": 85.5,
    "status": "working",
    "timestamp": "2024-01-15T10:35:45Z"
  }
}
```

2. **Status Change**
```json
{
  "type": "status_change",
  "robot_id": "550e8400-e29b-41d4-a716-446655440000",
  "old_status": "online",
  "new_status": "offline",
  "timestamp": "2024-01-15T10:35:45Z"
}
```

3. **Alert Triggered**
```json
{
  "type": "alert_triggered",
  "alert": {
    "id": "alert-uuid",
    "robot_id": "robot-uuid",
    "type": "robot_offline",
    "severity": "critical",
    "message": "Robot has been offline for 45 seconds",
    "triggered_at": "2024-01-15T10:35:45Z"
  }
}
```

4. **Alert Acknowledged**
```json
{
  "type": "alert_acknowledged",
  "alert_id": "alert-uuid",
  "acknowledged_by": "user-uuid",
  "acknowledged_at": "2024-01-15T10:36:00Z"
}
```

---

## Security Architecture

### Defense in Depth Strategy

```
┌─────────────────────────────────────────┐
│  Layer 1: Network (TLS, Firewall)       │
├─────────────────────────────────────────┤
│  Layer 2: Authentication (JWT, API Keys)│
├─────────────────────────────────────────┤
│  Layer 3: Authorization (RBAC)          │
├─────────────────────────────────────────┤
│  Layer 4: Input Validation              │
├─────────────────────────────────────────┤
│  Layer 5: Audit Logging                 │
└─────────────────────────────────────────┘
```

### 1. Transport Security

**TLS Configuration**:
```yaml
# Nginx/Traefik configuration
ssl_protocols: TLSv1.3 TLSv1.2
ssl_ciphers: ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
ssl_prefer_server_ciphers: on
ssl_session_cache: shared:SSL:10m
ssl_session_tickets: off
ssl_stapling: on
ssl_stapling_verify: on
```

**MQTT TLS**:
- Port 8883 (MQTT over TLS)
- Port 1883 disabled in production
- Client certificates optional (mTLS)

### 2. Authentication

**JWT Token Structure**:
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "user_id": "uuid",
    "username": "admin",
    "role": "admin",
    "iat": 1610000000,
    "exp": 1610086400  // 24 hours
  },
  "signature": "..."
}
```

**Implementation** (Go):
```go
// Generate JWT
func generateJWT(user *models.User) (string, error) {
    secret := []byte(os.Getenv("JWT_SECRET"))
    
    claims := jwt.MapClaims{
        "user_id": user.ID.String(),
        "username": user.Username,
        "role": user.Role,
        "iat": time.Now().Unix(),
        "exp": time.Now().Add(24 * time.Hour).Unix(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secret)
}

// Validate JWT
func validateJWT(tokenString string) (*jwt.Token, error) {
    secret := []byte(os.Getenv("JWT_SECRET"))
    
    return jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method")
        }
        return secret, nil
    })
}
```

**Password Hashing** (bcrypt):
```go
import "golang.org/x/crypto/bcrypt"

func hashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), 12) // cost factor 12
    return string(bytes), err
}

func checkPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}
```

### 3. Authorization (RBAC)

**Role Permissions Matrix**:

| Endpoint | Admin | Operator | Viewer |
|----------|-------|----------|--------|
| `POST /api/v1/robots` | ✅ | ❌ | ❌ |
| `GET /api/v1/robots` | ✅ | ✅ | ✅ |
| `PATCH /api/v1/robots/*` | ✅ | ❌ | ❌ |
| `DELETE /api/v1/robots/*` | ✅ | ❌ | ❌ |
| `GET /api/v1/telemetry` | ✅ | ✅ | ✅ |
| `GET /api/v1/alerts` | ✅ | ✅ | ✅ |
| `POST /api/v1/alerts/*/acknowledge` | ✅ | ✅ | ❌ |
| `GET /api/v1/audit-logs` | ✅ | ✅ | ❌ |

**Middleware Implementation**:
```go
func RequireRole(allowedRoles ...string) gin.HandlerFunc {
    return func(c *gin.Context) {
        userRole := c.GetString("role")
        
        for _, role := range allowedRoles {
            if userRole == role {
                c.Next()
                return
            }
        }
        
        c.JSON(403, gin.H{"error": "Insufficient permissions"})
        c.Abort()
    }
}

// Usage
router.POST("/robots", 
    middleware.AuthRequired(),
    middleware.RequireRole("admin"),
    handlers.CreateRobot)
```

### 4. Input Validation

**Validation Strategy**:
1. **Schema validation**: Use struct tags in Go
2. **Sanitization**: Strip dangerous characters
3. **Type checking**: Strict type enforcement
4. **Range checking**: Validate numeric ranges

**Example** (Go):
```go
type CreateRobotRequest struct {
    SerialNumber    string  `json:"serial_number" binding:"required,alphanum,max=100"`
    Model           string  `json:"model" binding:"required,max=100"`
    Manufacturer    string  `json:"manufacturer" binding:"max=100"`
    FirmwareVersion string  `json:"firmware_version" binding:"omitempty,semver"`
}

func (r *CreateRobotRequest) Validate() error {
    // Additional custom validation
    if !isValidSerialNumber(r.SerialNumber) {
        return errors.New("invalid serial number format")
    }
    return nil
}
```

### 5. Rate Limiting

**Implementation** (using golang.org/x/time/rate):
```go
import "golang.org/x/time/rate"

var loginLimiter = rate.NewLimiter(rate.Every(12*time.Second), 5) // 5 req per minute

func RateLimitMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        if !loginLimiter.Allow() {
            c.JSON(429, gin.H{"error": "Too many requests"})
            c.Abort()
            return
        }
        c.Next()
    }
}
```

**Rate Limits**:
- Login endpoint: 5 req/min per IP
- API endpoints: 1000 req/min per user
- Telemetry endpoint: 10 req/sec per robot

### 6. Security Headers

**Implementation** (Gin middleware):
```go
func SecurityHeadersMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("X-Frame-Options", "DENY")
        c.Header("X-Content-Type-Options", "nosniff")
        c.Header("X-XSS-Protection", "1; mode=block")
        c.Header("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        c.Header("Content-Security-Policy", "default-src 'self'")
        c.Next()
    }
}
```

### 7. Secrets Management

**Environment Variables** (12-factor app):
```bash
# .env (never commit to Git)
DATABASE_URL=postgresql://user:pass@localhost:5432/db
JWT_SECRET=your-secret-key-here-min-32-chars
MQTT_PASSWORD=mqtt-password
API_KEY_SALT=random-salt-for-api-keys
```

**Docker Secrets** (production):
```yaml
# docker-compose.yml (production)
version: '3.8'
services:
  api:
    secrets:
      - jwt_secret
      - db_password
      
secrets:
  jwt_secret:
    external: true
  db_password:
    external: true
```

---

## Testing Strategy

### Test Pyramid

```
        ┌─────────────┐
        │     E2E     │  ← 10% (Full system tests)
        │     Tests   │
        ├─────────────┤
        │Integration  │  ← 25% (Service + DB tests)
        │   Tests     │
        ├─────────────┤
        │    Unit     │  ← 65% (Function-level tests)
        │   Tests     │
        └─────────────┘
```

### 1. Unit Tests

**Coverage Target**: >80% for backend, >70% for frontend

**Go Unit Tests** (using testify):
```go
// backend/internal/services/robot_service_test.go
package services

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

func TestRobotService_CreateRobot(t *testing.T) {
    // Arrange
    mockRepo := new(MockRobotRepository)
    service := NewRobotService(mockRepo)
    
    robot := &models.Robot{
        SerialNumber: "HV1-00123",
        Model: "HumanoidV1",
    }
    
    mockRepo.On("Create", mock.Anything).Return(nil)
    
    // Act
    err := service.CreateRobot(robot)
    
    // Assert
    assert.NoError(t, err)
    mockRepo.AssertExpectations(t)
}
```

**Python Unit Tests** (using pytest):
```python
# agent/tests/test_simulator.py
import pytest
from humanoidops_agent.simulator import RobotSimulator

def test_simulator_initialization():
    robot = RobotSimulator(model="TestBot")
    assert robot.state.status == "idle"
    assert robot.state.battery_level == 100.0

def test_battery_drain():
    robot = RobotSimulator()
    initial_battery = robot.state.battery_level
    robot.state.status = "working"
    robot.update(delta_time=10.0)
    assert robot.state.battery_level < initial_battery
```

**React Unit Tests** (using Jest + React Testing Library):
```typescript
// dashboard/components/RobotCard.test.tsx
import { render, screen } from '@testing-library/react';
import RobotCard from './RobotCard';

test('renders robot status correctly', () => {
  const robot = {
    id: '123',
    serial_number: 'HV1-00123',
    status: 'online',
    battery_level: 85.5,
  };
  
  render(<RobotCard robot={robot} />);
  
  expect(screen.getByText('HV1-00123')).toBeInTheDocument();
  expect(screen.getByText('85%')).toBeInTheDocument();
});
```

---

### 2. Integration Tests

**Coverage Target**: Key workflows and database interactions

**Go Integration Tests** (with real database):
```go
// backend/tests/integration/robot_test.go
package integration

import (
    "testing"
    "github.com/stretchr/testify/suite"
)

type RobotIntegrationTestSuite struct {
    suite.Suite
    db *gorm.DB
}

func (suite *RobotIntegrationTestSuite) SetupTest() {
    // Setup test database
    suite.db = setupTestDB()
}

func (suite *RobotIntegrationTestSuite) TearDownTest() {
    // Cleanup
    cleanupTestDB(suite.db)
}

func (suite *RobotIntegrationTestSuite) TestCreateAndRetrieveRobot() {
    // Create robot via API
    resp := makeRequest("POST", "/api/v1/robots", robotPayload)
    suite.Equal(201, resp.StatusCode)
    
    // Retrieve robot
    resp = makeRequest("GET", "/api/v1/robots/"+robotID, nil)
    suite.Equal(200, resp.StatusCode)
    
    // Verify in database
    var robot models.Robot
    suite.db.First(&robot, robotID)
    suite.Equal("HV1-00123", robot.SerialNumber)
}

func TestRobotIntegrationTestSuite(t *testing.T) {
    suite.Run(t, new(RobotIntegrationTestSuite))
}
```

---

### 3. End-to-End Tests

**Coverage Target**: Critical user journeys

**Scenario 1: Robot Registration → Telemetry → Dashboard Display**

```typescript
// e2e/tests/robot-lifecycle.spec.ts (using Playwright)
import { test, expect } from '@playwright/test';

test('robot lifecycle: register, send telemetry, appear in dashboard', async ({ page }) => {
  // 1. Login as admin
  await page.goto('http://localhost:3000/login');
  await page.fill('[name="username"]', 'admin');
  await page.fill('[name="password"]', 'adminpass');
  await page.click('button[type="submit"]');
  
  // 2. Register new robot
  await page.goto('http://localhost:3000/robots/new');
  await page.fill('[name="serial_number"]', 'HV1-TEST-001');
  await page.selectOption('[name="model"]', 'HumanoidV1');
  await page.click('button[type="submit"]');
  
  // Wait for success message
  await expect(page.locator('.success-toast')).toBeVisible();
  
  // 3. Start robot simulator (via API)
  const apiKey = await page.locator('.api-key').textContent();
  startSimulator('HV1-TEST-001', apiKey);
  
  // 4. Verify robot appears in dashboard
  await page.goto('http://localhost:3000/fleet');
  await expect(page.locator('text=HV1-TEST-001')).toBeVisible();
  await expect(page.locator('[data-robot="HV1-TEST-001"] .status-online')).toBeVisible();
  
  // 5. Verify telemetry updates
  await page.waitForTimeout(2000); // Wait for telemetry
  const battery = await page.locator('[data-robot="HV1-TEST-001"] .battery').textContent();
  expect(parseInt(battery)).toBeGreaterThan(0);
});
```

---

### 4. Load Testing

**Tool**: k6 (https://k6.io/)

**Test Scenario: 1000 Robots at 1Hz**

```javascript
// loadtests/telemetry-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 robots
    { duration: '5m', target: 1000 },  // Ramp up to 1000 robots
    { duration: '10m', target: 1000 }, // Stay at 1000 robots
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'],  // 95% of requests < 200ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  const robotId = `robot-${__VU}`;
  const apiKey = 'test-api-key';
  
  const telemetry = JSON.stringify({
    robot_id: robotId,
    timestamp: new Date().toISOString(),
    battery_level: Math.random() * 100,
    status: 'working',
    location: { x: Math.random() * 100, y: Math.random() * 100, z: 0 },
    metrics: {
      cpu_temp: 60 + Math.random() * 20,
      cpu_usage: Math.random() * 100,
      memory_usage: Math.random() * 100,
    },
  });
  
  const res = http.post('http://localhost:8080/api/v1/telemetry', telemetry, {
    headers: { 
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`,
    },
  });
  
  check(res, {
    'status is 201': (r) => r.status === 201,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });
  
  sleep(1); // 1Hz telemetry rate
}
```

**Run**:
```bash
k6 run loadtests/telemetry-load.js
```

---

### 5. Security Testing

**1. Dependency Scanning**

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk
        uses: snyk/actions/golang@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Run Trivy (container scan)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'humanoidops-api:latest'
          severity: 'CRITICAL,HIGH'
```

**2. SAST (Static Application Security Testing)**

```bash
# Using gosec for Go
gosec ./...

# Using bandit for Python
bandit -r agent/
```

**3. Penetration Testing Checklist**

- [ ] SQL injection attempts (parameterized queries)
- [ ] XSS payloads in API inputs
- [ ] CSRF token validation
- [ ] JWT token tampering
- [ ] Rate limiting bypass attempts
- [ ] Authentication brute force
- [ ] Authorization bypass (lateral movement)
- [ ] Sensitive data exposure in logs
- [ ] TLS configuration strength

---

## Deployment Architecture

### Development Environment

**Docker Compose Setup**:
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Database
  postgres:
    image: timescale/timescaledb:latest-pg14
    environment:
      POSTGRES_DB: humanoidops
      POSTGRES_USER: humanoidops
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U humanoidops"]
      interval: 10s
      timeout: 5s
      retries: 5

  # MQTT Broker
  emqx:
    image: emqx/emqx:5.3.0
    ports:
      - "1883:1883"      # MQTT
      - "8883:8883"      # MQTT/SSL
      - "18083:18083"    # Dashboard
    environment:
      EMQX_NAME: humanoidops
      EMQX_HOST: localhost
    volumes:
      - emqx_data:/opt/emqx/data
      - emqx_log:/opt/emqx/log

  # API Service
  api:
    build:
      context: ./backend
      dockerfile: docker/api.Dockerfile
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgresql://humanoidops:dev_password@postgres:5432/humanoidops
      MQTT_BROKER_URL: tcp://emqx:1883
      JWT_SECRET: dev-secret-change-in-production
      PORT: 8080
    depends_on:
      postgres:
        condition: service_healthy
      emqx:
        condition: service_started
    volumes:
      - ./backend:/app
    command: go run cmd/api/main.go

  # MQTT Handler
  mqtt-handler:
    build:
      context: ./backend
      dockerfile: docker/mqtt-handler.Dockerfile
    environment:
      DATABASE_URL: postgresql://humanoidops:dev_password@postgres:5432/humanoidops
      MQTT_BROKER_URL: tcp://emqx:1883
    depends_on:
      postgres:
        condition: service_healthy
      emqx:
        condition: service_started
    volumes:
      - ./backend:/app
    command: go run cmd/mqtt-handler/main.go

  # Dashboard
  dashboard:
    build:
      context: ./dashboard
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:8080/api/v1
      NEXT_PUBLIC_WS_URL: ws://localhost:8080/api/v1/ws
    volumes:
      - ./dashboard:/app
      - /app/node_modules
      - /app/.next
    command: npm run dev

  # Prometheus
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./docker/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  # Grafana
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_INSTALL_PLUGINS: grafana-clock-panel
    volumes:
      - grafana_data:/var/lib/grafana
      - ./docker/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./docker/grafana/datasources:/etc/grafana/provisioning/datasources

volumes:
  postgres_data:
  emqx_data:
  emqx_log:
  prometheus_data:
  grafana_data:
```

**Start Development Environment**:
```bash
docker-compose up -d
```

---

### Production Deployment

**Option 1: Docker Compose (Small Scale)**

```yaml
# docker-compose.production.yml
version: '3.8'

services:
  postgres:
    image: timescale/timescaledb:latest-pg14
    restart: always
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - /data/postgres:/var/lib/postgresql/data
    networks:
      - internal

  emqx:
    image: emqx/emqx:5.3.0
    restart: always
    ports:
      - "8883:8883"  # MQTT/TLS only (not 1883)
    volumes:
      - /data/emqx:/opt/emqx/data
      - /etc/ssl/certs:/opt/emqx/etc/certs:ro
    networks:
      - internal

  api:
    image: humanoidops/api:0.1.0
    restart: always
    environment:
      DATABASE_URL_FILE: /run/secrets/database_url
      JWT_SECRET_FILE: /run/secrets/jwt_secret
      MQTT_BROKER_URL: tcp://emqx:1883
    secrets:
      - database_url
      - jwt_secret
    networks:
      - internal
      - web
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1.0'
          memory: 1G

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
    networks:
      - web
    depends_on:
      - api
      - dashboard

secrets:
  db_password:
    external: true
  database_url:
    external: true
  jwt_secret:
    external: true

networks:
  internal:
  web:
```

---

**Option 2: Kubernetes (Future - v0.2+)**

```yaml
# k8s/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: humanoidops-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: humanoidops-api
  template:
    metadata:
      labels:
        app: humanoidops-api
    spec:
      containers:
      - name: api
        image: humanoidops/api:0.1.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /api/v1/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

---

## Implementation Phases

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Repository & Development Environment

**Goals**:
- Complete project structure
- Working local development environment
- CI/CD pipelines configured
- Documentation skeleton

**Deliverables**:
- [x] GitHub repository with folder structure
- [x] `docker-compose.yml` working
- [x] Database migrations framework
- [x] CI/CD workflows (GitHub Actions)
- [x] `README.md` with quick start
- [x] `Makefile` with common commands

**Tasks**: *(See detailed breakdown in PRD.md → Next Steps)*

---

#### Week 2: Robot Simulator

**Goals**:
- Functional robot simulator
- MQTT client library
- CLI tool for simulation
- Basic telemetry generation

**Deliverables**:
- [x] Python package `humanoidops-agent`
- [x] `RobotSimulator` class with realistic behavior
- [x] MQTT client wrapper
- [x] CLI to run N simulated robots
- [x] Unit tests for simulator
- [x] Example usage documentation

**Implementation Files**:
```
agent/
├── humanoidops_agent/
│   ├── simulator.py        ← Core simulator
│   ├── mqtt.py             ← MQTT transport
│   ├── cli.py              ← Command-line tool
│   └── models.py           ← Pydantic models
└── tests/
    └── test_simulator.py
```

---

#### Week 3: Telemetry Ingestion (Backend - Part 1)

**Goals**:
- Database schema implemented
- MQTT handler service receiving messages
- Telemetry written to database
- Basic API structure

**Deliverables**:
- [x] PostgreSQL + TimescaleDB running
- [x] Database migrations (initial schema)
- [x] MQTT handler service (Go)
- [x] Telemetry writing to `telemetry` table
- [x] Health check endpoint

**Implementation Files**:
```
backend/
├── migrations/
│   └── 001_initial_schema.up.sql
├── internal/
│   ├── database/database.go
│   ├── models/telemetry.go
│   └── mqtt/handler.go
└── cmd/
    └── mqtt-handler/main.go
```

**Testing**:
```bash
# Start services
docker-compose up -d

# Run simulator
cd agent && python -m humanoidops_agent.cli --num-robots 5

# Verify data in DB
psql -h localhost -U humanoidops -c "SELECT COUNT(*) FROM telemetry;"
```

---

#### Week 4: REST API (Backend - Part 2)

**Goals**:
- Complete REST API implementation
- Authentication (JWT)
- RBAC middleware
- OpenAPI documentation

**Deliverables**:
- [x] All robot management endpoints
- [x] Authentication endpoints (login, refresh)
- [x] Telemetry query endpoints
- [x] JWT authentication middleware
- [x] RBAC authorization middleware
- [x] OpenAPI spec generated
- [x] Unit tests for handlers

**Implementation Files**:
```
backend/
├── internal/
│   ├── handlers/
│   │   ├── auth.go
│   │   ├── robots.go
│   │   └── telemetry.go
│   ├── middleware/
│   │   ├── auth.go
│   │   └── cors.go
│   └── services/
│       ├── auth_service.go
│       └── robot_service.go
└── cmd/
    └── api/main.go
```

**API Testing**:
```bash
# Login
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'

# Register robot
curl -X POST http://localhost:8080/api/v1/robots \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{"serial_number":"HV1-00123","model":"HumanoidV1"}'
```

---

### Phase 2: Dashboard & Alerts (Weeks 5-8)

#### Week 5: Authentication & User Management

**Goals**:
- User model and database
- Login/logout functionality
- Protected routes
- User creation (admin-only)

**Deliverables**:
- [x] Users table with RBAC
- [x] User creation endpoint (admin)
- [x] Password hashing (bcrypt)
- [x] JWT token generation
- [x] Token refresh mechanism
- [x] Bootstrap script (create first admin)

**Implementation**:
```sql
-- Create first admin user
INSERT INTO users (username, email, password_hash, role)
VALUES ('admin', 'admin@example.com', '$2a$12$...', 'admin');
```

---

#### Week 6: Dashboard MVP

**Goals**:
- Next.js application structure
- Login page
- Fleet overview page
- Real-time updates (WebSocket)
- Basic styling (Tailwind + shadcn/ui)

**Deliverables**:
- [x] Next.js app with App Router
- [x] Login page with authentication
- [x] Protected routes
- [x] Fleet overview page (robot grid)
- [x] WebSocket connection for real-time updates
- [x] Robot status cards
- [x] Basic filters (status, model)
- [x] Responsive layout

**Implementation Files**:
```
dashboard/
├── app/
│   ├── (auth)/login/page.tsx
│   ├── (dashboard)/fleet/page.tsx
│   └── layout.tsx
├── components/
│   ├── RobotCard.tsx
│   └── StatusFilter.tsx
└── lib/
    ├── api.ts
    └── websocket.ts
```

---

#### Week 7: Alert Engine & Dashboard

**Goals**:
- Alert rule engine (Go backend)
- Alert triggers on conditions
- Alert dashboard page
- Alert acknowledgment

**Deliverables**:
- [x] Alert evaluation in MQTT handler
- [x] Alert rules (offline, battery, temp, etc.)
- [x] Alerts table populated
- [x] Alert API endpoints
- [x] Alerts page in dashboard
- [x] Acknowledge alert functionality
- [x] Real-time alert notifications (WebSocket)
- [x] Alert badge in navigation

**Implementation Files**:
```
backend/
├── internal/
│   └── alert/
│       ├── engine.go
│       └── rules.go

dashboard/
└── app/
    └── (dashboard)/
        └── alerts/
            └── page.tsx
```

---

#### Week 8: Audit Logs & Finalization

**Goals**:
- Audit logging complete
- Audit log viewer in dashboard
- Documentation complete
- Testing complete
- Production-ready

**Deliverables**:
- [x] Audit logs for all actions
- [x] Audit log API endpoint
- [x] Audit log viewer page (admin/operator)
- [x] Complete API documentation (OpenAPI)
- [x] User guide documentation
- [x] Deployment guide
- [x] All tests passing (>80% coverage)
- [x] Security scan clean
- [x] Performance testing complete
- [x] Beta testing with 2 external orgs

**Final Checklist**: *(See PRD.md → Acceptance Criteria)*

---

## Architecture Decision Records

### ADR-001: Use Go for Backend Services

**Status**: Accepted

**Context**: 
Need to choose backend language for API and MQTT handler services. Requirements: performance, concurrency, maintainability, ecosystem.

**Decision**: 
Use Go 1.21+ for all backend services.

**Consequences**:
- **Positive**: 
  - Excellent performance (compiled, efficient)
  - Built-in concurrency (goroutines, channels)
  - Strong standard library
  - Good MQTT and database libraries
  - Single binary deployment
- **Negative**: 
  - Smaller talent pool than Python/JS
  - Less flexible than dynamic languages
  - Verbosity (error handling)

**Alternatives Considered**:
- Python: Easier for robotics devs, but slower
- Node.js: Good for real-time, less type-safe
- Rust: Better performance, steeper learning curve

---

### ADR-002: Use TimescaleDB Extension for Time-Series Data

**Status**: Accepted

**Context**: 
Telemetry data is time-series (robot sends data every second). Need efficient storage and querying for potentially 1000 robots × 30 days = 2.6B records.

**Decision**: 
Use PostgreSQL with TimescaleDB extension.

**Consequences**:
- **Positive**: 
  - 10-100x faster queries on time ranges
  - Automatic partitioning (no manual effort)
  - Compression (90% storage reduction)
  - Automatic data retention
  - Familiar PostgreSQL interface
- **Negative**: 
  - Additional dependency (extension)
  - Slightly more complex setup
  - Less mature than pure PostgreSQL

**Alternatives Considered**:
- InfluxDB: Purpose-built time-series, but requires separate DB
- PostgreSQL only: Simple, but poor performance on large time-series
- Cassandra: Massive scale, but overkill and complex

---

### ADR-003: Modular Monolith for v0.1 (Not Microservices)

**Status**: Accepted

**Context**: 
Need to decide on service architecture. Could go full microservices (separate services for each domain) or modular monolith.

**Decision**: 
Use modular monolith approach for v0.1:
- Separate services (API, MQTT Handler) deployed together
- Shared database
- Clear module boundaries within services

**Consequences**:
- **Positive**: 
  - Simpler deployment (docker-compose)
  - Easier debugging (no distributed tracing needed)
  - Faster development (no inter-service communication)
  - Lower operational overhead
  - Easy to split into microservices later
- **Negative**: 
  - Harder to scale individual components
  - Shared database (coupling)
  - Team must maintain boundaries

**Alternatives Considered**:
- Full microservices: Better scalability, but premature for MVP
- True monolith: Simpler, but harder to split later

---

### ADR-004: Use MQTT as Primary Telemetry Transport

**Status**: Accepted

**Context**: 
Robots need to send telemetry to platform. Options: MQTT, HTTP, gRPC, WebSocket.

**Decision**: 
Use MQTT as primary transport, HTTPS as fallback.

**Consequences**:
- **Positive**: 
  - Designed for IoT (lightweight, efficient)
  - Publish/subscribe pattern
  - QoS guarantees
  - Persistent connections (lower latency)
  - Industry standard for robotics
- **Negative**: 
  - Requires MQTT broker (additional service)
  - More complex than simple HTTP
  - Stateful connections

**Alternatives Considered**:
- HTTP only: Simple, but requires new connection per message
- gRPC: Efficient, but less robotics ecosystem support
- WebSocket: Real-time, but more complex server-side

---

### ADR-005: Use JWT for Authentication (Not Sessions)

**Status**: Accepted

**Context**: 
Need authentication mechanism for API. Options: session cookies, JWT tokens, OAuth2.

**Decision**: 
Use JWT tokens (HS256) with 24h expiry and refresh tokens.

**Consequences**:
- **Positive**: 
  - Stateless (no server-side session storage)
  - Scales horizontally (any API server validates)
  - Works with mobile/API clients
  - Contains user info (no DB lookup per request)
- **Negative**: 
  - Cannot revoke token before expiry (need blacklist)
  - Larger than session ID
  - Secret management critical

**Alternatives Considered**:
- Session cookies: Stateful, requires Redis or DB
- OAuth2: Complex, overkill for MVP

---

### ADR-006: shadcn/ui for Component Library

**Status**: Accepted

**Context**: 
Need UI component library for dashboard. Options: Material-UI, Ant Design, shadcn/ui, build custom.

**Decision**: 
Use shadcn/ui (Radix UI primitives + Tailwind CSS).

**Consequences**:
- **Positive**: 
  - Copy-paste components (not npm dependency)
  - Full customization control
  - Lightweight (only includes what you use)
  - Accessible (Radix UI primitives)
  - Modern design
- **Negative**: 
  - Manual component management
  - Less out-of-the-box features than MUI
  - Smaller community

**Alternatives Considered**:
- Material-UI: Feature-rich, but heavier bundle
- Ant Design: Great for dashboards, but opinionated design
- Build custom: Full control, but time-consuming

---

### ADR-007: Role-Based Access Control (3 Roles)

**Status**: Accepted

**Context**: 
Need authorization model. Options: RBAC, ABAC (attribute-based), ACL (access control lists).

**Decision**: 
Implement RBAC with 3 fixed roles: admin, operator, viewer.

**Consequences**:
- **Positive**: 
  - Simple to understand and implement
  - Sufficient for v0.1 use cases
  - Easy to enforce in middleware
  - Standard pattern
- **Negative**: 
  - Not flexible (can't create custom roles)
  - Doesn't support fine-grained permissions
  - May need ABAC for enterprise (v1.0+)

**Alternatives Considered**:
- ABAC: More flexible, but complex to implement
- ACL: Fine-grained, but requires permission database

---

### ADR-008: Use Docker Compose (Not Kubernetes) for v0.1

**Status**: Accepted

**Context**: 
Need deployment strategy. Options: Docker Compose, Kubernetes, VMs, managed services.

**Decision**: 
Provide Docker Compose for v0.1 deployment, document Kubernetes option for future.

**Consequences**:
- **Positive**: 
  - Simple deployment (single command)
  - Works on any Docker host
  - Familiar to most developers
  - Low operational overhead
  - Easy local development
- **Negative**: 
  - Single-node deployment
  - No auto-scaling
  - Manual high availability
  - Not production-ready for large scale

**Future**: Provide Kubernetes manifests in v0.2 for production deployments.

**Alternatives Considered**:
- Kubernetes: Production-ready, but complex for MVP
- Managed services (AWS/GCP): Vendor lock-in
