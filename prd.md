# Product Requirements Document (PRD)
**HumanoidOps v0.1 - Humanoid Robots Operations Platform**

---

## Document Information

| Field | Value |
|-------|-------|
| **Product Name** | HumanoidOps |
| **Version** | 0.1 (MVP) |
| **Document Version** | 1.0 |
| **Author** | Federico Pacifici |
| **Status** | Review |
| **Created** | 2026-02-12 |
| **Last Updated** | 2026-02-12 |
| **Stakeholders** | Engineering, Product, Security |

---

## Table of Contents

1. [Executive Summary]
2. [Problem Statement]
3. [Goals & Success Metrics]
4. [User Stories & Personas]
5. [Functional Requirements]
6. [Non-Functional Requirements]
7. [User Interface & Experience]
8. [Acceptance Criteria]
9. [Out of Scope]
10. [Dependencies & Assumptions]
11. [Risks & Mitigations]
12. [Release Plan]
13. [Appendices]

---

## Executive Summary

### What We're Building

**HumanoidOps** is an open-source operations platform designed to manage, monitor, and secure fleets of humanoid robots at scale. It provides a vendor-neutral control layer that bridges the gap between robot manufacturers and operations teams.

### Why We're Building It

The humanoid robotics industry lacks a standardized, secure, open-source platform for fleet operations. Current solutions are either:
- **Vendor-locked**: Proprietary platforms tied to specific manufacturers
- **Incomplete**: IoT platforms not designed for humanoid robotics
- **Insecure**: Legacy systems without modern security architecture

### Who It's For

- **Robotics companies** deploying humanoid robot fleets
- **Operations teams** managing warehouse or facility robots
- **Developers** building robotics applications
- **Research institutions** studying robot fleet behavior

### Key Value Propositions

1. **Vendor-neutral**: Works with any humanoid robot manufacturer
2. **Security-first**: Built-in identity management, encryption, audit logging
3. **Open-source**: Transparent, extensible, community-driven
4. **Production-ready**: Designed for real-world deployment, not just demos
5. **Developer-friendly**: Clean APIs, SDKs, comprehensive documentation

### MVP Scope Summary

The v0.1 release focuses on **core fleet visibility and health monitoring**:

✅ **Included**:
- Robot identity and registration
- Real-time telemetry ingestion (MQTT/HTTPS)
- Fleet dashboard with status visualization
- Alerting engine (offline, battery, errors)
- Audit logging for compliance
- REST API & Python SDK

❌ **Not Included** (future versions):
- Tele-operation control
- AI training pipelines
- Multi-tenant SaaS features
- Mobile applications
- Video streaming

---

## Problem Statement

### Current State

Organizations deploying humanoid robot fleets face several critical challenges:

#### 1. Lack of Visibility
- **Problem**: No unified view of robot fleet health and status
- **Impact**: Delayed response to failures, inefficient resource allocation
- **Evidence**: Operations teams report spending 40% of time manually checking robot status

#### 2. Security Gaps
- **Problem**: Most robotics platforms lack enterprise-grade security
- **Impact**: Vulnerable to unauthorized access, data breaches, tampering
- **Evidence**: 60% of IoT/robotics deployments have critical security vulnerabilities (Gartner, 2023)

#### 3. Vendor Lock-in
- **Problem**: Proprietary platforms tie customers to single manufacturers
- **Impact**: Limited flexibility, high switching costs, reduced innovation
- **Evidence**: 75% of robotics customers cite vendor lock-in as a major concern

#### 4. Operational Inefficiency
- **Problem**: Manual monitoring, no proactive alerting, poor diagnostics
- **Impact**: High MTTR (Mean Time To Repair), increased downtime costs
- **Evidence**: Average downtime cost for industrial robots: $50K/hour

### Desired State

With HumanoidOps, organizations will have:

- ✅ **Unified dashboard** showing all robots regardless of manufacturer
- ✅ **Proactive alerts** detecting issues before they cause failures
- ✅ **Security baseline** with authentication, encryption, audit trails
- ✅ **Operational insights** from real-time and historical telemetry
- ✅ **Developer tools** (APIs, SDKs) for custom integrations

### Impact of Not Solving This

Without a solution:
- Robotics adoption will be slowed by operational complexity
- Security incidents will damage trust in autonomous systems
- Companies will waste resources building proprietary tools
- The industry will fragment into incompatible ecosystems

---

## Goals & Success Metrics

### Primary Goals

#### 1. Enable Fleet Visibility
**Goal**: Provide real-time visibility into robot fleet health and status

**Success Metrics**:
- Dashboard displays status of 100+ robots with <2s load time
- Real-time updates with <1s latency from robot to dashboard
- 99% telemetry delivery success rate

#### 2. Establish Security Foundation
**Goal**: Implement production-grade security from day one

**Success Metrics**:
- Zero critical security vulnerabilities at launch
- All communications encrypted (TLS 1.3, mTLS)
- Complete audit trail for all administrative actions
- RBAC implemented with 3+ roles

#### 3. Deliver Developer Experience
**Goal**: Make it easy for developers to integrate robots

**Success Metrics**:
- Developers can register and monitor first robot in <15 minutes
- Python SDK published with <5 dependencies
- API documentation scored 80+ on documentation quality metrics
- OpenAPI spec available for automated client generation

#### 4. Prove Scalability
**Goal**: Demonstrate platform can handle production workloads

**Success Metrics**:
- Support 1000+ concurrent robot connections
- Handle 100 telemetry messages/second aggregate throughput
- Database can store 30 days of telemetry for 1000 robots
- API p95 latency <200ms under normal load

### Secondary Goals

#### 5. Build Community Foundation
**Goal**: Establish open-source community and adoption

**Success Metrics** (3 months post-launch):
- 100+ GitHub stars
- 10+ external contributors
- 5+ production deployments outside core team
- Active discussions on GitHub Discussions/Discord

#### 6. Enable Future Features
**Goal**: Architecture supports planned v0.2+ features

**Success Metrics**:
- Service-oriented architecture allows adding new services
- Database schema supports future data types (video metadata, commands)
- API versioning strategy in place

### Key Performance Indicators (KPIs)

| KPI | Target (v0.1 Launch) | Measurement Method |
|-----|----------------------|-------------------|
| **System Uptime** | 99% | Prometheus uptime metrics |
| **API Latency (p95)** | <200ms | API response time tracking |
| **Dashboard Load Time** | <2s for 100 robots | Lighthouse performance score |
| **Telemetry Success Rate** | >99% | Messages received vs sent |
| **Alert Latency** | <5s from trigger | Alert timestamp analysis |
| **Documentation Coverage** | 100% of API endpoints | OpenAPI completeness |
| **Test Coverage** | >80% backend, >70% frontend | Code coverage tools |
| **Security Vulnerabilities** | 0 critical/high | Dependency scanning (Snyk) |

### Success Criteria

The v0.1 release is considered **successful** if:

1. ✅ All acceptance criteria met (see section 8)
2. ✅ Successfully demonstrated with 100+ simulated robots
3. ✅ At least 2 external organizations pilot the platform
4. ✅ Zero critical bugs in first 30 days post-launch
5. ✅ Positive feedback from 80%+ of pilot users

---

## User Stories & Personas

### Primary Personas

#### Persona 1: Fleet Operations Manager (Sarah)

**Background**:
- Age: 35-45
- Role: Operations Manager at warehouse robotics company
- Experience: 10+ years in operations, 2 years with robots
- Technical proficiency: Medium (can use dashboards, not a developer)

**Goals**:
- Monitor robot fleet health in real-time
- Quickly identify and respond to issues
- Minimize robot downtime
- Generate reports for management

**Pain Points**:
- Currently uses 3 different tools to monitor fleet
- No proactive alerting (discovers problems reactively)
- Can't easily see historical trends

**User Stories**:

```
US-001: As a fleet operations manager, I want to see all my robots' 
        status at a glance, so that I can quickly identify problems.
        
Priority: P0 (Must Have)
Acceptance: Dashboard shows all robots with color-coded status indicators

---

US-002: As a fleet operations manager, I want to receive alerts when 
        robots go offline, so that I can respond immediately.
        
Priority: P0 (Must Have)
Acceptance: Alert appears in dashboard <5s after robot offline >30s

---

US-003: As a fleet operations manager, I want to view historical 
        battery levels, so that I can identify robots needing maintenance.
        
Priority: P1 (Should Have)
Acceptance: Charts show battery trends for last 24 hours

---

US-004: As a fleet operations manager, I want to filter robots by 
        status or location, so that I can focus on specific subsets.
        
Priority: P1 (Should Have)
Acceptance: Dashboard has working filters for status, model, location

---

US-005: As a fleet operations manager, I want to acknowledge alerts, 
        so that I can track which issues I've addressed.
        
Priority: P1 (Should Have)
Acceptance: Alert can be marked acknowledged with timestamp and user
```

---

#### Persona 2: System Administrator (Marcus)

**Background**:
- Age: 28-40
- Role: DevOps/Systems Administrator
- Experience: 5+ years managing cloud infrastructure
- Technical proficiency: High (comfortable with APIs, CLI, Docker)

**Goals**:
- Securely register new robots
- Maintain system security and compliance
- Monitor system health (not just robots)
- Automate routine tasks

**Pain Points**:
- Manual robot registration is time-consuming
- No visibility into who did what (audit trail gaps)
- Difficult to grant appropriate permissions to team members

**User Stories**:

```
US-006: As a system administrator, I want to register robots via API, 
        so that I can automate onboarding of new robots.
        
Priority: P0 (Must Have)
Acceptance: POST /api/v1/robots endpoint works, returns robot credentials

---

US-007: As a system administrator, I want audit logs of all actions, 
        so that I can maintain compliance and troubleshoot issues.
        
Priority: P0 (Must Have)
Acceptance: All API calls logged with user, timestamp, IP, result

---

US-008: As a system administrator, I want role-based access control, 
        so that I can grant appropriate permissions to team members.
        
Priority: P0 (Must Have)
Acceptance: 3 roles (admin, operator, viewer) with different permissions

---

US-009: As a system administrator, I want to deploy with Docker Compose, 
        so that I can quickly set up dev/staging environments.
        
Priority: P0 (Must Have)
Acceptance: `docker-compose up` starts all services successfully

---

US-010: As a system administrator, I want Prometheus metrics, 
        so that I can monitor system health and set up alerting.
        
Priority: P1 (Should Have)
Acceptance: /api/v1/metrics endpoint exposes Prometheus-compatible metrics
```

---

#### Persona 3: Robotics Developer (Priya)

**Background**:
- Age: 25-35
- Role: Software Engineer at robotics startup
- Experience: 3+ years in robotics/embedded systems
- Technical proficiency: High (Python, C++, comfortable with SDKs)

**Goals**:
- Integrate robots with monitoring platform quickly
- Build custom tools on top of the platform
- Test without physical robots (using simulators)
- Understand API capabilities

**Pain Points**:
- Documentation is often incomplete or outdated
- SDKs have too many dependencies or are poorly designed
- No easy way to test integration without hardware

**User Stories**:

```
US-011: As a robotics developer, I want a Python SDK, 
        so that I can integrate robots without dealing with raw MQTT/HTTP.
        
Priority: P0 (Must Have)
Acceptance: pip install humanoidops-agent works, basic example runs

---

US-012: As a robotics developer, I want comprehensive API docs, 
        so that I can build custom integrations.
        
Priority: P0 (Must Have)
Acceptance: OpenAPI spec available, all endpoints documented with examples

---

US-013: As a robotics developer, I want a robot simulator, 
        so that I can test my integration without physical hardware.
        
Priority: P0 (Must Have)
Acceptance: CLI tool can simulate 10+ robots sending realistic telemetry

---

US-014: As a robotics developer, I want code examples, 
        so that I can get started quickly.
        
Priority: P1 (Should Have)
Acceptance: GitHub repo has examples/ folder with 3+ working examples

---

US-015: As a robotics developer, I want simple authentication, 
        so that I can connect robots securely without complexity.
        
Priority: P0 (Must Have)
Acceptance: Robot can authenticate with API key or certificate
```

---

### Secondary Personas

#### Persona 4: Security Engineer (James)

**Concerns**:
- Threat modeling and vulnerability assessment
- Compliance with security standards
- Secure communication channels
- Audit trail completeness

**Key Requirements**:
- All communications encrypted (TLS 1.3 minimum)
- mTLS for robot-to-server authentication
- Comprehensive audit logging
- No critical vulnerabilities in dependencies

---

#### Persona 5: C-Level Executive (Lisa - CTO)

**Concerns**:
- Vendor lock-in avoidance
- Total cost of ownership
- Scalability for future growth
- Competitive differentiation

**Key Requirements**:
- Open-source with permissive license
- Cloud-agnostic deployment
- Clear roadmap for future capabilities
- Active community and ecosystem

---

## Functional Requirements

### FR-1: Robot Identity & Registration

#### FR-1.1: Robot Registration

**Description**: Administrators can register new robots in the system, creating a unique identity and metadata record.

**Requirements**:
1. System MUST generate unique robot ID (UUID v4 format)
2. System MUST require the following fields during registration:
   - Serial number (unique, alphanumeric, max 100 chars)
   - Model name (string, max 100 chars)
3. System SHOULD accept optional fields:
   - Manufacturer name
   - Firmware version
   - Location name
   - Location coordinates (latitude/longitude)
4. System MUST validate serial number uniqueness before registration
5. System MUST record who registered the robot (user ID) and when
6. System MUST generate authentication credentials for the robot (API key or certificate)
7. Registration MUST require admin-level authorization

**API Endpoint**:
```http
POST /api/v1/robots
Authorization: Bearer {jwt_token}
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
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "serial_number": "HV1-00123",
  "model": "HumanoidV1",
  "registered_at": "2024-01-15T10:30:00Z",
  "registered_by": "admin-user-id",
  "api_key": "secret_key_abc123...",
  "status": "active"
}
```

#### FR-1.2: Robot Metadata Management

**Description**: Administrators can view and update robot metadata.

**Requirements**:
1. System MUST provide endpoints to:
   - List all robots (with pagination)
   - Get individual robot details
   - Update robot metadata (except ID and serial number)
   - Deactivate/reactivate robots
2. List endpoint MUST support filtering by:
   - Status (active, inactive, maintenance)
   - Model
   - Location name
3. List endpoint MUST support pagination (default 20 items/page, max 100)
4. System MUST NOT allow deletion of robots with historical telemetry
5. Deactivation MUST be soft delete (status change, not record deletion)

**API Endpoints**:
```http
GET /api/v1/robots?status=active&model=HumanoidV1&page=1&limit=20
GET /api/v1/robots/{robot_id}
PATCH /api/v1/robots/{robot_id}
DELETE /api/v1/robots/{robot_id}  # Soft delete (deactivate)
```

#### FR-1.3: Robot Status Tracking

**Description**: System automatically tracks robot online/offline status.

**Requirements**:
1. System MUST update `last_seen` timestamp when telemetry received
2. System MUST consider robot **offline** if no telemetry received for >30 seconds
3. System MUST provide real-time status in dashboard
4. Status values MUST be one of: `online`, `offline`, `error`, `maintenance`, `charging`

---

### FR-2: Telemetry Ingestion

#### FR-2.1: MQTT Telemetry

**Description**: Robots send telemetry data via MQTT protocol.

**Requirements**:
1. System MUST accept telemetry via MQTT on topic: `robots/{robot_id}/telemetry`
2. System MUST support MQTT protocol version 5.0 (backward compatible with 3.1.1)
3. System MUST use QoS 1 (at least once delivery)
4. System MUST validate robot authentication before accepting messages
5. System MUST support TLS encryption for MQTT connections
6. System SHOULD support mTLS (mutual TLS) for robot authentication
7. System MUST handle at least 100 messages/second aggregate throughput
8. System MUST timestamp each message server-side (in addition to robot timestamp)

**MQTT Configuration**:
```yaml
broker: emqx
port: 1883 (non-TLS) / 8883 (TLS)
protocol: MQTT v5.0
qos: 1
topic_pattern: robots/{robot_id}/telemetry
authentication: username/password or client certificates
```

**Message Schema** (see FR-2.3 for details)

#### FR-2.2: HTTPS Telemetry Fallback

**Description**: Robots can send telemetry via HTTPS POST as fallback when MQTT unavailable.

**Requirements**:
1. System MUST accept telemetry via `POST /api/v1/telemetry`
2. System MUST validate JWT or API key authentication
3. System MUST accept same JSON schema as MQTT
4. System MUST respond with 201 Created on success, 400 on validation error
5. System MUST rate limit to 10 requests/second per robot

**API Endpoint**:
```http
POST /api/v1/telemetry
Authorization: Bearer {robot_api_key}
Content-Type: application/json

{
  "robot_id": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-15T10:35:45Z",
  "battery_level": 85.5,
  "status": "working",
  "location": {"x": 12.5, "y": 8.3, "z": 0.0},
  "metrics": {
    "cpu_temp": 65.2,
    "cpu_usage": 45.8,
    "memory_usage": 60.3,
    "joint_health": [98, 97, 99, 100, 95, 98, 97, 99, 100, 96, 98, 97],
    "error_codes": [],
    "network_latency": 45
  }
}
```

#### FR-2.3: Telemetry Data Schema

**Description**: Standardized telemetry format for all robots.

**Required Fields**:
```json
{
  "robot_id": "uuid (string, required)",
  "timestamp": "ISO 8601 datetime (string, required)",
  "battery_level": "float 0-100 (required)",
  "status": "enum: idle|working|error|offline|charging (required)"
}
```

**Optional Fields**:
```json
{
  "location": {
    "x": "float (meters)",
    "y": "float (meters)",
    "z": "float (meters)"
  },
  "metrics": {
    "cpu_temp": "float (celsius)",
    "cpu_usage": "float 0-100 (percent)",
    "memory_usage": "float 0-100 (percent)",
    "joint_health": "array of floats 0-100 (percent per joint)",
    "error_codes": "array of strings",
    "network_latency": "integer (milliseconds)",
    "custom": "object (extensible for manufacturer-specific data)"
  }
}
```

**Validation Rules**:
1. `robot_id` MUST be valid UUID and exist in robots table
2. `timestamp` MUST be within 5 minutes of server time (clock skew tolerance)
3. `battery_level` MUST be 0-100
4. `status` MUST be one of allowed enum values
5. If `metrics.joint_health` provided, MUST match number of joints for robot model
6. Total message size MUST NOT exceed 10KB

#### FR-2.4: Telemetry Storage

**Description**: Telemetry data is stored in TimescaleDB for time-series analysis.

**Requirements**:
1. System MUST store all telemetry in `telemetry` hypertable
2. System MUST index on `(robot_id, timestamp DESC)` for fast queries
3. System MUST retain telemetry data for **30 days**
4. System MUST automatically compress data older than 7 days
5. System MUST handle large JSON in `metrics` column (JSONB type)
6. System MUST validate robot exists before inserting telemetry

**Database Schema**:
```sql
CREATE TABLE telemetry (
    robot_id UUID NOT NULL REFERENCES robots(id) ON DELETE CASCADE,
    timestamp TIMESTAMPTZ NOT NULL,
    battery_level DOUBLE PRECISION,
    status VARCHAR(50),
    location JSONB,
    metrics JSONB,
    server_timestamp TIMESTAMPTZ DEFAULT NOW()
);

SELECT create_hypertable('telemetry', 'timestamp');
CREATE INDEX idx_telemetry_robot_time ON telemetry(robot_id, timestamp DESC);
```

---

### FR-3: Fleet Dashboard

#### FR-3.1: Fleet Overview Page

**Description**: Main dashboard showing all robots with key metrics.

**Requirements**:
1. Dashboard MUST display robot cards in grid layout
2. Each card MUST show:
   - Robot ID (first 8 chars) and serial number
   - Status indicator (color-coded: 🟢 online, 🔴 offline, 🟡 warning)
   - Battery level (percentage + visual bar)
   - Last seen timestamp (relative time, e.g., "2m ago")
   - Location name (if available)
3. Dashboard MUST update in real-time (<1s latency from telemetry to display)
4. Dashboard MUST support filtering by:
   - Status (online, offline, error, all)
   - Model
   - Location
5. Dashboard MUST support search by robot ID or serial number
6. Dashboard MUST display fleet summary:
   - Total robot count
   - Robots online/offline/error (count and percentage)
   - Average battery level
   - Active alerts count
7. Dashboard MUST load in <2 seconds for 100 robots

**UI Layout**:
```
┌─────────────────────────────────────────────────────────┐
│  HumanoidOps                           [User] [Logout]  │
├─────────────────────────────────────────────────────────┤
│  Fleet Overview     Alerts     Audit Logs     Settings  │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────┐   │
│  │  Fleet Summary                                   │   │
│  │  • Total: 125 robots                            │   │
│  │  • Online: 118 (94%) | Offline: 5 | Error: 2   │   │
│  │  • Avg Battery: 78%  | Active Alerts: 7        │   │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
│  Filter: [All ▼] [Model ▼] [Location ▼]  [Search box]  │
│                                                          │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                  │
│  │ 🟢   │ │ 🟢   │ │ 🔴   │ │ 🟡   │  ...             │
│  │ ID   │ │ ID   │ │ ID   │ │ ID   │                  │
│  │ Batt │ │ Batt │ │ Batt │ │ Batt │                  │
│  │ 2m   │ │ 10s  │ │ 45s  │ │ 1m   │                  │
│  └──────┘ └──────┘ └──────┘ └──────┘                  │
└─────────────────────────────────────────────────────────┘
```

#### FR-3.2: Robot Detail Page

**Description**: Detailed view of individual robot with historical data.

**Requirements**:
1. Page MUST display complete robot metadata
2. Page MUST show telemetry charts:
   - Battery level over time (line chart, last 24h)
   - CPU temperature over time (line chart, last 24h)
   - Status timeline (swimlane chart showing status changes)
3. Page MUST show current metrics table:
   - All values from latest telemetry
   - Timestamp of last update
4. Page MUST show recent alerts (last 10 for this robot)
5. Page MUST allow time range selection for charts (1h, 6h, 24h, 7d)
6. Charts MUST support zoom and pan
7. Page MUST provide "Refresh Now" button for manual refresh
8. Page MUST auto-refresh every 5 seconds

**Data Fetching**:
```http
GET /api/v1/robots/{robot_id}
GET /api/v1/robots/{robot_id}/telemetry?start={iso_time}&end={iso_time}
GET /api/v1/alerts?robot_id={robot_id}&limit=10
```

#### FR-3.3: Real-time Updates

**Description**: Dashboard updates automatically when new data arrives.

**Requirements**:
1. System MUST provide WebSocket endpoint for real-time updates
2. WebSocket MUST broadcast when:
   - New telemetry received (for dashboard refresh)
   - Robot status changes (online ↔ offline)
   - New alert triggered
3. Updates MUST be sent within 1 second of event occurrence
4. WebSocket MUST require authentication (JWT in connection params)
5. WebSocket MUST gracefully handle reconnection if connection lost
6. Client MUST batch updates if receiving >10 messages/second

**WebSocket Protocol**:
```javascript
// Connect
ws = new WebSocket('ws://localhost:8080/api/v1/ws?token={jwt}')

// Message types
{
  "type": "telemetry_update",
  "robot_id": "uuid",
  "data": { ... telemetry object ... }
}

{
  "type": "status_change",
  "robot_id": "uuid",
  "old_status": "online",
  "new_status": "offline"
}

{
  "type": "alert_triggered",
  "alert": { ... alert object ... }
}
```

---

### FR-4: Alert Engine

#### FR-4.1: Alert Rules

**Description**: System automatically triggers alerts based on configurable rules.

**Required Alert Types** (v0.1):

1. **Robot Offline Alert**
   - **Trigger**: No telemetry received for >30 seconds
   - **Severity**: Critical
   - **Message**: "Robot {serial} has been offline for {duration}"
   - **Auto-clear**: When robot comes back online

2. **Low Battery Alert**
   - **Trigger**: Battery level <20% (warning) or <10% (critical)
   - **Severity**: Warning (20%), Critical (10%)
   - **Message**: "Battery level critically low ({battery}%)"
   - **Auto-clear**: When battery >25%

3. **Error Status Alert**
   - **Trigger**: Robot reports status='error'
   - **Severity**: Critical
   - **Message**: "Robot reporting error status. Error codes: {codes}"
   - **Auto-clear**: When robot returns to idle/working status

4. **High Temperature Alert**
   - **Trigger**: CPU temperature >80°C (warning) or >85°C (critical)
   - **Severity**: Warning (80°C), Critical (85°C)
   - **Message**: "CPU temperature above threshold ({temp}°C)"
   - **Auto-clear**: When temperature drops below 75°C

5. **High Network Latency Alert**
   - **Trigger**: Network latency >1000ms consistently (3 consecutive readings)
   - **Severity**: Warning
   - **Message**: "High network latency detected ({latency}ms)"
   - **Auto-clear**: When latency <500ms

**Requirements**:
1. System MUST evaluate all alert rules on every telemetry message received
2. System MUST NOT create duplicate alerts (check if active alert exists)
3. System MUST record who acknowledged alert and when
4. System MUST support alert threshold customization per robot model (future: v0.2)

#### FR-4.2: Alert Display

**Description**: Alerts are displayed in dashboard with clear visibility.

**Requirements**:
1. Dashboard MUST show alert count badge in navigation
2. Dashboard MUST highlight robots with active alerts (red/yellow border)
3. Alerts page MUST list all alerts with:
   - Severity indicator (icon + color)
   - Robot ID and serial number
   - Alert type and message
   - Triggered timestamp (relative time)
   - Acknowledged status
4. Alerts MUST be sortable by severity, time, robot
5. Alerts MUST be filterable by:
   - Acknowledged status (all, active, acknowledged)
   - Severity (all, critical, warning, info)
   - Robot
   - Time range
6. System MUST show notification toast when new critical alert arrives

#### FR-4.3: Alert Acknowledgment

**Description**: Users can acknowledge alerts to indicate they're being addressed.

**Requirements**:
1. User MUST be able to acknowledge individual alerts
2. Acknowledgment MUST record:
   - User who acknowledged
   - Timestamp of acknowledgment
   - Optional notes (future: v0.2)
3. Acknowledged alerts MUST remain visible but visually distinguished
4. System MUST support bulk acknowledgment (select multiple alerts)
5. Only users with `operator` or `admin` role can acknowledge alerts

**API Endpoint**:
```http
POST /api/v1/alerts/{alert_id}/acknowledge
Authorization: Bearer {jwt_token}

Response 200 OK:
{
  "id": "alert-uuid",
  "acknowledged": true,
  "acknowledged_by": "user-uuid",
  "acknowledged_at": "2024-01-15T10:45:00Z"
}
```

---

### FR-5: Audit Logging

#### FR-5.1: Audit Trail

**Description**: System logs all significant actions for security and compliance.

**Events to Log**:
1. **Authentication events**:
   - User login (success/failure)
   - User logout
   - Token refresh
   - Failed authentication attempts (with IP)

2. **Robot management**:
   - Robot registration
   - Robot metadata updates
   - Robot deactivation
   - Robot credential regeneration

3. **Alert management**:
   - Alert acknowledgment

4. **User management** (admin only):
   - User creation
   - User role changes
   - User deactivation

5. **Configuration changes** (future):
   - Alert rule modifications
   - System settings updates

**Requirements**:
1. Audit logs MUST be immutable (insert-only, no updates or deletes)
2. Audit logs MUST include:
   - Timestamp (UTC, microsecond precision)
   - User ID (null if system-generated)
   - Action name (e.g., 'robot.create', 'alert.acknowledge')
   - Resource type and ID
   - Result (success/failure)
   - IP address
   - User agent
   - Additional details (JSON)
3. Audit logs MUST be retained for **90 days** minimum
4. Audit logs MUST be indexed for fast querying
5. System MUST provide API to query audit logs with filtering

**Database Schema**:
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    result VARCHAR(20) NOT NULL,  -- 'success' or 'failure'
    ip_address INET,
    user_agent TEXT,
    details JSONB
);

CREATE INDEX idx_audit_timestamp ON audit_logs(timestamp DESC);
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
```

#### FR-5.2: Audit Log Viewer

**Description**: Dashboard page to search and view audit logs.

**Requirements**:
1. Page MUST be accessible only to `admin` and `operator` roles
2. Page MUST display audit log entries in table format
3. Table MUST show: timestamp, user, action, resource, result, IP
4. Page MUST support filtering by:
   - Time range (last hour, day, week, custom)
   - User
   - Action type
   - Result (success/failure)
   - Resource type
5. Page MUST support search by resource ID
6. Page MUST support pagination (50 entries per page)
7. Page MUST allow export to CSV (admin only)
8. Details column MUST be expandable (show full JSON)

**API Endpoint**:
```http
GET /api/v1/audit-logs?start={iso_time}&end={iso_time}&user_id={uuid}&action={string}&result={success|failure}&page=1&limit=50
```

---

### FR-6: API & SDK

#### FR-6.1: REST API

**Description**: Comprehensive REST API for all platform operations.

**Requirements**:
1. API MUST follow RESTful principles
2. API MUST use JSON for request/response bodies
3. API MUST include OpenAPI 3.0 specification
4. API MUST version endpoints (e.g., `/api/v1/...`)
5. API MUST return consistent error responses:
   ```json
   {
     "error": "Error message in plain English",
     "code": "ERROR_CODE",
     "details": { ... additional context ... }
   }
   ```
6. API MUST support pagination with `page` and `limit` query params
7. API MUST include `X-Request-ID` header for tracing
8. API MUST rate limit to 1000 requests/minute per user
9. API MUST return `429 Too Many Requests` when rate limit exceeded

**Complete Endpoint List**:

```
Authentication:
POST   /api/v1/auth/login             Login with username/password
POST   /api/v1/auth/refresh           Refresh JWT token
POST   /api/v1/auth/logout            Logout (invalidate token)

Robot Management:
POST   /api/v1/robots                 Register new robot (admin)
GET    /api/v1/robots                 List all robots (paginated)
GET    /api/v1/robots/{id}            Get robot details
PATCH  /api/v1/robots/{id}            Update robot metadata (admin)
DELETE /api/v1/robots/{id}            Deactivate robot (admin)

Telemetry:
POST   /api/v1/telemetry              Submit telemetry (HTTPS fallback)
GET    /api/v1/robots/{id}/telemetry  Get robot telemetry history

Alerts:
GET    /api/v1/alerts                 List all alerts (filterable)
GET    /api/v1/alerts/{id}            Get alert details
POST   /api/v1/alerts/{id}/acknowledge  Acknowledge alert

Audit Logs:
GET    /api/v1/audit-logs             Query audit logs (admin/operator)

System:
GET    /api/v1/health                 Health check (no auth)
GET    /api/v1/metrics                Prometheus metrics (no auth)
GET    /api/v1/openapi                OpenAPI spec (no auth)

WebSocket:
WS     /api/v1/ws                     Real-time updates (authenticated)
```

#### FR-6.2: Python SDK

**Description**: Official Python SDK for robot-side integration.

**Requirements**:
1. SDK MUST be published to PyPI as `humanoidops-agent`
2. SDK MUST support Python 3.10+
3. SDK MUST have <5 core dependencies
4. SDK MUST provide simple client API:
   ```python
   from humanoidops_agent import HumanoidOpsClient
   
   client = HumanoidOpsClient(
       robot_id="uuid",
       api_key="secret",
       mqtt_broker="mqtt://broker.example.com"
   )
   
   # Send telemetry (auto-retries on failure)
   client.send_telemetry(
       battery_level=85.5,
       status="working",
       location={"x": 12.5, "y": 8.3, "z": 0.0},
       metrics={...}
   )
   ```
5. SDK MUST handle connection failures gracefully:
   - Automatic reconnection with exponential backoff
   - Local telemetry queue if offline (max 1000 messages)
   - Flush queue when connection restored
6. SDK MUST support both MQTT and HTTPS transports
7. SDK MUST include type hints for all public methods
8. SDK MUST have >90% test coverage

**Package Structure**:
```
humanoidops_agent/
├── __init__.py
├── client.py         # Main HumanoidOpsClient
├── mqtt.py           # MQTT transport
├── http.py           # HTTPS transport
├── queue.py          # Offline message queue
├── simulator.py      # Robot simulator (for testing)
└── exceptions.py     # Custom exceptions
```

#### FR-6.3: API Documentation

**Description**: Complete, accurate, up-to-date API documentation.

**Requirements**:
1. OpenAPI spec MUST be auto-generated from code (if possible)
2. OpenAPI spec MUST include:
   - All endpoints with descriptions
   - Request/response schemas
   - Authentication requirements
   - Example requests and responses
   - Error codes and meanings
3. Documentation MUST be hosted at `/api/v1/openapi.yaml`
4. Interactive API explorer (Swagger UI) MUST be hosted at `/api/docs`
5. Documentation MUST include "Getting Started" guide
6. Documentation MUST include code examples in Python, Go, JavaScript
7. Documentation MUST be versioned alongside API

---

### FR-7: Authentication & Authorization

#### FR-7.1: User Authentication

**Description**: Secure user login with JWT tokens.

**Requirements**:
1. System MUST support username/password authentication
2. Passwords MUST be hashed with bcrypt (cost factor 12)
3. System MUST generate JWT tokens on successful login
4. JWT tokens MUST include claims: `user_id`, `username`, `role`, `exp`
5. JWT tokens MUST expire after 24 hours
6. System MUST support token refresh (extend expiry without re-login)
7. System MUST log all login attempts (success and failure)
8. System MUST implement rate limiting on login endpoint (5 attempts/minute)
9. System MUST lock account after 10 failed login attempts in 1 hour

**Login Flow**:
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "securepassword123"
}

Response 200 OK:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "refresh_xyz...",
  "expires_at": "2024-01-16T10:30:00Z",
  "user": {
    "id": "user-uuid",
    "username": "admin",
    "role": "admin"
  }
}
```

#### FR-7.2: Role-Based Access Control (RBAC)

**Description**: Three-tier permission system.

**Roles**:

| Role | Permissions |
|------|------------|
| **Admin** | • All permissions<br>• Register robots<br>• Manage users<br>• View audit logs<br>• Configure system |
| **Operator** | • View dashboard<br>• View all robots<br>• Acknowledge alerts<br>• View audit logs<br>• View telemetry |
| **Viewer** | • View dashboard<br>• View robots (read-only)<br>• View alerts (read-only)<br>• View telemetry (read-only) |

**Requirements**:
1. Every user MUST have exactly one role
2. API endpoints MUST enforce role-based permissions
3. Unauthorized access attempts MUST return `403 Forbidden`
4. System MUST create default admin user on first startup
5. Dashboard MUST hide UI elements based on user role
6. System MUST audit all authorization failures

#### FR-7.3: Robot Authentication

**Description**: Robots authenticate to send telemetry.

**Requirements**:
1. Each robot MUST have unique API key (minimum 32 chars, crypto-random)
2. Robot MUST include API key in MQTT username OR HTTP Authorization header
3. System MUST validate API key before accepting telemetry
4. Invalid API key MUST result in connection rejection
5. System MAY support client certificates (mTLS) as alternative
6. API keys MUST be rotatable without re-registering robot

---

## Non-Functional Requirements

### NFR-1: Performance

#### NFR-1.1: Response Time

| Metric | Target | Measurement |
|--------|--------|-------------|
| **API Response (p50)** | <100ms | 50th percentile of all API calls |
| **API Response (p95)** | <200ms | 95th percentile of all API calls |
| **API Response (p99)** | <500ms | 99th percentile of all API calls |
| **Dashboard Load (initial)** | <2s | Time to interactive for 100 robots |
| **Dashboard Load (subsequent)** | <500ms | Navigation between pages |
| **WebSocket Latency** | <100ms | Event to dashboard update |
| **Telemetry Processing** | <100ms | MQTT receive to DB insert |

**Testing**:
- Load testing with k6 or Locust
- Monitor with Prometheus + Grafana
- Lighthouse for frontend performance

#### NFR-1.2: Throughput

| Metric | Target | Notes |
|--------|--------|-------|
| **Telemetry Ingestion** | 100 msg/s | Aggregate across all robots |
| **API Requests** | 1000 req/min | Per server instance |
| **Concurrent Users** | 50+ | Without performance degradation |
| **Concurrent Robots** | 1000+ | Sustained connections |
| **Database Writes** | 500 writes/s | Inserts + updates |
| **Database Reads** | 2000 reads/s | Queries for dashboard |

#### NFR-1.3: Scalability

**Requirements**:
1. System MUST support horizontal scaling of API servers
2. API servers MUST be stateless (session in JWT, not server)
3. Database MUST handle 1000 robots × 1 msg/sec × 30 days = 2.6B rows
4. TimescaleDB compression MUST keep DB size manageable
5. System MUST support future Redis cache layer (not in v0.1)

**Capacity Planning**:
```
1000 robots × 1 telemetry/sec × 30 days
= 1000 × 86400 sec/day × 30 days
= 2,592,000,000 telemetry records

Average telemetry size: ~1KB
Storage requirement: ~2.6TB (uncompressed)
With TimescaleDB compression (7 days): ~500GB
```

---

### NFR-2: Security

#### NFR-2.1: Data Encryption

**Requirements**:
1. All communications MUST use TLS 1.3 (minimum TLS 1.2)
2. MQTT MUST support TLS on port 8883
3. HTTPS MUST enforce TLS (no HTTP allowed in production)
4. Certificates MUST be from trusted CA (Let's Encrypt for production)
5. System MAY support mTLS for robot authentication
6. Database connections MUST use TLS
7. Passwords MUST be hashed with bcrypt (cost 12, salted)
8. API keys MUST be stored hashed (SHA-256 minimum)

#### NFR-2.2: Authentication & Authorization

**Requirements**:
1. All API endpoints MUST require authentication (except /health, /metrics, /openapi)
2. JWT tokens MUST be signed with HS256 or RS256
3. JWT secret MUST be >32 characters from cryptographically secure random source
4. System MUST implement RBAC (see FR-7.2)
5. Failed authentication MUST NOT leak information about valid usernames
6. System MUST implement rate limiting on authentication endpoints

#### NFR-2.3: Security Best Practices

**Requirements**:
1. No secrets in code (use environment variables)
2. No secrets in logs (sanitize before logging)
3. SQL injection prevention (parameterized queries only)
4. XSS prevention (React auto-escaping, CSP headers)
5. CSRF prevention (SameSite cookies, CSRF tokens if using cookies)
6. CORS properly configured (whitelist origins, not *)
7. Security headers (HSTS, X-Frame-Options, X-Content-Type-Options)
8. Input validation on all API endpoints
9. Output encoding for all user-generated content
10. Regular dependency updates (automated with Dependabot)

#### NFR-2.4: Vulnerability Management

**Requirements**:
1. Zero critical or high vulnerabilities at launch
2. Dependency scanning with Snyk or similar (weekly)
3. SAST (Static Application Security Testing) in CI/CD
4. Security disclosure policy (SECURITY.md)
5. CVE monitoring for all dependencies
6. Security patches applied within 7 days of disclosure

#### NFR-2.5: Audit & Compliance

**Requirements**:
1. Complete audit trail (see FR-5)
2. Audit logs retained 90 days minimum
3. Audit logs tamper-evident (immutable, checksummed)
4. Support for export to SIEM systems (future)
5. Compliance documentation for SOC2/ISO27001 (future)

---

### NFR-3: Reliability

#### NFR-3.1: Availability

| Metric | Target |
|--------|--------|
| **Uptime SLO** | 99% (7.2 hours downtime/month) |
| **Planned Maintenance** | <2 hours/month, off-peak |
| **Recovery Time Objective (RTO)** | <15 minutes |
| **Recovery Point Objective (RPO)** | <1 hour |

**Requirements**:
1. System MUST have health check endpoints (`/api/v1/health`)
2. Health checks MUST verify:
   - API server responding
   - Database connection alive
   - MQTT broker reachable
3. System MUST gracefully degrade if MQTT unavailable (HTTPS fallback)
4. Database MUST support automated backups (daily minimum)
5. System MUST support blue-green deployments for zero-downtime updates

#### NFR-3.2: Error Handling

**Requirements**:
1. All errors MUST be logged with context (request ID, user, timestamp)
2. User-facing errors MUST be in plain English (not stack traces)
3. API MUST return appropriate HTTP status codes:
   - 400 Bad Request (validation errors)
   - 401 Unauthorized (missing/invalid auth)
   - 403 Forbidden (insufficient permissions)
   - 404 Not Found
   - 429 Too Many Requests (rate limit)
   - 500 Internal Server Error (unexpected errors)
4. System MUST implement circuit breakers for external dependencies
5. System MUST retry transient failures (exponential backoff)
6. Database transactions MUST rollback on error

#### NFR-3.3: Data Integrity

**Requirements**:
1. Database MUST enforce foreign key constraints
2. Telemetry MUST validate against schema before insertion
3. Database MUST use transactions for multi-step operations
4. System MUST detect and log duplicate telemetry messages
5. System MUST handle clock skew (accept telemetry ±5 minutes)

---

### NFR-4: Observability

#### NFR-4.1: Logging

**Requirements**:
1. All logs MUST be structured (JSON format)
2. Logs MUST include:
   - Timestamp (ISO 8601, UTC)
   - Level (DEBUG, INFO, WARN, ERROR)
   - Service name
   - Request ID (for tracing)
   - Message
   - Additional context (key-value pairs)
3. Logs MUST go to stdout (Docker captures)
4. Log levels MUST be configurable via environment variable
5. Production MUST use INFO level minimum
6. Sensitive data MUST NOT be logged (passwords, API keys)

**Example Log Format**:
```json
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "INFO",
  "service": "api",
  "request_id": "req-abc-123",
  "message": "Robot registered successfully",
  "robot_id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "admin-user-id"
}
```

#### NFR-4.2: Metrics

**Requirements**:
1. System MUST expose Prometheus-compatible metrics at `/api/v1/metrics`
2. Metrics MUST include:
   - Request count (by endpoint, status code)
   - Request duration histogram
   - Active connections (MQTT, WebSocket)
   - Telemetry messages received (by robot)
   - Database query duration
   - Error rate
   - Robot count by status
   - Alert count by severity
3. Metrics MUST be scraped by Prometheus every 15 seconds
4. Grafana dashboards MUST be provided for ops monitoring

**Key Metrics**:
```
# API
http_requests_total{endpoint="/api/v1/robots", method="GET", status="200"}
http_request_duration_seconds{endpoint="/api/v1/robots"}

# Telemetry
telemetry_messages_received_total{robot_id="..."}
telemetry_processing_duration_seconds

# Database
db_query_duration_seconds{query="insert_telemetry"}
db_connection_pool_active

# Alerts
alerts_triggered_total{severity="critical"}
alerts_active{severity="critical"}
```

#### NFR-4.3: Tracing (Optional for v0.1)

**Requirements** (future enhancement):
1. System SHOULD support distributed tracing (OpenTelemetry)
2. Trace context SHOULD propagate across services
3. Spans SHOULD be exported to Jaeger or similar

---

### NFR-5: Usability

#### NFR-5.1: Dashboard UX

**Requirements**:
1. Dashboard MUST be mobile-responsive (desktop-first, tablet works)
2. Dashboard MUST support dark mode and light mode
3. Dashboard MUST be accessible (WCAG 2.1 AA compliance):
   - Keyboard navigation works
   - Screen reader compatible
   - Color contrast ratios meet standards
   - Alt text for all images/icons
   - Proper heading hierarchy
4. Dashboard MUST show loading states (skeletons, spinners)
5. Dashboard MUST show informative error messages
6. Dashboard MUST provide contextual help (tooltips, info icons)

#### NFR-5.2: API Developer Experience

**Requirements**:
1. API MUST have comprehensive documentation
2. API MUST provide interactive explorer (Swagger UI)
3. API MUST return helpful error messages with actionable guidance
4. Python SDK MUST have example code in README
5. Python SDK MUST have type hints for IDE autocomplete
6. Getting started guide MUST allow first robot in <15 minutes

#### NFR-5.3: Internationalization (Future)

**Requirements** (not v0.1):
1. UI text SHOULD be externalized (i18n-ready)
2. Date/time SHOULD be localized based on user preference
3. Initial release: English only

---

### NFR-6: Maintainability

#### NFR-6.1: Code Quality

**Requirements**:
1. Code MUST follow language-specific style guides:
   - Go: `gofmt`, `golangci-lint`
   - Python: PEP 8, Black formatter
   - TypeScript: Prettier, ESLint
2. Code MUST have >80% test coverage (backend)
3. Code MUST have >70% test coverage (frontend)
4. All public functions MUST have documentation comments
5. Complex logic MUST have inline comments explaining "why"
6. No commented-out code in main branch
7. No hardcoded values (use config/env vars)

#### NFR-6.2: Testing

**Requirements**:
1. Unit tests MUST run in <5 seconds
2. Integration tests MUST run in <30 seconds
3. All tests MUST pass before merge
4. CI MUST run tests on every PR
5. Coverage MUST NOT decrease with new code
6. Critical paths MUST have integration tests:
   - Robot registration → telemetry → dashboard display
   - Telemetry → alert trigger → notification

#### NFR-6.3: Documentation

**Requirements**:
1. README MUST include:
   - Project description
   - Quick start (5-minute setup)
   - Architecture overview
   - Contributing guidelines
2. Code MUST have inline comments for non-obvious logic
3. API MUST have OpenAPI spec (auto-generated preferred)
4. Architecture diagrams MUST be up-to-date
5. Deployment guide MUST be included

---

### NFR-7: Deployment & Operations

#### NFR-7.1: Containerization

**Requirements**:
1. All services MUST have Dockerfiles
2. Docker images MUST use multi-stage builds (minimize size)
3. Docker images MUST run as non-root user
4. Docker images MUST use specific version tags (not `latest` in prod)
5. docker-compose.yml MUST allow local development with single command
6. Environment variables MUST be documented in .env.example

#### NFR-7.2: Configuration

**Requirements**:
1. All configuration MUST be via environment variables (12-factor app)
2. Sensible defaults MUST be provided
3. Required config MUST fail fast on startup if missing
4. Configuration validation MUST happen at startup
5. No secrets in docker-compose.yml (use .env file)

#### NFR-7.3: Database Migrations

**Requirements**:
1. Database schema MUST be versioned (migrations)
2. Migrations MUST be idempotent (safe to rerun)
3. Migrations MUST have rollback capability
4. Migrations MUST run automatically on startup OR have clear instructions
5. Migration tool: golang-migrate or similar

---

## User Interface & Experience

### UI-1: Design System

#### UI-1.1: Visual Design

**Color Palette**:

```
Status Colors:
- Online/Success:  #10B981 (Green 500)
- Offline/Error:   #EF4444 (Red 500)
- Warning:         #F59E0B (Amber 500)
- Unknown:         #6B7280 (Gray 500)

Primary Colors:
- Primary:         #3B82F6 (Blue 500)
- Primary Dark:    #2563EB (Blue 600)
- Primary Light:   #60A5FA (Blue 400)

Background (Light Mode):
- Background:      #FFFFFF (White)
- Surface:         #F9FAFB (Gray 50)
- Border:          #E5E7EB (Gray 200)

Background (Dark Mode):
- Background:      #111827 (Gray 900)
- Surface:         #1F2937 (Gray 800)
- Border:          #374151 (Gray 700)

Text (Light Mode):
- Primary:         #111827 (Gray 900)
- Secondary:       #6B7280 (Gray 500)

Text (Dark Mode):
- Primary:         #F9FAFB (Gray 50)
- Secondary:       #9CA3AF (Gray 400)
```

**Typography**:
- Font family: System fonts (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, ...`)
- Headings: 600 weight, 1.5 line height
- Body: 400 weight, 1.6 line height
- Code: Monospace font (`"Courier New", monospace`)

**Spacing**:
- Base unit: 4px
- Use multiples: 4, 8, 12, 16, 24, 32, 48, 64

**Border Radius**:
- Small (buttons, badges): 4px
- Medium (cards): 8px
- Large (modals): 12px

#### UI-1.2: Component Library

**Required Components**:
1. **Button**: Primary, secondary, danger, ghost variants
2. **Card**: Container for robot status, metrics
3. **Badge**: Status indicators, counts
4. **Table**: Data tables with sorting, filtering
5. **Chart**: Line charts, bar charts (using Recharts)
6. **Modal**: Dialogs for confirmations, forms
7. **Toast**: Notifications for alerts, successes, errors
8. **Dropdown**: Filters, menus
9. **Skeleton**: Loading placeholders
10. **Input**: Text, password, search fields
11. **Checkbox/Radio**: Form controls
12. **Tabs**: Page sections (robot detail page)

**Component Library Options**:
- shadcn/ui (recommended - Tailwind-based, customizable)
- Radix UI (headless, full control)
- Material-UI (feature-rich, heavier)

### UI-2: Page Layouts

#### UI-2.1: Authentication Pages

**Login Page**:
```
┌─────────────────────────────────────┐
│                                     │
│          HumanoidOps Logo           │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  Username                   │   │
│  ├─────────────────────────────┤   │
│  │  Password                   │   │
│  ├─────────────────────────────┤   │
│  │  [Login Button]             │   │
│  └─────────────────────────────┘   │
│                                     │
│  Open-source fleet operations       │
│  platform for humanoid robots       │
│                                     │
└─────────────────────────────────────┘
```

#### UI-2.2: Main Dashboard Layout

**Desktop Layout**:
```
┌──────────────────────────────────────────────────────────┐
│  [Logo] HumanoidOps        [User Menu] [Dark/Light] [⚙️] │
├──────────────┬───────────────────────────────────────────┤
│              │  Fleet Overview                           │
│ 🏠 Dashboard │                                           │
│ 🚨 Alerts    │  Fleet Summary: 125 robots                │
│ 📋 Audit     │  Online: 118 | Offline: 5 | Error: 2     │
│              │                                           │
│ (Admin only) │  Filters: [Status ▼] [Model ▼] [Search]  │
│ ⚙️ Settings  │                                           │
│ 👥 Users     │  ┌────────┐ ┌────────┐ ┌────────┐        │
│              │  │ 🟢 Bot │ │ 🟢 Bot │ │ 🔴 Bot │  ...   │
│              │  │ #001   │ │ #002   │ │ #003   │        │
│              │  │ 85% 🔋 │ │ 92% 🔋 │ │ 15% 🔋 │        │
│              │  │ 2m ago │ │ 30s    │ │ OFFLINE│        │
│              │  └────────┘ └────────┘ └────────┘        │
│              │                                           │
│              │  Pagination: < 1 2 3 4 5 >                │
└──────────────┴───────────────────────────────────────────┘
```

**Mobile/Tablet** (Responsive):
- Sidebar collapses to hamburger menu
- Robot cards stack vertically
- Summary stats remain at top

#### UI-2.3: Robot Detail Page

```
┌──────────────────────────────────────────────────────────┐
│  ← Back to Fleet                                         │
├──────────────────────────────────────────────────────────┤
│  Robot #HV1-00123                        🟢 Online        │
│  Model: HumanoidV1  |  Location: Warehouse A             │
├──────────────────────────────────────────────────────────┤
│  [Overview] [Telemetry] [Alerts] [Settings]              │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Current Status                                           │
│  ┌────────────────────┬────────────────────┐             │
│  │ Battery: 85% 🔋    │ CPU Temp: 65°C     │             │
│  │ Status: Working    │ CPU Load: 45%      │             │
│  │ Last Seen: 2s ago  │ Memory: 60%        │             │
│  └────────────────────┴────────────────────┘             │
│                                                           │
│  Battery Level (Last 24h)                                 │
│  ┌──────────────────────────────────────────────────┐    │
│  │                      📈 Line Chart               │    │
│  │                                                   │    │
│  └──────────────────────────────────────────────────┘    │
│                                                           │
│  CPU Temperature (Last 24h)                               │
│  ┌──────────────────────────────────────────────────┐    │
│  │                      📈 Line Chart                │    │
│  │                                                   │    │
│  └──────────────────────────────────────────────────┘    │
│                                                           │
│  Recent Alerts (3)                                        │
│  ┌──────────────────────────────────────────────────┐    │
│  │ 🔴 Low Battery - 10:30 AM - [Acknowledge]        │    │
│  │ 🟡 High Temp   - 09:15 AM - Acknowledged         │    │
│  │ 🔴 Offline     - 08:00 AM - Acknowledged         │    │
│  └──────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

#### UI-2.4: Alerts Page

```
┌──────────────────────────────────────────────────────────┐
│  Alerts                                                   │
├──────────────────────────────────────────────────────────┤
│  Filters: [All ▼] [Critical ▼] [Last 24h ▼]  [Search]    │
│  [Acknowledge Selected] [Export CSV]                      │
├──────────────────────────────────────────────────────────┤
│  Active Alerts (7)                                        │
│  ┌──────────────────────────────────────────────────┐    │
│  │ ☑️ 🔴 Robot Offline - #HV1-00045                 │    │
│  │    Triggered: 2m ago  |  Duration: 2m 15s        │    │
│  │    [View Robot] [Acknowledge]                    │    │
│  ├──────────────────────────────────────────────────┤    │
│  │ ☑️ 🔴 Low Battery (8%) - #HV1-00012              │    │
│  │    Triggered: 5m ago  |  Battery dropping fast   │    │
│  │    [View Robot] [Acknowledge]                    │    │
│  └──────────────────────────────────────────────────┘    │
│                                                           │
│  Acknowledged Alerts (15) - [Expand ▼]                    │
└──────────────────────────────────────────────────────────┘
```

### UI-3: Responsive Design

**Breakpoints**:
```css
/* Mobile */
@media (max-width: 640px) { ... }

/* Tablet */
@media (min-width: 641px) and (max-width: 1024px) { ... }

/* Desktop */
@media (min-width: 1025px) { ... }
```

**Responsive Behavior**:
1. **Mobile**: Single column, hamburger menu, stacked cards
2. **Tablet**: Two-column grid, collapsible sidebar
3. **Desktop**: Multi-column grid, persistent sidebar

### UI-4: Accessibility (WCAG 2.1 AA)

**Requirements**:
1. **Keyboard Navigation**: All interactive elements accessible via Tab
2. **Focus Indicators**: Visible focus states for keyboard users
3. **Screen Readers**: Proper ARIA labels, semantic HTML
4. **Color Contrast**: Minimum 4.5:1 for text, 3:1 for UI components
5. **Alt Text**: All images and icons have descriptive alt text
6. **Heading Hierarchy**: Proper H1 → H2 → H3 structure
7. **Form Labels**: All inputs have associated labels
8. **Error Messages**: Clear, linked to form fields

---

## Acceptance Criteria

The following criteria MUST all pass for v0.1 to be considered complete:

### Functional Acceptance

- [ ] **AC-1**: Admin can register 10 simulated robots via API
- [ ] **AC-2**: Robots successfully send telemetry via MQTT at 1Hz
- [ ] **AC-3**: Telemetry appears in database within 1 second of sending
- [ ] **AC-4**: Dashboard displays all 10 robots with correct status
- [ ] **AC-5**: Dashboard updates in real-time when robot goes offline
- [ ] **AC-6**: Dashboard updates in real-time when robot comes online
- [ ] **AC-7**: Alert triggers when robot offline >30 seconds
- [ ] **AC-8**: Alert triggers when battery <20%
- [ ] **AC-9**: Alert appears in dashboard within 5 seconds of trigger
- [ ] **AC-10**: User can acknowledge alert (recorded in audit log)
- [ ] **AC-11**: Audit log shows all robot registrations with correct user and timestamp
- [ ] **AC-12**: Audit log shows all alert acknowledgments
- [ ] **AC-13**: Python SDK can send telemetry successfully
- [ ] **AC-14**: Robot simulator can run 50 concurrent robots
- [ ] **AC-15**: Dashboard loads in <2 seconds with 100 robots

### API Acceptance

- [ ] **AC-16**: All API endpoints documented in OpenAPI spec
- [ ] **AC-17**: OpenAPI spec validates with official tools
- [ ] **AC-18**: Swagger UI available at `/api/docs`
- [ ] **AC-19**: All endpoints require authentication except `/health`, `/metrics`, `/openapi`
- [ ] **AC-20**: Authentication with invalid credentials returns 401
- [ ] **AC-21**: Authorization failure returns 403
- [ ] **AC-22**: Validation errors return 400 with descriptive messages
- [ ] **AC-23**: Rate limiting returns 429 after threshold

### Security Acceptance

- [ ] **AC-24**: All API communications over HTTPS in production
- [ ] **AC-25**: MQTT connections use TLS on port 8883
- [ ] **AC-26**: Passwords hashed with bcrypt (not plaintext)
- [ ] **AC-27**: JWT tokens expire after 24 hours
- [ ] **AC-28**: RBAC enforced (viewer cannot register robots)
- [ ] **AC-29**: Dependency scan shows zero critical vulnerabilities
- [ ] **AC-30**: Security headers present (HSTS, X-Frame-Options, etc.)

### Performance Acceptance

- [ ] **AC-31**: API p95 latency <200ms under normal load
- [ ] **AC-32**: Dashboard loads in <2s for 100 robots
- [ ] **AC-33**: System handles 100 telemetry messages/second
- [ ] **AC-34**: Database query time <50ms for dashboard data
- [ ] **AC-35**: WebSocket latency <100ms from event to dashboard

### Testing Acceptance

- [ ] **AC-36**: Backend test coverage >80%
- [ ] **AC-37**: Frontend test coverage >70%
- [ ] **AC-38**: All unit tests pass
- [ ] **AC-39**: All integration tests pass
- [ ] **AC-40**: Load test completes without errors (100 robots, 1 hour)

### Documentation Acceptance

- [ ] **AC-41**: README has complete quick start guide
- [ ] **AC-42**: Architecture documentation exists and is accurate
- [ ] **AC-43**: API documentation complete (all endpoints)
- [ ] **AC-44**: Deployment guide exists and has been tested
- [ ] **AC-45**: Python SDK has API reference documentation
- [ ] **AC-46**: Contributing guide exists

### Deployment Acceptance

- [ ] **AC-47**: `docker-compose up` successfully starts all services
- [ ] **AC-48**: Database migrations run automatically
- [ ] **AC-49**: Environment variables documented in `.env.example`
- [ ] **AC-50**: Health check endpoint returns 200 when healthy

---

## Out of Scope

The following features are **explicitly NOT included** in v0.1 to maintain focus and ensure timely delivery:

### Out of Scope: Tele-operation
- Manual remote control of robots
- Joystick/gamepad interface
- Video streaming from robot cameras
- Real-time robot command execution

**Rationale**: Complex feature requiring significant security considerations, low-latency infrastructure, and safety mechanisms. Planned for v0.3+.

### Out of Scope: AI/ML Features
- AI training pipelines
- Model deployment to robots
- Inference tracking
- Dataset management
- Computer vision pipelines

**Rationale**: Outside core platform scope. Users can integrate with external ML platforms. May add hooks in v0.2+.

### Out of Scope: Multi-Tenancy
- Organization/workspace isolation
- Per-tenant databases
- SaaS billing
- User invitations across tenants

**Rationale**: MVP targets single-organization self-hosted deployments. Multi-tenancy adds significant complexity. Planned for v1.0 (SaaS offering).

### Out of Scope: Mobile Applications
- Native iOS app
- Native Android app
- Progressive Web App (PWA)

**Rationale**: Limited resources. Web dashboard works on tablets. Native apps planned post-v1.0.

### Out of Scope: Advanced Analytics
- Custom dashboards/widgets
- Business intelligence (BI) integration
- Predictive maintenance ML models
- Anomaly detection
- Report generation

**Rationale**: MVP focuses on monitoring, not analytics. Can add in v0.2+ based on user feedback.

### Out of Scope: Integrations
- Slack notifications
- PagerDuty integration
- Email alerts
- Webhook support
- Third-party API integrations

**Rationale**: Core platform first, integrations second. Will add webhook support in v0.2, specific integrations based on demand.

### Out of Scope: Advanced Features
- OTA (Over-the-Air) firmware updates
- Robot task scheduler
- Mission planning
- Collaborative robots (fleet coordination)
- Digital twin visualization
- Map visualization of robot locations

**Rationale**: Complex features requiring significant development. Planned for future versions based on user needs.

### Out of Scope: Enterprise Features (for now)
- SSO (SAML, OAuth)
- Advanced RBAC (custom roles)
- Compliance reports (SOC2, ISO27001)
- SLA monitoring
- Disaster recovery automation

**Rationale**: MVP targets developer and small team adoption. Enterprise features in v1.0+.

---

## Dependencies & Assumptions

### Technical Dependencies

**Required Infrastructure**:
- Docker 20.10+ and Docker Compose 2.0+
- PostgreSQL 14+ with TimescaleDB extension 2.0+
- EMQX 5.0+ or Mosquitto 2.0+ MQTT broker
- Linux host (Ubuntu 22.04 recommended for production)

**Backend Dependencies** (Go 1.21+):
- `github.com/gin-gonic/gin` - Web framework
- `gorm.io/gorm` - ORM
- `gorm.io/driver/postgres` - PostgreSQL driver
- `github.com/golang-jwt/jwt/v5` - JWT
- `github.com/eclipse/paho.mqtt.golang` - MQTT client
- `golang.org/x/crypto/bcrypt` - Password hashing

**Frontend Dependencies** (Node.js 18+):
- `next` 14+ - React framework
- `react` 18+ - UI library
- `@tanstack/react-query` - Data fetching
- `axios` - HTTP client
- `recharts` - Charting
- `tailwindcss` - Styling

**Agent Dependencies** (Python 3.10+):
- `paho-mqtt` 2.0+ - MQTT client
- `requests` 2.31+ - HTTP client
- `pydantic` 2.0+ - Data validation
- `python-dotenv` 1.0+ - Config management

**Development Tools**:
- Git 2.30+
- make (optional, for convenience)
- curl or Postman (API testing)

### Operational Dependencies

**Deployment**:
- Server with 4GB RAM minimum (8GB recommended for 1000 robots)
- 100GB disk space (for telemetry storage)
- Internet connectivity (for dependency installation)
- Domain name and SSL certificate (Let's Encrypt in production)

**Monitoring** (optional but recommended):
- Prometheus for metrics collection
- Grafana for visualization
- Log aggregation (ELK, Loki, or similar)

### Assumptions

**Technical Assumptions**:
1. ✓ Robots have network connectivity (WiFi, cellular, or Ethernet)
2. ✓ Robots can handle TLS/HTTPS (sufficient CPU for encryption)
3. ✓ Robots can run Python 3.10+ or implement MQTT protocol
4. ✓ Deployment environment supports Docker
5. ✓ Clock synchronization (NTP) available on all systems
6. ✓ IPv4 networking available (IPv6 support future)

**Business Assumptions**:
1. ✓ Single organization/tenant per deployment (v0.1)
2. ✓ Self-hosted deployment model (not managed SaaS)
3. ✓ Developer/technical users (not end consumers)
4. ✓ English language support sufficient for MVP
5. ✓ Open-source Apache 2.0 license acceptable

**Operational Assumptions**:
1. ✓ Admin manually creates first user account (bootstrap)
2. ✓ Database backups managed by deployment team
3. ✓ SSL certificate provisioning handled externally
4. ✓ Monitoring/alerting configured separately (Prometheus/Grafana)
5. ✓ 24/7 operation not required for v0.1 (dev/staging environments)

**User Assumptions**:
1. ✓ Users comfortable with web dashboards
2. ✓ Admins have basic Docker/command-line knowledge
3. ✓ Developers can read API documentation
4. ✓ Users accept 99% uptime SLO (not 99.9%)

---

## Risks & Mitigations

### Technical Risks

#### RISK-1: TimescaleDB Performance Degradation
**Probability**: Medium  
**Impact**: High  
**Description**: As telemetry data grows, query performance may degrade.

**Mitigation**:
- Implement data retention policy (30 days)
- Enable TimescaleDB compression (7 days)
- Add database query caching (Redis) if needed
- Monitor query performance from day 1
- Load test with 1000 robots × 30 days data before launch

---

#### RISK-2: MQTT Broker Bottleneck
**Probability**: Medium  
**Impact**: Medium  
**Description**: MQTT broker may become bottleneck at 1000+ robots.

**Mitigation**:
- Choose EMQX (supports clustering)
- Provide HTTPS fallback for telemetry
- Load test MQTT with 1000 concurrent connections
- Document scaling plan (MQTT cluster) for v0.2
- Monitor MQTT broker metrics (connections, throughput)

---

#### RISK-3: WebSocket Scalability
**Probability**: Medium  
**Impact**: Medium  
**Description**: WebSocket connections don't scale horizontally easily.

**Mitigation**:
- Accept limitation for v0.1 (single API server)
- Document that WebSocket requires sticky sessions if load-balanced
- Future: Use Redis Pub/Sub for multi-server WebSocket
- Monitor concurrent WebSocket connections

---

#### RISK-4: Database Migration Failures
**Probability**: Low  
**Impact**: High  
**Description**: Schema changes could corrupt production data.

**Mitigation**:
- Test all migrations on staging data
- Implement migration rollback scripts
- Backup database before migrations
- Use migration tools (golang-migrate)
- Document manual recovery procedures

---

### Security Risks

#### RISK-5: Credential Compromise
**Probability**: Medium  
**Impact**: Critical  
**Description**: Robot API keys or user passwords could be compromised.

**Mitigation**:
- Implement rate limiting on authentication
- Hash API keys in database
- Support API key rotation
- Monitor for unusual authentication patterns
- Require strong passwords (minimum 12 chars, complexity)
- Audit log all authentication events

---

#### RISK-6: Dependency Vulnerabilities
**Probability**: High (over time)  
**Impact**: Medium-High  
**Description**: Third-party dependencies may have security vulnerabilities.

**Mitigation**:
- Enable Dependabot on GitHub
- Weekly dependency scans (Snyk)
- Automated security patches in CI/CD
- Pin dependency versions (not `latest`)
- Subscribe to security advisories for major dependencies
- Monthly security review

---

#### RISK-7: MQTT Authentication Bypass
**Probability**: Low  
**Impact**: Critical  
**Description**: Flaw in MQTT authentication could allow unauthorized telemetry.

**Mitigation**:
- Use established MQTT broker (EMQX) with proven security
- Enable TLS for all MQTT connections
- Test authentication logic thoroughly
- Implement API key validation before processing telemetry
- Audit log all MQTT authentication failures

---

### Operational Risks

#### RISK-8: Insufficient Documentation
**Probability**: Medium  
**Impact**: Medium  
**Description**: Users unable to deploy/use platform due to poor docs.

**Mitigation**:
- Prioritize documentation as P0 deliverable
- Test docs with external beta users
- Include troubleshooting section
- Provide video walkthrough
- Community feedback on docs quality

---

#### RISK-9: Resource Capacity Planning
**Probability**: Medium  
**Impact**: Medium  
**Description**: Underestimating resource needs leads to performance issues.

**Mitigation**:
- Document minimum and recommended specs
- Provide capacity planning calculator
- Load test before launch
- Monitor resource usage (CPU, memory, disk)
- Implement alerts for resource exhaustion

---

#### RISK-10: Inadequate Testing
**Probability**: Medium  
**Impact**: High  
**Description**: Bugs in production due to insufficient test coverage.

**Mitigation**:
- Enforce >80% code coverage
- Implement integration tests for critical paths
- Load testing before launch
- Beta testing with external users (2+ orgs)
- Staging environment mirrors production
- Rollback plan for releases

---

### Product Risks

#### RISK-11: Feature Creep
**Probability**: High  
**Impact**: Medium  
**Description**: Scope expansion delays v0.1 launch.

**Mitigation**:
- Strict adherence to PRD scope
- Regular scope reviews with "Out of Scope" section
- Defer nice-to-have features to v0.2
- Product manager enforces priorities
- Time-boxed sprints

---

#### RISK-12: User Adoption
**Probability**: Medium  
**Impact**: High  
**Description**: Platform too complex or missing critical features for adoption.

**Mitigation**:
- Beta testing with 2+ external organizations
- User feedback sessions during development
- Prioritize developer experience (quick start)
- Provide working examples and tutorials
- Active community building (Discord/GitHub Discussions)

---

## Release Plan

### Pre-Release (3 weeks before v0.1)

**Week -3**:
- [ ] Complete all development
- [ ] All acceptance criteria met
- [ ] Test coverage >80% backend, >70% frontend
- [ ] Documentation complete (README, API docs, deployment guide)
- [ ] Security scan shows zero critical/high vulnerabilities

**Week -2**:
- [ ] Beta release to 2 external organizations
- [ ] Set up staging environment (mirrors production)
- [ ] Performance testing (100 robots, 1000 robots)
- [ ] Security review (internal or external)
- [ ] Create release notes draft

**Week -1**:
- [ ] Address beta feedback
- [ ] Final bug fixes
- [ ] Tag release candidate (v0.1-rc1)
- [ ] Test deployment from scratch (clean machine)
- [ ] Update CHANGELOG.md
- [ ] Prepare announcement (blog post, social media)

### Release (v0.1 Launch Day)

**Launch Checklist**:
- [ ] Tag final release (v0.1.0)
- [ ] Publish Docker images to Docker Hub
- [ ] Publish Python SDK to PyPI (`humanoidops-agent`)
- [ ] Create GitHub Release with release notes
- [ ] Update README with installation instructions
- [ ] Publish announcement blog post
- [ ] Share on Reddit (r/robotics, r/golang, r/Python), blogs, tech websites, other social networks
- [ ] Monitor GitHub Issues for bug reports

### Post-Release (First 30 Days)

**Week 1-2**:
- [ ] Monitor for critical bugs
- [ ] Rapid response to issue reports (<24h)
- [ ] Daily check of GitHub Issues
- [ ] Community engagement (answer questions)

**Week 3-4**:
- [ ] Collect user feedback
- [ ] Analyze usage metrics (if telemetry opt-in)
- [ ] Prioritize v0.2 features based on feedback
- [ ] Plan v0.2 roadmap
- [ ] Security patches if needed

---

## Appendices

### Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Alert** | Automated notification triggered when system detects issue (e.g., robot offline) |
| **Audit Log** | Immutable record of all administrative actions for security/compliance |
| **EMQX** | Open-source MQTT broker supporting millions of concurrent connections |
| **Fleet** | Collection of robots managed by the platform |
| **HumanoidOps** | This platform - operations software for humanoid robot fleets |
| **JWT** | JSON Web Token - compact authentication token format |
| **MQTT** | Message Queuing Telemetry Transport - lightweight pub/sub protocol |
| **mTLS** | Mutual TLS - both client and server authenticate with certificates |
| **RBAC** | Role-Based Access Control - permission model based on user roles |
| **Telemetry** | Data sent from robots (battery, temperature, location, etc.) |
| **TimescaleDB** | PostgreSQL extension optimized for time-series data |
| **WebSocket** | Full-duplex communication protocol for real-time updates |

### Appendix B: References

**External Standards**:
- MQTT 5.0 Specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- OpenAPI 3.0 Specification: https://spec.openapis.org/oas/v3.0.3
- JWT RFC 7519: https://datatracker.ietf.org/doc/html/rfc7519
- WCAG 2.1 AA: https://www.w3.org/WAI/WCAG21/quickref/

**Similar Projects** (for reference):
- ROS 2 (Robot Operating System): https://www.ros.org/
- K8s (Kubernetes for orchestration): https://kubernetes.io/
- Prometheus (monitoring): https://prometheus.io/

### Appendix C: Success Metrics Dashboard

**After 3 Months Post-Launch**:

| Metric | Target | Measurement |
|--------|--------|-------------|
| GitHub Stars | 100+ | GitHub API |
| External Deployments | 5+ | Self-reported (opt-in telemetry) |
| Contributors | 10+ | GitHub contributors count |
| Issues Opened | 20+ | GitHub Issues (indicates usage) |
| Issues Resolved | 80%+ | Closed/Total ratio |
| Documentation Views | 1000+ | GitHub README views |
| PyPI Downloads | 500+ | PyPI stats for `humanoidops-agent` |
| Community Members | 50+ | Discord/Discussions members |

