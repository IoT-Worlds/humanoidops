# TODO.md - HumanoidOps v0.1 Implementation Checklist

**Project**: HumanoidOps Fleet Operations Platform  
**Version**: 0.1 (MVP)  
**Timeline**: 8 weeks  
**References**: [PRD.md](./PRD.md) | [PLAN.md](./PLAN.md)

---

## 📋 How to Use This Document

- [ ] Check off items as you complete them
- [ ] Each week builds on the previous week
- [ ] Critical path items marked with 🔴
- [ ] Optional/nice-to-have items marked with 🟡
- [ ] Keep this file updated in Git
- [ ] Review progress at end of each week

---

## Pre-Project Setup

### Prerequisites Check

- [ ] Docker 20.10+ installed
- [ ] Docker Compose 2.0+ installed
- [ ] Go 1.21+ installed
- [ ] Python 3.10+ installed
- [ ] Node.js 18+ installed
- [ ] Git 2.30+ installed
- [ ] GitHub account created
- [ ] Code editor setup (VS Code recommended)

---

## Week 1: Repository Setup & Development Environment

**Goal**: Create project foundation with working local development environment

### Day 1: Monday - Repository & Project Structure

#### Morning: GitHub Setup
- [ ] Create GitHub repository `humanoidops`
  ```bash
  # Option 1: Via GitHub CLI
  gh repo create humanoidops --public --description "Open-source operations platform for humanoid robots"
  
  # Option 2: Via GitHub web interface
  # https://github.com/new
  ```
- [ ] Clone repository locally
  ```bash
  git clone https://github.com/YOUR_USERNAME/humanoidops.git
  cd humanoidops
  ```
- [ ] Create initial README.md
  ```bash
  touch README.md
  # Add project title, description, quick start placeholder
  ```
- [ ] Add LICENSE file (Apache 2.0 or MIT)
  ```bash
  # Copy from https://choosealicense.com/
  curl https://www.apache.org/licenses/LICENSE-2.0.txt -o LICENSE
  ```
- [ ] Create .gitignore
  ```bash
  # Add Go, Python, Node.js, IDE entries
  cat > .gitignore << 'EOF'
  # Go
  *.exe
  *.out
  vendor/
  
  # Python
  __pycache__/
  *.py[cod]
  .venv/
  venv/
  *.egg-info/
  
  # Node.js
  node_modules/
  .next/
  .env.local
  
  # IDEs
  .vscode/
  .idea/
  *.swp
  
  # Secrets
  .env
  *.pem
  *.key
  EOF
  ```

#### Afternoon: Folder Structure
- [ ] Create backend folder structure
  ```bash
  mkdir -p backend/{cmd/{api,mqtt-handler},internal/{config,middleware,handlers,services,repositories,models,websocket,database,metrics,mqtt,telemetry,alert},pkg/{jwt,validator},migrations}
  ```
- [ ] Create agent folder structure
  ```bash
  mkdir -p agent/{humanoidops_agent,examples,tests}
  ```
- [ ] Create dashboard folder structure
  ```bash
  mkdir dashboard
  # Will initialize Next.js later
  ```
- [ ] Create docs folder structure
  ```bash
  mkdir -p docs/{api,architecture,guides,images}
  ```
- [ ] Create supporting folders
  ```bash
  mkdir -p {scripts,docker,loadtests,.github/workflows}
  ```

#### End of Day: Initial Commit
- [ ] Create main branch protection rules on GitHub
  - Require pull request reviews
  - Require status checks to pass
- [ ] Create development branch
  ```bash
  git checkout -b develop
  ```
- [ ] Initial commit
  ```bash
  git add .
  git commit -m "feat: initial project structure"
  git push -u origin develop
  ```

---

### Day 2: Tuesday - Docker & Database Setup

#### Morning: Docker Compose Configuration
- [ ] 🔴 Create `docker-compose.yml`
  ```yaml
  # See PLAN.md → Deployment Architecture → Development Environment
  # Include: PostgreSQL+TimescaleDB, EMQX, Prometheus, Grafana
  ```
- [ ] Create `.env.example`
  ```bash
  cat > .env.example << 'EOF'
  # Database
  DATABASE_URL=postgresql://humanoidops:development_password@localhost:5432/humanoidops
  POSTGRES_DB=humanoidops
  POSTGRES_USER=humanoidops
  POSTGRES_PASSWORD=development_password
  
  # MQTT
  MQTT_BROKER_URL=tcp://localhost:1883
  MQTT_USERNAME=
  MQTT_PASSWORD=
  
  # API
  API_PORT=8080
  JWT_SECRET=change-this-in-production-min-32-chars
  JWT_EXPIRY=24h
  
  # Frontend
  NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1
  NEXT_PUBLIC_WS_URL=ws://localhost:8080/api/v1/ws
  EOF
  ```
- [ ] Copy to actual .env file
  ```bash
  cp .env.example .env
  ```

#### Afternoon: Docker Testing
- [ ] 🔴 Test PostgreSQL + TimescaleDB
  ```bash
  docker-compose up -d postgres
  
  # Verify TimescaleDB extension
  docker-compose exec postgres psql -U humanoidops -c "CREATE EXTENSION IF NOT EXISTS timescaledb;"
  docker-compose exec postgres psql -U humanoidops -c "SELECT default_version, installed_version FROM pg_available_extensions WHERE name = 'timescaledb';"
  ```
- [ ] 🔴 Test EMQX broker
  ```bash
  docker-compose up -d emqx
  
  # Access dashboard: http://localhost:18083
  # Default credentials: admin / public
  ```
- [ ] Test Prometheus
  ```bash
  docker-compose up -d prometheus
  
  # Access: http://localhost:9090
  ```
- [ ] Test Grafana
  ```bash
  docker-compose up -d grafana
  
  # Access: http://localhost:3001
  # Default credentials: admin / admin
  ```

#### End of Day: Prometheus Configuration
- [ ] Create `docker/prometheus.yml`
  ```yaml
  global:
    scrape_interval: 15s
    evaluation_interval: 15s
  
  scrape_configs:
    - job_name: 'humanoidops-api'
      static_configs:
        - targets: ['api:8080']
      metrics_path: '/api/v1/metrics'
    
    - job_name: 'humanoidops-mqtt-handler'
      static_configs:
        - targets: ['mqtt-handler:9091']
      metrics_path: '/metrics'
  ```
- [ ] Commit Docker configuration
  ```bash
  git add docker-compose.yml .env.example docker/
  git commit -m "feat: add Docker Compose configuration"
  ```

---

### Day 3: Wednesday - Backend & Agent Initialization

#### Morning: Go Backend Setup
- [ ] 🔴 Initialize Go module
  ```bash
  cd backend
  go mod init github.com/YOUR_USERNAME/humanoidops
  ```
- [ ] Install Go dependencies
  ```bash
  go get github.com/gin-gonic/gin
  go get gorm.io/gorm
  go get gorm.io/driver/postgres
  go get github.com/golang-jwt/jwt/v5
  go get github.com/eclipse/paho.mqtt.golang
  go get golang.org/x/crypto/bcrypt
  go get github.com/google/uuid
  go get github.com/joho/godotenv
  go get github.com/prometheus/client_golang/prometheus
  go get github.com/prometheus/client_golang/prometheus/promhttp
  ```
- [ ] Create `backend/cmd/api/main.go` skeleton
  ```go
  package main
  
  import (
      "log"
      "github.com/gin-gonic/gin"
  )
  
  func main() {
      router := gin.Default()
      
      router.GET("/api/v1/health", func(c *gin.Context) {
          c.JSON(200, gin.H{"status": "healthy"})
      })
      
      log.Println("API server starting on :8080")
      router.Run(":8080")
  }
  ```
- [ ] Test basic API
  ```bash
  go run cmd/api/main.go
  # In another terminal: curl http://localhost:8080/api/v1/health
  ```

#### Afternoon: Python Agent Setup
- [ ] 🔴 Create `agent/pyproject.toml`
  ```toml
  [build-system]
  requires = ["setuptools>=65.0", "wheel"]
  build-backend = "setuptools.build_meta"
  
  [project]
  name = "humanoidops-agent"
  version = "0.1.0"
  description = "Robot agent SDK for HumanoidOps"
  requires-python = ">=3.10"
  dependencies = [
      "paho-mqtt>=2.0.0",
      "requests>=2.31.0",
      "pydantic>=2.0.0",
      "python-dotenv>=1.0.0",
      "pyyaml>=6.0",
  ]
  
  [project.optional-dependencies]
  dev = [
      "pytest>=7.0.0",
      "pytest-cov>=4.0.0",
      "black>=23.0.0",
      "flake8>=6.0.0",
  ]
  
  [project.scripts]
  humanoidops-simulator = "humanoidops_agent.cli:main"
  ```
- [ ] Create `agent/humanoidops_agent/__init__.py`
  ```python
  """HumanoidOps Robot Agent SDK"""
  
  __version__ = "0.1.0"
  
  from .client import HumanoidOpsClient
  from .simulator import RobotSimulator
  
  __all__ = ["HumanoidOpsClient", "RobotSimulator"]
  ```
- [ ] Install agent in development mode
  ```bash
  cd agent
  python -m venv .venv
  source .venv/bin/activate  # On Windows: .venv\Scripts\activate
  pip install -e ".[dev]"
  ```
- [ ] Test installation
  ```bash
  python -c "import humanoidops_agent; print(humanoidops_agent.__version__)"
  ```

#### End of Day: Frontend Initialization
- [ ] 🔴 Initialize Next.js
  ```bash
  npx create-next-app@latest dashboard --typescript --tailwind --app --no-src-dir
  cd dashboard
  ```
- [ ] Install additional dependencies
  ```bash
  npm install @tanstack/react-query axios recharts
  npm install lucide-react class-variance-authority clsx tailwind-merge
  npm install -D @types/node
  ```
- [ ] Test Next.js
  ```bash
  npm run dev
  # Access: http://localhost:3000
  ```
- [ ] Commit initialization
  ```bash
  git add .
  git commit -m "feat: initialize backend, agent, and frontend"
  ```

---

### Day 4: Thursday - CI/CD & Documentation

#### Morning: GitHub Actions Setup
- [ ] 🔴 Create `.github/workflows/backend.yml`
  ```yaml
  name: Backend CI
  
  on:
    push:
      branches: [ main, develop ]
      paths:
        - 'backend/**'
    pull_request:
      branches: [ main, develop ]
      paths:
        - 'backend/**'
  
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - name: Set up Go
          uses: actions/setup-go@v4
          with:
            go-version: '1.21'
        
        - name: Install dependencies
          working-directory: backend
          run: go mod download
        
        - name: Run tests
          working-directory: backend
          run: go test ./... -v -coverprofile=coverage.out
        
        - name: Upload coverage
          uses: codecov/codecov-action@v3
          with:
            files: ./backend/coverage.out
  ```
- [ ] Create `.github/workflows/agent.yml`
  ```yaml
  name: Agent CI
  
  on:
    push:
      branches: [ main, develop ]
      paths:
        - 'agent/**'
    pull_request:
      branches: [ main, develop ]
      paths:
        - 'agent/**'
  
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - name: Set up Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.10'
        
        - name: Install dependencies
          working-directory: agent
          run: |
            pip install -e ".[dev]"
        
        - name: Run tests
          working-directory: agent
          run: |
            pytest --cov=humanoidops_agent --cov-report=xml
        
        - name: Upload coverage
          uses: codecov/codecov-action@v3
          with:
            files: ./agent/coverage.xml
  ```
- [ ] Create `.github/workflows/dashboard.yml`
  ```yaml
  name: Dashboard CI
  
  on:
    push:
      branches: [ main, develop ]
      paths:
        - 'dashboard/**'
    pull_request:
      branches: [ main, develop ]
      paths:
        - 'dashboard/**'
  
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: '18'
        
        - name: Install dependencies
          working-directory: dashboard
          run: npm ci
        
        - name: Run linter
          working-directory: dashboard
          run: npm run lint
        
        - name: Build
          working-directory: dashboard
          run: npm run build
  ```

#### Afternoon: Documentation
- [ ] Update `README.md` with comprehensive content
  ```markdown
  # HumanoidOps
  
  Open-source operations platform for humanoid robot fleets.
  
  ## Quick Start
  
  ### Prerequisites
  - Docker & Docker Compose
  - Go 1.21+ (for backend development)
  - Python 3.10+ (for agent development)
  - Node.js 18+ (for dashboard development)
  
  ### Run Locally
  
  1. Clone repository:
     ```bash
     git clone https://github.com/YOUR_USERNAME/humanoidops.git
     cd humanoidops
     ```
  
  2. Start services:
     ```bash
     docker-compose up -d
     ```
  
  3. Access:
     - Dashboard: http://localhost:3000
     - API: http://localhost:8080
     - EMQX Dashboard: http://localhost:18083 (admin/public)
     - Grafana: http://localhost:3001 (admin/admin)
  
  ## Architecture
  
  See [docs/architecture/ARCHITECTURE.md](docs/architecture/ARCHITECTURE.md)
  
  ## Documentation
  
  - [API Reference](docs/api/README.md)
  - [Developer Guide](docs/guides/DEVELOPMENT.md)
  - [Deployment Guide](docs/guides/DEPLOYMENT.md)
  
  ## License
  
  Apache 2.0
  ```
- [ ] Create `docs/ARCHITECTURE.md`
  - Copy C4 diagrams from PLAN.md
  - Add component descriptions
- [ ] Create `docs/guides/DEVELOPMENT.md`
  ```markdown
  # Development Guide
  
  ## Local Development
  
  ### Backend
  ```bash
  cd backend
  go run cmd/api/main.go
  ```
  
  ### Agent
  ```bash
  cd agent
  source .venv/bin/activate
  python -m humanoidops_agent.cli --num-robots 10
  ```
  
  ### Dashboard
  ```bash
  cd dashboard
  npm run dev
  ```
  
  ## Testing
  
  ### Backend Tests
  ```bash
  cd backend
  go test ./... -v
  ```
  
  ### Agent Tests
  ```bash
  cd agent
  pytest
  ```
  ```
- [ ] Create `CONTRIBUTING.md`
  ```markdown
  # Contributing to HumanoidOps
  
  ## Development Workflow
  
  1. Fork the repository
  2. Create feature branch: `git checkout -b feature/my-feature`
  3. Make changes and commit: `git commit -am 'feat: add new feature'`
  4. Push to branch: `git push origin feature/my-feature`
  5. Create Pull Request
  
  ## Commit Message Format
  
  We use conventional commits:
  - `feat:` New feature
  - `fix:` Bug fix
  - `docs:` Documentation changes
  - `test:` Adding tests
  - `refactor:` Code refactoring
  - `chore:` Maintenance tasks
  
  ## Code Style
  
  - Go: `gofmt`, `golangci-lint`
  - Python: `black`, `flake8`
  - TypeScript: `prettier`, `eslint`
  ```
- [ ] Create `SECURITY.md`
  ```markdown
  # Security Policy
  
  ## Reporting a Vulnerability
  
  Please report security vulnerabilities to: security@humanoidops.com
  
  We will respond within 48 hours.
  
  ## Supported Versions
  
  | Version | Supported          |
  | ------- | ------------------ |
  | 0.1.x   | :white_check_mark: |
  ```

#### End of Day: Makefile & Scripts
- [ ] Create `Makefile`
  ```makefile
  .PHONY: help dev-up dev-down test-backend test-agent test-dashboard clean
  
  help: ## Show this help
  	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'
  
  dev-up: ## Start development environment
  	docker-compose up -d
  	@echo "✅ Services started:"
  	@echo "   Dashboard:        http://localhost:3000"
  	@echo "   API:              http://localhost:8080"
  	@echo "   EMQX Dashboard:   http://localhost:18083 (admin/public)"
  	@echo "   Prometheus:       http://localhost:9090"
  	@echo "   Grafana:          http://localhost:3001 (admin/admin)"
  
  dev-down: ## Stop development environment
  	docker-compose down
  
  dev-logs: ## Show logs
  	docker-compose logs -f
  
  test-backend: ## Run backend tests
  	cd backend && go test ./... -v -coverprofile=coverage.out
  
  test-agent: ## Run agent tests
  	cd agent && pytest --cov=humanoidops_agent
  
  test-dashboard: ## Run dashboard tests
  	cd dashboard && npm test
  
  test-all: test-backend test-agent test-dashboard ## Run all tests
  
  lint-backend: ## Lint backend code
  	cd backend && golangci-lint run
  
  lint-agent: ## Lint agent code
  	cd agent && flake8 . && black --check .
  
  lint-dashboard: ## Lint dashboard code
  	cd dashboard && npm run lint
  
  format-backend: ## Format backend code
  	cd backend && go fmt ./...
  
  format-agent: ## Format agent code
  	cd agent && black .
  
  format-dashboard: ## Format dashboard code
  	cd dashboard && npm run format
  
  clean: ## Clean up
  	docker-compose down -v
  	rm -rf backend/coverage.out agent/.coverage dashboard/.next
  ```
- [ ] Create `scripts/setup-dev.sh`
  ```bash
  #!/bin/bash
  set -e
  
  echo "🚀 Setting up HumanoidOps development environment..."
  
  # Check prerequisites
  command -v docker >/dev/null 2>&1 || { echo "❌ Docker not found. Install: https://docs.docker.com/get-docker/"; exit 1; }
  command -v docker-compose >/dev/null 2>&1 || { echo "❌ Docker Compose not found."; exit 1; }
  command -v go >/dev/null 2>&1 || { echo "❌ Go not found. Install: https://go.dev/doc/install"; exit 1; }
  command -v python3 >/dev/null 2>&1 || { echo "❌ Python 3 not found."; exit 1; }
  command -v node >/dev/null 2>&1 || { echo "❌ Node.js not found. Install: https://nodejs.org/"; exit 1; }
  
  echo "✅ Prerequisites check passed"
  
  # Copy .env.example if .env doesn't exist
  if [ ! -f .env ]; then
    cp .env.example .env
    echo "✅ Created .env file"
  fi
  
  # Install backend dependencies
  echo "📦 Installing backend dependencies..."
  cd backend && go mod download && cd ..
  
  # Install agent dependencies
  echo "📦 Installing agent dependencies..."
  cd agent
  python3 -m venv .venv
  source .venv/bin/activate
  pip install -e ".[dev]"
  cd ..
  
  # Install dashboard dependencies
  echo "📦 Installing dashboard dependencies..."
  cd dashboard && npm install && cd ..
  
  # Start Docker services
  echo "🐳 Starting Docker services..."
  docker-compose up -d
  
  echo "✅ Setup complete!"
  echo ""
  echo "Next steps:"
  echo "  1. Run backend:    cd backend && go run cmd/api/main.go"
  echo "  2. Run simulator:  cd agent && source .venv/bin/activate && python -m humanoidops_agent.cli"
  echo "  3. Run dashboard:  cd dashboard && npm run dev"
  ```
- [ ] Make script executable
  ```bash
  chmod +x scripts/setup-dev.sh
  ```
- [ ] Commit documentation and scripts
  ```bash
  git add .
  git commit -m "docs: add comprehensive documentation and dev scripts"
  ```

---

### Day 5: Friday - Testing & Review

#### Morning: Test Infrastructure
- [ ] Test full setup from scratch
  ```bash
  # In fresh clone
  ./scripts/setup-dev.sh
  make dev-up
  ```
- [ ] Verify all services start correctly
  - [ ] PostgreSQL accessible on 5432
  - [ ] EMQX on 1883 and dashboard on 18083
  - [ ] Prometheus on 9090
  - [ ] Grafana on 3001
- [ ] Create database migration tool setup
  ```bash
  cd backend
  go get -u github.com/golang-migrate/migrate/v4/cmd/migrate
  ```
- [ ] Create first migration skeleton
  ```bash
  mkdir -p backend/migrations
  touch backend/migrations/001_initial_schema.up.sql
  touch backend/migrations/001_initial_schema.down.sql
  ```

#### Afternoon: Issue Templates
- [ ] Create `.github/ISSUE_TEMPLATE/bug_report.md`
- [ ] Create `.github/ISSUE_TEMPLATE/feature_request.md`
- [ ] Create `.github/PULL_REQUEST_TEMPLATE.md`
- [ ] 🟡 Set up GitHub Projects board
  - Columns: Backlog, In Progress, Review, Done
  - Add Week 2-8 tasks as issues

#### End of Week: Review & Planning
- [ ] ✅ Week 1 deliverables complete:
  - [ ] Repository structure created
  - [ ] Docker Compose working
  - [ ] CI/CD pipelines configured
  - [ ] Documentation skeleton created
  - [ ] Development environment tested
- [ ] 🔴 Create PR: `develop` → `main` for Week 1
- [ ] Merge and tag release: `v0.1.0-week1`
- [ ] Plan Week 2 sprint

---

## Week 2: Robot Simulator Development

**Goal**: Create functional robot simulator that generates realistic telemetry

### Day 1: Monday - Simulator Core

#### Morning: Data Models
- [ ] 🔴 Create `agent/humanoidops_agent/models.py`
  ```python
  """Pydantic models for telemetry and configuration"""
  
  from pydantic import BaseModel, Field
  from typing import Optional, List, Dict, Any
  from datetime import datetime
  from uuid import UUID
  
  class Location(BaseModel):
      x: float = 0.0
      y: float = 0.0
      z: float = 0.0
  
  class Metrics(BaseModel):
      cpu_temp: float = Field(..., ge=0, le=150)
      cpu_usage: float = Field(..., ge=0, le=100)
      memory_usage: float = Field(..., ge=0, le=100)
      joint_health: List[float] = Field(default_factory=list)
      error_codes: List[str] = Field(default_factory=list)
      network_latency: Optional[int] = None
      custom: Optional[Dict[str, Any]] = None
  
  class TelemetryMessage(BaseModel):
      robot_id: str
      timestamp: str
      battery_level: float = Field(..., ge=0, le=100)
      status: str = Field(..., pattern="^(idle|working|error|offline|charging)$")
      location: Optional[Location] = None
      metrics: Optional[Metrics] = None
      
      class Config:
          json_schema_extra = {
              "example": {
                  "robot_id": "550e8400-e29b-41d4-a716-446655440000",
                  "timestamp": "2024-01-15T10:30:00Z",
                  "battery_level": 85.5,
                  "status": "working",
                  "location": {"x": 12.5, "y": 8.3, "z": 0.0},
                  "metrics": {
                      "cpu_temp": 65.2,
                      "cpu_usage": 45.8,
                      "memory_usage": 60.3,
                      "joint_health": [98, 97, 99, 100],
                      "error_codes": []
                  }
              }
          }
  ```

#### Afternoon: Simulator Logic
- [ ] 🔴 Create `agent/humanoidops_agent/simulator.py`
  - See PLAN.md → Component Design → Robot Agent for full code
  - Implement `RobotSimulator` class
  - Implement state machine (idle → working → charging)
  - Implement battery drain/charge logic
  - Implement random walk for location
  - Implement joint degradation simulation
- [ ] Test simulator locally
  ```python
  # Test script
  from humanoidops_agent.simulator import RobotSimulator
  
  robot = RobotSimulator(model="TestBot")
  for _ in range(10):
      robot.update(delta_time=1.0)
      telemetry = robot.generate_telemetry()
      print(telemetry)
  ```

#### End of Day: Unit Tests
- [ ] Create `agent/tests/test_simulator.py`
  ```python
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
  
  def test_telemetry_generation():
      robot = RobotSimulator()
      telemetry = robot.generate_telemetry()
      assert "robot_id" in telemetry
      assert "timestamp" in telemetry
      assert 0 <= telemetry["battery_level"] <= 100
  
  def test_status_transitions():
      robot = RobotSimulator()
      robot.state.battery_level = 10.0
      robot.update(delta_time=1.0)
      assert robot.state.status == "charging"
  ```
- [ ] Run tests
  ```bash
  cd agent
  pytest -v
  ```

---

### Day 2: Tuesday - MQTT Client

#### Morning: MQTT Transport
- [ ] 🔴 Create `agent/humanoidops_agent/mqtt_transport.py`
  ```python
  """MQTT client for telemetry publishing"""
  
  import paho.mqtt.client as mqtt
  import json
  import logging
  from typing import Optional, Callable
  
  logger = logging.getLogger(__name__)
  
  class MQTTTransport:
      def __init__(
          self,
          broker_url: str,
          port: int = 1883,
          username: Optional[str] = None,
          password: Optional[str] = None,
          client_id: Optional[str] = None,
          use_tls: bool = False
      ):
          self.broker_url = broker_url
          self.port = port
          self.use_tls = use_tls
          
          self.client = mqtt.Client(
              client_id=client_id or "",
              protocol=mqtt.MQTTv5
          )
          
          if username and password:
              self.client.username_pw_set(username, password)
          
          if use_tls:
              self.client.tls_set()
          
          self.client.on_connect = self._on_connect
          self.client.on_disconnect = self._on_disconnect
          self.client.on_publish = self._on_publish
          
          self.connected = False
      
      def _on_connect(self, client, userdata, flags, rc, properties=None):
          if rc == 0:
              logger.info(f"Connected to MQTT broker at {self.broker_url}:{self.port}")
              self.connected = True
          else:
              logger.error(f"Failed to connect: {rc}")
      
      def _on_disconnect(self, client, userdata, rc):
          logger.warning(f"Disconnected from MQTT broker: {rc}")
          self.connected = False
      
      def _on_publish(self, client, userdata, mid):
          logger.debug(f"Message {mid} published")
      
      def connect(self):
          """Connect to MQTT broker"""
          self.client.connect(self.broker_url, self.port, keepalive=60)
          self.client.loop_start()
      
      def disconnect(self):
          """Disconnect from broker"""
          self.client.loop_stop()
          self.client.disconnect()
      
      def publish_telemetry(self, robot_id: str, telemetry: dict) -> bool:
          """Publish telemetry to MQTT topic"""
          topic = f"robots/{robot_id}/telemetry"
          payload = json.dumps(telemetry)
          
          result = self.client.publish(topic, payload, qos=1)
          
          if result.rc != mqtt.MQTT_ERR_SUCCESS:
              logger.error(f"Failed to publish: {result.rc}")
              return False
          
          return True
  ```

#### Afternoon: HTTP Fallback
- [ ] Create `agent/humanoidops_agent/http_transport.py`
  ```python
  """HTTP transport for telemetry (fallback)"""
  
  import requests
  import logging
  from typing import Dict, Any
  
  logger = logging.getLogger(__name__)
  
  class HTTPTransport:
      def __init__(
          self,
          api_url: str,
          api_key: str,
          timeout: int = 10
      ):
          self.api_url = api_url.rstrip('/')
          self.api_key = api_key
          self.timeout = timeout
          self.session = requests.Session()
          self.session.headers.update({
              'Authorization': f'Bearer {api_key}',
              'Content-Type': 'application/json'
          })
      
      def publish_telemetry(self, robot_id: str, telemetry: Dict[str, Any]) -> bool:
          """Send telemetry via HTTP POST"""
          url = f"{self.api_url}/telemetry"
          
          try:
              response = self.session.post(
                  url,
                  json=telemetry,
                  timeout=self.timeout
              )
              
              if response.status_code == 201:
                  logger.debug(f"Telemetry sent successfully for {robot_id}")
                  return True
              else:
                  logger.error(f"HTTP {response.status_code}: {response.text}")
                  return False
          
          except requests.exceptions.RequestException as e:
              logger.error(f"HTTP request failed: {e}")
              return False
  ```

#### End of Day: Offline Queue
- [ ] Create `agent/humanoidops_agent/queue.py`
  ```python
  """Offline message queue for telemetry"""
  
  import json
  from collections import deque
  from typing import Dict, Any, Optional
  import logging
  
  logger = logging.getLogger(__name__)
  
  class TelemetryQueue:
      def __init__(self, max_size: int = 1000):
          self.queue = deque(maxlen=max_size)
          self.max_size = max_size
      
      def enqueue(self, telemetry: Dict[str, Any]):
          """Add telemetry to queue"""
          self.queue.append(telemetry)
          if len(self.queue) == self.max_size:
              logger.warning("Queue is full, dropping oldest messages")
      
      def dequeue(self) -> Optional[Dict[str, Any]]:
          """Remove and return oldest telemetry"""
          try:
              return self.queue.popleft()
          except IndexError:
              return None
      
      def peek(self) -> Optional[Dict[str, Any]]:
          """View oldest telemetry without removing"""
          return self.queue[0] if self.queue else None
      
      def size(self) -> int:
          """Get queue size"""
          return len(self.queue)
      
      def is_empty(self) -> bool:
          """Check if queue is empty"""
          return len(self.queue) == 0
      
      def clear(self):
          """Clear all messages"""
          self.queue.clear()
  ```
- [ ] Commit day's work
  ```bash
  git add .
  git commit -m "feat(agent): add MQTT, HTTP transports and offline queue"
  ```

---

### Day 3: Wednesday - Main Client & CLI

#### Morning: Main Client
- [ ] 🔴 Create `agent/humanoidops_agent/client.py`
  ```python
  """Main HumanoidOps client for robot integration"""
  
  import logging
  import time
  from typing import Optional, Dict, Any
  from .mqtt_transport import MQTTTransport
  from .http_transport import HTTPTransport
  from .queue import TelemetryQueue
  from .models import TelemetryMessage
  
  logger = logging.getLogger(__name__)
  
  class HumanoidOpsClient:
      def __init__(
          self,
          robot_id: str,
          api_key: str,
          mqtt_broker: Optional[str] = None,
          api_url: Optional[str] = None,
          use_mqtt: bool = True,
          max_queue_size: int = 1000
      ):
          self.robot_id = robot_id
          self.api_key = api_key
          self.use_mqtt = use_mqtt
          
          # Initialize transports
          self.mqtt = None
          self.http = None
          
          if use_mqtt and mqtt_broker:
              self.mqtt = MQTTTransport(
                  broker_url=mqtt_broker,
                  client_id=f"robot-{robot_id}"
              )
          
          if api_url:
              self.http = HTTPTransport(api_url, api_key)
          
          # Offline queue
          self.queue = TelemetryQueue(max_queue_size)
      
      def connect(self):
          """Connect to platform"""
          if self.mqtt:
              self.mqtt.connect()
              # Wait for connection
              timeout = 10
              while not self.mqtt.connected and timeout > 0:
                  time.sleep(0.5)
                  timeout -= 0.5
              
              if not self.mqtt.connected:
                  logger.warning("MQTT connection failed, will use HTTP fallback")
      
      def disconnect(self):
          """Disconnect from platform"""
          if self.mqtt:
              self.mqtt.disconnect()
      
      def send_telemetry(
          self,
          battery_level: float,
          status: str,
          location: Optional[Dict[str, float]] = None,
          metrics: Optional[Dict[str, Any]] = None
      ) -> bool:
          """Send telemetry data"""
          from datetime import datetime
          
          # Build telemetry message
          telemetry = {
              "robot_id": self.robot_id,
              "timestamp": datetime.utcnow().isoformat() + "Z",
              "battery_level": battery_level,
              "status": status,
          }
          
          if location:
              telemetry["location"] = location
          
          if metrics:
              telemetry["metrics"] = metrics
          
          # Validate with Pydantic
          try:
              TelemetryMessage(**telemetry)
          except Exception as e:
              logger.error(f"Invalid telemetry: {e}")
              return False
          
          # Try MQTT first, then HTTP
          success = False
          
          if self.mqtt and self.mqtt.connected:
              success = self.mqtt.publish_telemetry(self.robot_id, telemetry)
          
          if not success and self.http:
              success = self.http.publish_telemetry(self.robot_id, telemetry)
          
          if not success:
              # Queue for later
              self.queue.enqueue(telemetry)
              logger.warning("Telemetry queued for retry")
          
          return success
      
      def flush_queue(self):
          """Flush queued messages"""
          while not self.queue.is_empty():
              telemetry = self.queue.dequeue()
              if telemetry:
                  # Try to send
                  success = False
                  if self.mqtt and self.mqtt.connected:
                      success = self.mqtt.publish_telemetry(
                          telemetry["robot_id"],
                          telemetry
                      )
                  
                  if not success and self.http:
                      success = self.http.publish_telemetry(
                          telemetry["robot_id"],
                          telemetry
                      )
                  
                  if not success:
                      # Re-queue and stop
                      self.queue.enqueue(telemetry)
                      break
  ```

#### Afternoon: CLI Tool
- [ ] 🔴 Create `agent/humanoidops_agent/cli.py`
  ```python
  """Command-line interface for robot simulator"""
  
  import argparse
  import logging
  import time
  import yaml
  from pathlib import Path
  from .simulator import RobotSimulator
  from .client import HumanoidOpsClient
  
  logging.basicConfig(
      level=logging.INFO,
      format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
  )
  logger = logging.getLogger(__name__)
  
  def load_config(config_file: Path) -> dict:
      """Load configuration from YAML file"""
      with open(config_file) as f:
          return yaml.safe_load(f)
  
  def run_simulator(args):
      """Run robot simulator"""
      # Load config if provided
      config = {}
      if args.config:
          config = load_config(Path(args.config))
      
      # Create client
      client = HumanoidOpsClient(
          robot_id="simulator",  # Will be overridden per robot
          api_key=args.api_key or config.get('api_key', ''),
          mqtt_broker=args.broker or config.get('mqtt', {}).get('broker', 'localhost'),
          api_url=args.api_url or config.get('api_url'),
          use_mqtt=not args.http_only
      )
      
      # Connect (for MQTT)
      if not args.http_only:
          client.connect()
      
      # Create robots
      robots = []
      num_robots = args.num_robots or config.get('simulator', {}).get('num_robots', 10)
      
      for i in range(num_robots):
          robot = RobotSimulator(model=f"HumanoidV1-{i}")
          robots.append(robot)
          logger.info(f"Created robot {robot.robot_id}")
      
      logger.info(f"Starting simulation with {num_robots} robots")
      
      # Main loop
      interval = args.interval or config.get('simulator', {}).get('update_interval', 1.0)
      
      try:
          while True:
              for robot in robots:
                  # Update robot state
                  robot.update(delta_time=interval)
                  
                  # Generate and send telemetry
                  telemetry = robot.generate_telemetry()
                  
                  # Use robot-specific client
                  robot_client = HumanoidOpsClient(
                      robot_id=robot.robot_id,
                      api_key=args.api_key or config.get('api_key', ''),
                      mqtt_broker=args.broker,
                      api_url=args.api_url
                  )
                  
                  robot_client.send_telemetry(
                      battery_level=telemetry['battery_level'],
                      status=telemetry['status'],
                      location=telemetry.get('location'),
                      metrics=telemetry.get('metrics')
                  )
              
              time.sleep(interval)
      
      except KeyboardInterrupt:
          logger.info("Stopping simulation...")
      
      finally:
          client.disconnect()
  
  def main():
      """Main CLI entry point"""
      parser = argparse.ArgumentParser(
          description="HumanoidOps Robot Simulator"
      )
      
      parser.add_argument(
          '--broker',
          default='localhost',
          help='MQTT broker address (default: localhost)'
      )
      parser.add_argument(
          '--port',
          type=int,
          default=1883,
          help='MQTT broker port (default: 1883)'
      )
      parser.add_argument(
          '--api-url',
          help='API URL for HTTP fallback'
      )
      parser.add_argument(
          '--api-key',
          help='API key for authentication'
      )
      parser.add_argument(
          '--num-robots',
          type=int,
          default=10,
          help='Number of robots to simulate (default: 10)'
      )
      parser.add_argument(
          '--interval',
          type=float,
          default=1.0,
          help='Update interval in seconds (default: 1.0)'
      )
      parser.add_argument(
          '--config',
          help='Path to YAML config file'
      )
      parser.add_argument(
          '--http-only',
          action='store_true',
          help='Use HTTP transport only (no MQTT)'
      )
      parser.add_argument(
          '--verbose',
          action='store_true',
          help='Enable verbose logging'
      )
      
      args = parser.parse_args()
      
      if args.verbose:
          logging.getLogger().setLevel(logging.DEBUG)
      
      run_simulator(args)
  
  if __name__ == '__main__':
      main()
  ```

#### End of Day: Config File
- [ ] Create `agent/config.example.yaml`
  ```yaml
  # HumanoidOps Agent Configuration
  
  # API Configuration
  api_url: http://localhost:8080/api/v1
  api_key: your-api-key-here
  
  # MQTT Configuration
  mqtt:
    broker: localhost
    port: 1883
    use_tls: false
    username: ""
    password: ""
  
  # Simulator Configuration
  simulator:
    num_robots: 10
    update_interval: 1.0
    
  # Robot Templates
  robots:
    - model: HumanoidV1
      initial_battery: 100.0
    - model: HumanoidV2
      initial_battery: 85.0
  ```
- [ ] Test CLI
  ```bash
  cd agent
  python -m humanoidops_agent.cli --help
  python -m humanoidops_agent.cli --num-robots 5 --interval 2
  ```

---

### Day 4: Thursday - Integration Testing

#### Morning: Example Scripts
- [ ] Create `agent/examples/basic_integration.py`
  ```python
  """Basic integration example"""
  
  from humanoidops_agent import HumanoidOpsClient
  import time
  
  # Initialize client
  client = HumanoidOpsClient(
      robot_id="example-robot-001",
      api_key="your-api-key",
      mqtt_broker="localhost",
      api_url="http://localhost:8080/api/v1"
  )
  
  # Connect
  client.connect()
  
  # Send telemetry in loop
  try:
      battery = 100.0
      while True:
          client.send_telemetry(
              battery_level=battery,
              status="working",
              location={"x": 10.0, "y": 5.0, "z": 0.0},
              metrics={
                  "cpu_temp": 65.0,
                  "cpu_usage": 45.0,
                  "memory_usage": 60.0
              }
          )
          
          battery = max(0, battery - 0.5)  # Drain battery
          time.sleep(1)
  
  except KeyboardInterrupt:
      print("Stopping...")
  
  finally:
      client.disconnect()
  ```
- [ ] Create `agent/examples/simulator_fleet.py`
  ```python
  """Fleet simulation example"""
  
  from humanoidops_agent import RobotSimulator, HumanoidOpsClient
  import time
  
  # Create 5 robots
  robots = [RobotSimulator(model=f"Robot-{i}") for i in range(5)]
  
  # Create clients
  clients = [
      HumanoidOpsClient(
          robot_id=robot.robot_id,
          api_key="test-api-key",
          mqtt_broker="localhost"
      )
      for robot in robots
  ]
  
  # Connect all
  for client in clients:
      client.connect()
  
  # Simulation loop
  try:
      while True:
          for robot, client in zip(robots, clients):
              robot.update(delta_time=1.0)
              telemetry = robot.generate_telemetry()
              
              client.send_telemetry(
                  battery_level=telemetry['battery_level'],
                  status=telemetry['status'],
                  location=telemetry['location'],
                  metrics=telemetry['metrics']
              )
          
          time.sleep(1)
  
  except KeyboardInterrupt:
      print("Stopping...")
  
  finally:
      for client in clients:
          client.disconnect()
  ```

#### Afternoon: Tests
- [ ] Create `agent/tests/test_client.py`
- [ ] Create `agent/tests/test_mqtt.py`
- [ ] Create `agent/tests/test_queue.py`
- [ ] Run full test suite
  ```bash
  cd agent
  pytest -v --cov=humanoidops_agent --cov-report=term-missing
  ```
- [ ] Ensure >80% coverage

#### End of Day: Integration Test
- [ ] 🔴 End-to-end test: Simulator → MQTT → Verify message
  ```bash
  # Terminal 1: Start EMQX
  docker-compose up -d emqx
  
  # Terminal 2: Subscribe to MQTT topic
  mosquitto_sub -h localhost -t "robots/+/telemetry" -v
  
  # Terminal 3: Run simulator
  cd agent
  python -m humanoidops_agent.cli --num-robots 3 --interval 2
  
  # Verify messages appear in Terminal 2
  ```
- [ ] Commit agent code
  ```bash
  git add .
  git commit -m "feat(agent): complete robot simulator with MQTT/HTTP transport"
  ```

---

### Day 5: Friday - Documentation & Week Review

#### Morning: Agent Documentation
- [ ] Update `agent/README.md`
  ```markdown
  # HumanoidOps Agent
  
  Python SDK for integrating robots with HumanoidOps platform.
  
  ## Installation
  
  ```bash
  pip install humanoidops-agent
  ```
  
  ## Quick Start
  
  ### Basic Integration
  
  ```python
  from humanoidops_agent import HumanoidOpsClient
  
  client = HumanoidOpsClient(
      robot_id="your-robot-id",
      api_key="your-api-key",
      mqtt_broker="mqtt://broker.example.com"
  )
  
  client.connect()
  
  client.send_telemetry(
      battery_level=85.5,
      status="working",
      metrics={"cpu_temp": 65.0}
  )
  ```
  
  ### Run Simulator
  
  ```bash
  humanoidops-simulator --num-robots 10 --broker localhost
  ```
  
  ## API Reference
  
  See [API.md](docs/API.md)
  ```
- [ ] Create API documentation
- [ ] Add docstrings to all public methods

#### Afternoon: Week 2 Review
- [ ] ✅ Week 2 deliverables complete:
  - [ ] Functional robot simulator
  - [ ] MQTT client library
  - [ ] HTTP fallback transport
  - [ ] Offline queue
  - [ ] CLI tool
  - [ ] Unit tests (>80% coverage)
  - [ ] Example scripts
  - [ ] Documentation
- [ ] Test with 50 concurrent robots
  ```bash
  python -m humanoidops_agent.cli --num-robots 50
  ```
- [ ] Performance check: Monitor CPU/memory usage
- [ ] 🔴 Create PR: Week 2 changes
- [ ] Merge and tag: `v0.1.0-week2`

---

## Week 3: Telemetry Ingestion (Backend Part 1)

**Goal**: Database schema + MQTT handler receiving telemetry

### Day 1: Monday - Database Schema

#### Morning: Migration Framework
- [ ] 🔴 Install golang-migrate
  ```bash
  go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
  ```
- [ ] Create migration helper script
  ```bash
  cat > scripts/create-migration.sh << 'EOF'
  #!/bin/bash
  NAME=$1
  if [ -z "$NAME" ]; then
    echo "Usage: ./scripts/create-migration.sh <migration_name>"
    exit 1
  fi
  
  migrate create -ext sql -dir backend/migrations -seq $NAME
  EOF
  chmod +x scripts/create-migration.sh
  ```

#### Afternoon: Initial Schema
- [ ] 🔴 Create `backend/migrations/000001_initial_schema.up.sql`
  ```sql
  -- See PLAN.md → Data Architecture → Table Definitions
  -- Copy complete schema including:
  -- - users table
  -- - robots table
  -- - telemetry table (hypertable)
  -- - alerts table
  -- - audit_logs table
  ```
- [ ] Create `backend/migrations/000001_initial_schema.down.sql`
  ```sql
  DROP TABLE IF EXISTS audit_logs CASCADE;
  DROP TABLE IF EXISTS alerts CASCADE;
  DROP TABLE IF EXISTS telemetry CASCADE;
  DROP TABLE IF EXISTS robots CASCADE;
  DROP TABLE IF EXISTS users CASCADE;
  ```
- [ ] Run migration
  ```bash
  migrate -path backend/migrations -database "postgresql://humanoidops:development_password@localhost:5432/humanoidops?sslmode=disable" up
  ```
- [ ] Verify tables created
  ```bash
  docker-compose exec postgres psql -U humanoidops -c "\dt"
  ```

#### End of Day: Database Models (Go)
- [ ] 🔴 Create `backend/internal/models/user.go`
- [ ] Create `backend/internal/models/robot.go`
- [ ] Create `backend/internal/models/telemetry.go`
- [ ] Create `backend/internal/models/alert.go`
- [ ] Create `backend/internal/models/audit_log.go`
- [ ] See PLAN.md → Data Architecture for model definitions
- [ ] Commit database schema
  ```bash
  git add backend/migrations backend/internal/models
  git commit -m "feat(backend): add database schema and models"
  ```

---

### Day 2: Tuesday - Database Connection & Repository Pattern

#### Morning: Database Connection
- [ ] 🔴 Create `backend/internal/database/database.go`
  ```go
  package database
  
  import (
      "fmt"
      "log"
      "os"
      
      "gorm.io/driver/postgres"
      "gorm.io/gorm"
      "gorm.io/gorm/logger"
  )
  
  var DB *gorm.DB
  
  func Connect() error {
      dsn := os.Getenv("DATABASE_URL")
      if dsn == "" {
          return fmt.Errorf("DATABASE_URL not set")
      }
      
      var err error
      DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{
          Logger: logger.Default.LogMode(logger.Info),
      })
      
      if err != nil {
          return fmt.Errorf("failed to connect: %w", err)
      }
      
      log.Println("✅ Database connected")
      return nil
  }
  
  func Close() error {
      sqlDB, err := DB.DB()
      if err != nil {
          return err
      }
      return sqlDB.Close()
  }
  
  func GetDB() *gorm.DB {
      return DB
  }
  ```
- [ ] Test connection
  ```go
  // backend/cmd/test-db/main.go
  package main
  
  import (
      "log"
      "github.com/YOUR_USERNAME/humanoidops/internal/database"
      "github.com/joho/godotenv"
  )
  
  func main() {
      godotenv.Load()
      
      if err := database.Connect(); err != nil {
          log.Fatal(err)
      }
      defer database.Close()
      
      log.Println("✅ Connection test successful")
  }
  ```

#### Afternoon: Repository Pattern
- [ ] Create `backend/internal/repositories/robot_repository.go`
  ```go
  package repositories
  
  import (
      "github.com/YOUR_USERNAME/humanoidops/internal/models"
      "github.com/google/uuid"
      "gorm.io/gorm"
  )
  
  type RobotRepository struct {
      db *gorm.DB
  }
  
  func NewRobotRepository(db *gorm.DB) *RobotRepository {
      return &RobotRepository{db: db}
  }
  
  func (r *RobotRepository) Create(robot *models.Robot) error {
      return r.db.Create(robot).Error
  }
  
  func (r *RobotRepository) FindByID(id uuid.UUID) (*models.Robot, error) {
      var robot models.Robot
      err := r.db.First(&robot, id).Error
      return &robot, err
  }
  
  func (r *RobotRepository) FindAll(limit, offset int) ([]models.Robot, error) {
      var robots []models.Robot
      err := r.db.Limit(limit).Offset(offset).Find(&robots).Error
      return robots, err
  }
  
  func (r *RobotRepository) Update(robot *models.Robot) error {
      return r.db.Save(robot).Error
  }
  
  func (r *RobotRepository) Delete(id uuid.UUID) error {
      return r.db.Delete(&models.Robot{}, id).Error
  }
  ```
- [ ] Create `backend/internal/repositories/telemetry_repository.go`
- [ ] Write repository tests
- [ ] Commit repositories
  ```bash
  git add backend/internal/repositories
  git commit -m "feat(backend): add repository pattern for data access"
  ```

---

### Day 3: Wednesday - MQTT Handler Service

#### Morning: MQTT Client Setup
- [ ] 🔴 Create `backend/internal/mqtt/client.go`
  ```go
  package mqtt
  
  import (
      "fmt"
      "log"
      "os"
      "time"
      
      mqtt "github.com/eclipse/paho.mqtt.golang"
  )
  
  type Client struct {
      client mqtt.Client
      config ClientConfig
  }
  
  type ClientConfig struct {
      BrokerURL  string
      ClientID   string
      OnMessage  mqtt.MessageHandler
  }
  
  func NewClient(config ClientConfig) *Client {
      opts := mqtt.NewClientOptions()
      opts.AddBroker(config.BrokerURL)
      opts.SetClientID(config.ClientID)
      opts.SetDefaultPublishHandler(config.OnMessage)
      opts.SetOnConnectHandler(func(c mqtt.Client) {
          log.Println("✅ Connected to MQTT broker")
      })
      opts.SetConnectionLostHandler(func(c mqtt.Client, err error) {
          log.Printf("❌ Connection lost: %v", err)
      })
      
      client := mqtt.NewClient(opts)
      
      return &Client{
          client: client,
          config: config,
      }
  }
  
  func (c *Client) Connect() error {
      token := c.client.Connect()
      if token.Wait() && token.Error() != nil {
          return fmt.Errorf("failed to connect: %w", token.Error())
      }
      return nil
  }
  
  func (c *Client) Subscribe(topic string, qos byte) error {
      token := c.client.Subscribe(topic, qos, nil)
      if token.Wait() && token.Error() != nil {
          return fmt.Errorf("failed to subscribe: %w", token.Error())
      }
      log.Printf("✅ Subscribed to %s", topic)
      return nil
  }
  
  func (c *Client) Disconnect() {
      c.client.Disconnect(250)
  }
  ```

#### Afternoon: Message Handler
- [ ] 🔴 Create `backend/internal/mqtt/handler.go`
  ```go
  package mqtt
  
  import (
      "encoding/json"
      "log"
      "time"
      
      mqtt "github.com/eclipse/paho.mqtt.golang"
      "github.com/google/uuid"
      "github.com/YOUR_USERNAME/humanoidops/internal/database"
      "github.com/YOUR_USERNAME/humanoidops/internal/models"
  )
  
  type TelemetryMessage struct {
      RobotID      string                 `json:"robot_id"`
      Timestamp    string                 `json:"timestamp"`
      BatteryLevel float64                `json:"battery_level"`
      Status       string                 `json:"status"`
      Location     map[string]interface{} `json:"location"`
      Metrics      map[string]interface{} `json:"metrics"`
  }
  
  func HandleTelemetryMessage(client mqtt.Client, msg mqtt.Message) {
      var telemetryMsg TelemetryMessage
      
      if err := json.Unmarshal(msg.Payload(), &telemetryMsg); err != nil {
          log.Printf("❌ Invalid JSON: %v", err)
          return
      }
      
      // Parse robot ID
      robotID, err := uuid.Parse(telemetryMsg.RobotID)
      if err != nil {
          log.Printf("❌ Invalid robot ID: %v", err)
          return
      }
      
      // Parse timestamp
      timestamp, err := time.Parse(time.RFC3339, telemetryMsg.Timestamp)
      if err != nil {
          log.Printf("❌ Invalid timestamp: %v", err)
          timestamp = time.Now()
      }
      
      // Create telemetry record
      telemetry := models.Telemetry{
          RobotID:      robotID,
          Timestamp:    timestamp,
          BatteryLevel: telemetryMsg.BatteryLevel,
          Status:       telemetryMsg.Status,
          Location:     (*models.JSONB)(&telemetryMsg.Location),
          Metrics:      (*models.JSONB)(&telemetryMsg.Metrics),
      }
      
      // Save to database
      db := database.GetDB()
      if err := db.Create(&telemetry).Error; err != nil {
          log.Printf("❌ Failed to save telemetry: %v", err)
          return
      }
      
      // Update robot last_seen
      now := time.Now()
      db.Model(&models.Robot{}).
          Where("id = ?", robotID).
          Update("last_seen", now)
      
      log.Printf("✅ Saved telemetry for robot %s", robotID)
  }
  ```

#### End of Day: MQTT Handler Service
- [ ] 🔴 Create `backend/cmd/mqtt-handler/main.go`
  ```go
  package main
  
  import (
      "log"
      "os"
      "os/signal"
      "syscall"
      
      "github.com/YOUR_USERNAME/humanoidops/internal/database"
      "github.com/YOUR_USERNAME/humanoidops/internal/mqtt"
      "github.com/joho/godotenv"
  )
  
  func main() {
      // Load .env
      godotenv.Load()
      
      // Connect to database
      if err := database.Connect(); err != nil {
          log.Fatal("Failed to connect to database:", err)
      }
      defer database.Close()
      
      // MQTT configuration
      brokerURL := os.Getenv("MQTT_BROKER_URL")
      if brokerURL == "" {
          brokerURL = "tcp://localhost:1883"
      }
      
      // Create MQTT client
      mqttClient := mqtt.NewClient(mqtt.ClientConfig{
          BrokerURL: brokerURL,
          ClientID:  "humanoidops-mqtt-handler",
          OnMessage: mqtt.HandleTelemetryMessage,
      })
      
      // Connect
      if err := mqttClient.Connect(); err != nil {
          log.Fatal("Failed to connect to MQTT:", err)
      }
      
      // Subscribe to telemetry topic
      if err := mqttClient.Subscribe("robots/+/telemetry", 1); err != nil {
          log.Fatal("Failed to subscribe:", err)
      }
      
      log.Println("🚀 MQTT Handler started")
      
      // Wait for interrupt
      sigChan := make(chan os.Signal, 1)
      signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)
      <-sigChan
      
      log.Println("Shutting down...")
      mqttClient.Disconnect()
  }
  ```
- [ ] Test MQTT handler
  ```bash
  # Terminal 1: Start services
  docker-compose up -d postgres emqx
  
  # Terminal 2: Run MQTT handler
  cd backend
  go run cmd/mqtt-handler/main.go
  
  # Terminal 3: Run simulator
  cd agent
  python -m humanoidops_agent.cli --num-robots 5
  
  # Terminal 4: Verify data in DB
  docker-compose exec postgres psql -U humanoidops -c "SELECT COUNT(*) FROM telemetry;"
  ```

---

### Day 4: Thursday - Dockerfile & Docker Compose Integration

#### Morning: Dockerfiles
- [ ] Create `backend/docker/api.Dockerfile`
  ```dockerfile
  FROM golang:1.21-alpine AS builder
  
  WORKDIR /app
  COPY go.mod go.sum ./
  RUN go mod download
  
  COPY . .
  RUN CGO_ENABLED=0 GOOS=linux go build -o /api cmd/api/main.go
  
  FROM alpine:latest
  RUN apk --no-cache add ca-certificates
  WORKDIR /root/
  COPY --from=builder /api .
  
  EXPOSE 8080
  CMD ["./api"]
  ```
- [ ] Create `backend/docker/mqtt-handler.Dockerfile`
  ```dockerfile
  FROM golang:1.21-alpine AS builder
  
  WORKDIR /app
  COPY go.mod go.sum ./
  RUN go mod download
  
  COPY . .
  RUN CGO_ENABLED=0 GOOS=linux go build -o /mqtt-handler cmd/mqtt-handler/main.go
  
  FROM alpine:latest
  RUN apk --no-cache add ca-certificates
  WORKDIR /root/
  COPY --from=builder /mqtt-handler .
  
  CMD ["./mqtt-handler"]
  ```

#### Afternoon: Update Docker Compose
- [ ] Update `docker-compose.yml` to include backend services
  ```yaml
  # Add mqtt-handler service
  mqtt-handler:
    build:
      context: ./backend
      dockerfile: docker/mqtt-handler.Dockerfile
    environment:
      DATABASE_URL: postgresql://humanoidops:dev_password@postgres:5432/humanoidops
      MQTT_BROKER_URL: tcp://emqx:1883
    depends_on:
      - postgres
      - emqx
    restart: unless-stopped
  ```
- [ ] Test full stack
  ```bash
  docker-compose up --build
  ```

#### End of Day: Integration Test
- [ ] 🔴 End-to-end test: Simulator → MQTT → Handler → Database
  ```bash
  # Start all services
  docker-compose up -d
  
  # Run simulator from agent
  cd agent
  python -m humanoidops_agent.cli --num-robots 10 --interval 1
  
  # Query database after 30 seconds
  docker-compose exec postgres psql -U humanoidops -c "SELECT robot_id, COUNT(*) FROM telemetry GROUP BY robot_id;"
  
  # Should see ~30 records per robot (30 seconds × 1Hz)
  ```
- [ ] Commit MQTT handler
  ```bash
  git add .
  git commit -m "feat(backend): add MQTT handler for telemetry ingestion"
  ```

---

### Day 5: Friday - Week 3 Review & Cleanup

#### Morning: Performance Testing
- [ ] Load test with 50 robots
  ```bash
  python -m humanoidops_agent.cli --num-robots 50
  ```
- [ ] Monitor database performance
  ```sql
  -- Query performance
  EXPLAIN ANALYZE SELECT * FROM telemetry WHERE robot_id = '...' ORDER BY timestamp DESC LIMIT 100;
  
  -- Check hypertable chunks
  SELECT * FROM timescaledb_information.chunks;
  ```
- [ ] Check MQTT handler logs for errors

#### Afternoon: Documentation & Cleanup
- [ ] Document database schema in README
- [ ] Add comments to migration files
- [ ] Create database diagram (optional)
- [ ] Clean up unused code

#### End of Week: Review
- [ ] ✅ Week 3 deliverables complete:
  - [ ] Database schema with TimescaleDB
  - [ ] MQTT handler receiving telemetry
  - [ ] Telemetry stored in database
  - [ ] Integration tested (simulator → DB)
  - [ ] Docker services working
  - [ ] 🔴 Health check endpoint works
- [ ] Create PR: Week 3 changes
- [ ] Merge and tag: `v0.1.0-week3`

---

## Week 4: REST API (Backend Part 2)

**Goal**: Complete REST API with authentication and all robot management endpoints

### Day 1: Monday - API Structure & Configuration

#### Morning: Configuration Management
- [ ] 🔴 Create `backend/internal/config/config.go`
  ```go
  package config
  
  import (
      "fmt"
      "os"
      "strconv"
  )
  
  type Config struct {
      DatabaseURL  string
      MQTTBrokerURL string
      JWTSecret    string
      JWTExpiry    string
      APIPort      string
  }
  
  func Load() (*Config, error) {
      config := &Config{
          DatabaseURL:   getEnv("DATABASE_URL", ""),
          MQTTBrokerURL: getEnv("MQTT_BROKER_URL", "tcp://localhost:1883"),
          JWTSecret:     getEnv("JWT_SECRET", ""),
          JWTExpiry:     getEnv("JWT_EXPIRY", "24h"),
          APIPort:       getEnv("API_PORT", "8080"),
      }
      
      if config.DatabaseURL == "" {
          return nil, fmt.Errorf("DATABASE_URL is required")
      }
      
      if config.JWTSecret == "" || len(config.JWTSecret) < 32 {
          return nil, fmt.Errorf("JWT_SECRET must be at least 32 characters")
      }
      
      return config, nil
  }
  
  func getEnv(key, defaultValue string) string {
      if value := os.Getenv(key); value != "" {
          return value
      }
      return defaultValue
  }
  ```

#### Afternoon: Middleware Setup
- [ ] 🔴 Create `backend/internal/middleware/cors.go`
  ```go
  package middleware
  
  import "github.com/gin-gonic/gin"
  
  func CORS() gin.HandlerFunc {
      return func(c *gin.Context) {
          c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
          c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")
          c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
          c.Writer.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE, OPTIONS")
          
          if c.Request.Method == "OPTIONS" {
              c.AbortWithStatus(204)
              return
          }
          
          c.Next()
      }
  }
  ```
- [ ] Create `backend/internal/middleware/logger.go`
  ```go
  package middleware
  
  import (
      "log"
      "time"
      
      "github.com/gin-gonic/gin"
      "github.com/google/uuid"
  )
  
  func Logger() gin.HandlerFunc {
      return func(c *gin.Context) {
          start := time.Now()
          
          // Add request ID
          requestID := uuid.New().String()
          c.Set("request_id", requestID)
          c.Header("X-Request-ID", requestID)
          
          // Process request
          c.Next()
          
          // Log request
          latency := time.Since(start)
          log.Printf(
              "[%s] %s %s %d %v",
              requestID[:8],
              c.Request.Method,
              c.Request.URL.Path,
              c.Writer.Status(),
              latency,
          )
      }
  }
  ```
- [ ] Create `backend/internal/middleware/rate_limit.go` (optional for v0.1)

#### End of Day: API Server Skeleton
- [ ] 🔴 Update `backend/cmd/api/main.go`
  ```go
  package main
  
  import (
      "log"
      
      "github.com/gin-gonic/gin"
      "github.com/YOUR_USERNAME/humanoidops/internal/config"
      "github.com/YOUR_USERNAME/humanoidops/internal/database"
      "github.com/YOUR_USERNAME/humanoidops/internal/handlers"
      "github.com/YOUR_USERNAME/humanoidops/internal/middleware"
      "github.com/joho/godotenv"
  )
  
  func main() {
      // Load .env
      godotenv.Load()
      
      // Load configuration
      cfg, err := config.Load()
      if err != nil {
          log.Fatal("Failed to load config:", err)
      }
      
      // Connect to database
      if err := database.Connect(); err != nil {
          log.Fatal("Failed to connect to database:", err)
      }
      defer database.Close()
      
      // Setup Gin
      router := gin.Default()
      
      // Middleware
      router.Use(middleware.CORS())
      router.Use(middleware.Logger())
      
      // Health check (no auth)
      router.GET("/api/v1/health", handlers.HealthCheck)
      
      // Public routes
      public := router.Group("/api/v1")
      {
          public.POST("/auth/login", handlers.Login)
      }
      
      // Protected routes (require authentication)
      protected := router.Group("/api/v1")
      protected.Use(middleware.AuthRequired())
      {
          // Robots
          protected.POST("/robots", middleware.RequireRole("admin"), handlers.CreateRobot)
          protected.GET("/robots", handlers.ListRobots)
          protected.GET("/robots/:id", handlers.GetRobot)
          protected.PATCH("/robots/:id", middleware.RequireRole("admin"), handlers.UpdateRobot)
          protected.DELETE("/robots/:id", middleware.RequireRole("admin"), handlers.DeleteRobot)
          
          // TODO: Add other endpoints
      }
      
      // Start server
      log.Printf("🚀 API server starting on port %s", cfg.APIPort)
      if err := router.Run(":" + cfg.APIPort); err != nil {
          log.Fatal("Failed to start server:", err)
      }
  }
  ```

---

### Day 2: Tuesday - Authentication System

#### Morning: JWT Utilities
- [ ] 🔴 Create `backend/pkg/jwt/jwt.go`
  ```go
  package jwt
  
  import (
      "fmt"
      "os"
      "time"
      
      "github.com/golang-jwt/jwt/v5"
      "github.com/google/uuid"
  )
  
  type Claims struct {
      UserID   string `json:"user_id"`
      Username string `json:"username"`
      Role     string `json:"role"`
      jwt.RegisteredClaims
  }
  
  func GenerateToken(userID, username, role string, expiry time.Duration) (string, error) {
      claims := Claims{
          UserID:   userID,
          Username: username,
          Role:     role,
          RegisteredClaims: jwt.RegisteredClaims{
              ExpiresAt: jwt.NewNumericDate(time.Now().Add(expiry)),
              IssuedAt:  jwt.NewNumericDate(time.Now()),
          },
      }
      
      token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
      
      secret := []byte(os.Getenv("JWT_SECRET"))
      return token.SignedString(secret)
  }
  
  func ValidateToken(tokenString string) (*Claims, error) {
      token, err := jwt.ParseWithClaims(
          tokenString,
          &Claims{},
          func(token *jwt.Token) (interface{}, error) {
              if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                  return nil, fmt.Errorf("unexpected signing method")
              }
              return []byte(os.Getenv("JWT_SECRET")), nil
          },
      )
      
      if err != nil {
          return nil, err
      }
      
      if claims, ok := token.Claims.(*Claims); ok && token.Valid {
          return claims, nil
      }
      
      return nil, fmt.Errorf("invalid token")
  }
  ```

#### Afternoon: Authentication Handlers
- [ ] 🔴 Create `backend/internal/handlers/auth.go`
  ```go
  package handlers
  
  import (
      "net/http"
      "time"
      
      "github.com/gin-gonic/gin"
      "golang.org/x/crypto/bcrypt"
      
      "github.com/YOUR_USERNAME/humanoidops/internal/database"
      "github.com/YOUR_USERNAME/humanoidops/internal/models"
      "github.com/YOUR_USERNAME/humanoidops/pkg/jwt"
  )
  
  type LoginRequest struct {
      Username string `json:"username" binding:"required"`
      Password string `json:"password" binding:"required"`
  }
  
  type LoginResponse struct {
      Token        string      `json:"token"`
      RefreshToken string      `json:"refresh_token"`
      ExpiresAt    time.Time   `json:"expires_at"`
      User         models.User `json:"user"`
  }
  
  func Login(c *gin.Context) {
      var req LoginRequest
      if err := c.ShouldBindJSON(&req); err != nil {
          c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
          return
      }
      
      // Find user
      var user models.User
      if err := database.GetDB().Where("username = ?", req.Username).First(&user).Error; err != nil {
          c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
          return
      }
      
      // Check password
      if err := bcrypt.CompareHashAndPassword([]byte(user.PasswordHash), []byte(req.Password)); err != nil {
          c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
          return
      }
      
      // Generate JWT
      expiry := 24 * time.Hour
      token, err := jwt.GenerateToken(user.ID.String(), user.Username, user.Role, expiry)
      if err != nil {
          c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to generate token"})
          return
      }
      
      // TODO: Generate refresh token
      
      // Clear password hash from response
      user.PasswordHash = ""
      
      c.JSON(http.StatusOK, LoginResponse{
          Token:     token,
          ExpiresAt: time.Now().Add(expiry),
          User:      user,
      })
  }
  ```
- [ ] Create `backend/internal/middleware/auth.go`
  ```go
  package middleware
  
  import (
      "net/http"
      "strings"
      
      "github.com/gin-gonic/gin"
      "github.com/YOUR_USERNAME/humanoidops/pkg/jwt"
  )
  
  func AuthRequired() gin.HandlerFunc {
      return func(c *gin.Context) {
          authHeader := c.GetHeader("Authorization")
          if authHeader == "" {
              c.JSON(http.StatusUnauthorized, gin.H{"error": "Authorization header required"})
              c.Abort()
              return
          }
          
          parts := strings.SplitN(authHeader, " ", 2)
          if len(parts) != 2 || parts[0] != "Bearer" {
              c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid authorization header"})
              c.Abort()
              return
          }
          
          claims, err := jwt.ValidateToken(parts[1])
          if err != nil {
              c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
              c.Abort()
              return
          }
          
          c.Set("user_id", claims.UserID)
          c.Set("username", claims.Username)
          c.Set("role", claims.Role)
          
          c.Next()
      }
  }
  
  func RequireRole(roles ...string) gin.HandlerFunc {
      return func(c *gin.Context) {
          userRole := c.GetString("role")
          
          for _, role := range roles {
              if userRole == role {
                  c.Next()
                  return
              }
          }
          
          c.JSON(http.StatusForbidden, gin.H{"error": "Insufficient permissions"})
          c.Abort()
      }
  }
  ```

#### End of Day: Bootstrap Admin User
- [ ] Create `scripts/create-admin.sh`
  ```bash
  #!/bin/bash
  
  # Generate bcrypt hash for password
  # You'll need to implement this in Go or use online tool
  
  docker-compose exec postgres psql -U humanoidops -c "
  INSERT INTO users (username, email, password_hash, role)
  VALUES ('admin', 'admin@humanoidops.com', '\$2a\$12\$...', 'admin')
  ON CONFLICT (username) DO NOTHING;
  "
  
  echo "✅ Admin user created (username: admin, password: admin)"
  ```
- [ ] Test authentication
  ```bash
  # Login
  curl -X POST http://localhost:8080/api/v1/auth/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"admin"}'
  
  # Should receive JWT token
  ```

---

### Day 3: Wednesday - Robot Management Endpoints

#### All Day: Implement Handlers
- [ ] 🔴 Create `backend/internal/handlers/robots.go`
  - Implement all robot endpoints from PLAN.md → API Design
  - `CreateRobot`, `ListRobots`, `GetRobot`, `UpdateRobot`, `DeleteRobot`
- [ ] Create `backend/internal/services/robot_service.go` (business logic)
- [ ] See PLAN.md for complete implementation
- [ ] Test each endpoint with curl

**Testing Checklist**:
- [ ] POST /api/v1/robots (create robot)
- [ ] GET /api/v1/robots (list robots)
- [ ] GET /api/v1/robots/:id (get robot)
- [ ] PATCH /api/v1/robots/:id (update robot)
- [ ] DELETE /api/v1/robots/:id (soft delete)

---

### Day 4: Thursday - Telemetry & Additional Endpoints

#### Morning: Telemetry Endpoints
- [ ] Create `backend/internal/handlers/telemetry.go`
  - POST /api/v1/telemetry (HTTPS fallback)
  - GET /api/v1/robots/:id/telemetry (query telemetry)
- [ ] Implement time-range queries
- [ ] Test telemetry retrieval

#### Afternoon: Alerts & Audit Logs
- [ ] Create `backend/internal/handlers/alerts.go`
  - GET /api/v1/alerts
  - GET /api/v1/alerts/:id
  - POST /api/v1/alerts/:id/acknowledge
- [ ] Create `backend/internal/handlers/audit_logs.go`
  - GET /api/v1/audit-logs
- [ ] Implement audit logging service
- [ ] Test all endpoints

#### End of Day: Health & Metrics
- [ ] Implement health check with dependencies
  ```go
  func HealthCheck(c *gin.Context) {
      db := database.GetDB()
      sqlDB, _ := db.DB()
      
      health := gin.H{
          "status": "healthy",
          "timestamp": time.Now(),
          "version": "0.1.0",
          "checks": gin.H{
              "database": "unknown",
              "mqtt": "unknown",
          },
      }
      
      // Check database
      if err := sqlDB.Ping(); err != nil {
          health["checks"].(gin.H)["database"] = "unhealthy"
          health["status"] = "unhealthy"
      } else {
          health["checks"].(gin.H)["database"] = "healthy"
      }
      
      c.JSON(http.StatusOK, health)
  }
  ```
- [ ] Create `backend/internal/metrics/metrics.go` for Prometheus
- [ ] Expose /api/v1/metrics endpoint

---

### Day 5: Friday - Testing & Documentation

#### Morning: API Tests
- [ ] Write integration tests for auth
- [ ] Write integration tests for robot endpoints
- [ ] Write integration tests for telemetry
- [ ] Run full test suite
  ```bash
  cd backend
  go test ./... -v -coverprofile=coverage.out
  go tool cover -html=coverage.out
  ```
- [ ] Ensure >80% coverage

#### Afternoon: API Documentation
- [ ] Create OpenAPI specification
  - Manual: `docs/api/openapi.yaml`
  - Or use Swag library for auto-generation
- [ ] Set up Swagger UI at /api/docs
- [ ] Test all endpoints with Swagger UI

#### End of Week: Review
- [ ] ✅ Week 4 deliverables complete:
  - [ ] Authentication system (JWT)
  - [ ] RBAC middleware
  - [ ] All robot endpoints
  - [ ] Telemetry endpoints
  - [ ] Alert endpoints
  - [ ] Audit log endpoints
  - [ ] Health check
  - [ ] Prometheus metrics
  - [ ] OpenAPI spec
  - [ ] Integration tests (>80% coverage)
- [ ] 🔴 Full integration test: Register robot via API → Send telemetry → Query via API
- [ ] Create PR: Week 4 changes
- [ ] Merge and tag: `v0.1.0-week4`

---

## Week 5: Authentication & User Management

**Goal**: Build authentication UI and user management system

### Day 1: Monday - Next.js Setup & Authentication UI

#### Morning: Project Configuration
- [ ] 🔴 Configure Next.js environment variables
  ```bash
  cd dashboard
  cat > .env.local << 'EOF'
  NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1
  NEXT_PUBLIC_WS_URL=ws://localhost:8080/api/v1/ws
  EOF
  ```
- [ ] Configure TypeScript paths
  ```json
  // dashboard/tsconfig.json
  {
    "compilerOptions": {
      "paths": {
        "@/*": ["./*"],
        "@/components/*": ["./components/*"],
        "@/lib/*": ["./lib/*"],
        "@/types/*": ["./types/*"],
        "@/hooks/*": ["./hooks/*"]
      }
    }
  }
  ```
- [ ] Install shadcn/ui
  ```bash
  cd dashboard
  npx shadcn-ui@latest init
  
  # Select options:
  # - TypeScript: Yes
  # - Style: Default
  # - Base color: Slate
  # - CSS variables: Yes
  ```
- [ ] Install required shadcn components
  ```bash
  npx shadcn-ui@latest add button
  npx shadcn-ui@latest add card
  npx shadcn-ui@latest add input
  npx shadcn-ui@latest add label
  npx shadcn-ui@latest add toast
  npx shadcn-ui@latest add dropdown-menu
  npx shadcn-ui@latest add table
  npx shadcn-ui@latest add badge
  npx shadcn-ui@latest add tabs
  ```

#### Afternoon: Type Definitions
- [ ] 🔴 Create `dashboard/types/index.ts`
  ```typescript
  // User types
  export interface User {
    id: string;
    username: string;
    email: string;
    role: 'admin' | 'operator' | 'viewer';
    created_at: string;
    updated_at: string;
  }
  
  export interface LoginRequest {
    username: string;
    password: string;
  }
  
  export interface LoginResponse {
    token: string;
    refresh_token?: string;
    expires_at: string;
    user: User;
  }
  
  // Robot types
  export interface Robot {
    id: string;
    serial_number: string;
    model: string;
    manufacturer?: string;
    firmware_version?: string;
    location_name?: string;
    location_coordinates?: {
      lat: number;
      lon: number;
    };
    status: 'active' | 'inactive' | 'maintenance';
    registered_at: string;
    last_seen?: string;
    operational_status?: 'online' | 'offline' | 'error' | 'unknown';
  }
  
  // Telemetry types
  export interface Telemetry {
    robot_id: string;
    timestamp: string;
    battery_level: number;
    status: 'idle' | 'working' | 'error' | 'offline' | 'charging';
    location?: {
      x: number;
      y: number;
      z: number;
    };
    metrics?: {
      cpu_temp?: number;
      cpu_usage?: number;
      memory_usage?: number;
      joint_health?: number[];
      error_codes?: string[];
      network_latency?: number;
    };
  }
  
  // Alert types
  export interface Alert {
    id: string;
    robot_id: string;
    robot_serial?: string;
    type: 'robot_offline' | 'low_battery' | 'error_status' | 'high_temperature' | 'high_latency';
    severity: 'info' | 'warning' | 'critical';
    message: string;
    triggered_at: string;
    acknowledged: boolean;
    acknowledged_by?: string;
    acknowledged_at?: string;
  }
  
  // API response types
  export interface ApiResponse<T> {
    data: T;
    meta?: {
      timestamp: string;
      request_id: string;
    };
  }
  
  export interface PaginatedResponse<T> {
    data: T[];
    meta: {
      total: number;
      page: number;
      limit: number;
      total_pages: number;
    };
  }
  
  export interface ApiError {
    error: {
      code: string;
      message: string;
      details?: any;
    };
    meta?: {
      timestamp: string;
      request_id: string;
    };
  }
  ```

#### End of Day: API Client
- [ ] 🔴 Create `dashboard/lib/api.ts`
  ```typescript
  import axios, { AxiosError } from 'axios';
  import { ApiResponse, ApiError } from '@/types';
  
  const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8080/api/v1';
  
  // Create axios instance
  const apiClient = axios.create({
    baseURL: API_URL,
    headers: {
      'Content-Type': 'application/json',
    },
  });
  
  // Request interceptor (add auth token)
  apiClient.interceptors.request.use(
    (config) => {
      const token = localStorage.getItem('auth_token');
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    },
    (error) => Promise.reject(error)
  );
  
  // Response interceptor (handle errors)
  apiClient.interceptors.response.use(
    (response) => response,
    (error: AxiosError<ApiError>) => {
      if (error.response?.status === 401) {
        // Redirect to login
        localStorage.removeItem('auth_token');
        localStorage.removeItem('user');
        if (typeof window !== 'undefined') {
          window.location.href = '/login';
        }
      }
      return Promise.reject(error);
    }
  );
  
  export default apiClient;
  
  // Export typed API methods
  export const api = {
    // Auth
    login: async (username: string, password: string) => {
      const response = await apiClient.post('/auth/login', { username, password });
      return response.data;
    },
    
    logout: async () => {
      await apiClient.post('/auth/logout');
      localStorage.removeItem('auth_token');
      localStorage.removeItem('user');
    },
    
    // Robots
    getRobots: async (params?: { status?: string; model?: string; page?: number; limit?: number }) => {
      const response = await apiClient.get('/robots', { params });
      return response.data;
    },
    
    getRobot: async (id: string) => {
      const response = await apiClient.get(`/robots/${id}`);
      return response.data;
    },
    
    createRobot: async (data: any) => {
      const response = await apiClient.post('/robots', data);
      return response.data;
    },
    
    updateRobot: async (id: string, data: any) => {
      const response = await apiClient.patch(`/robots/${id}`, data);
      return response.data;
    },
    
    deleteRobot: async (id: string) => {
      await apiClient.delete(`/robots/${id}`);
    },
    
    // Telemetry
    getRobotTelemetry: async (robotId: string, params?: { start?: string; end?: string; limit?: number }) => {
      const response = await apiClient.get(`/robots/${robotId}/telemetry`, { params });
      return response.data;
    },
    
    // Alerts
    getAlerts: async (params?: { acknowledged?: boolean; severity?: string; robot_id?: string }) => {
      const response = await apiClient.get('/alerts', { params });
      return response.data;
    },
    
    acknowledgeAlert: async (id: string) => {
      const response = await apiClient.post(`/alerts/${id}/acknowledge`);
      return response.data;
    },
    
    // Audit logs
    getAuditLogs: async (params?: { start?: string; end?: string; user_id?: string; action?: string }) => {
      const response = await apiClient.get('/audit-logs', { params });
      return response.data;
    },
  };
  ```
- [ ] Commit day's work
  ```bash
  git add .
  git commit -m "feat(dashboard): setup Next.js with TypeScript and API client"
  ```

---

### Day 2: Tuesday - Login Page & Auth Context

#### Morning: Auth Context
- [ ] 🔴 Create `dashboard/lib/auth-context.tsx`
  ```typescript
  'use client';
  
  import React, { createContext, useContext, useState, useEffect } from 'react';
  import { User } from '@/types';
  import { api } from './api';
  
  interface AuthContextType {
    user: User | null;
    isAuthenticated: boolean;
    isLoading: boolean;
    login: (username: string, password: string) => Promise<void>;
    logout: () => Promise<void>;
  }
  
  const AuthContext = createContext<AuthContextType | undefined>(undefined);
  
  export function AuthProvider({ children }: { children: React.ReactNode }) {
    const [user, setUser] = useState<User | null>(null);
    const [isLoading, setIsLoading] = useState(true);
  
    useEffect(() => {
      // Check for stored user on mount
      const storedUser = localStorage.getItem('user');
      const token = localStorage.getItem('auth_token');
      
      if (storedUser && token) {
        setUser(JSON.parse(storedUser));
      }
      
      setIsLoading(false);
    }, []);
  
    const login = async (username: string, password: string) => {
      const response = await api.login(username, password);
      
      // Store token and user
      localStorage.setItem('auth_token', response.token);
      localStorage.setItem('user', JSON.stringify(response.user));
      
      setUser(response.user);
    };
  
    const logout = async () => {
      await api.logout();
      setUser(null);
    };
  
    return (
      <AuthContext.Provider
        value={{
          user,
          isAuthenticated: !!user,
          isLoading,
          login,
          logout,
        }}
      >
        {children}
      </AuthContext.Provider>
    );
  }
  
  export function useAuth() {
    const context = useContext(AuthContext);
    if (context === undefined) {
      throw new Error('useAuth must be used within an AuthProvider');
    }
    return context;
  }
  ```

#### Afternoon: Login Page
- [ ] 🔴 Create `dashboard/app/(auth)/login/page.tsx`
  ```typescript
  'use client';
  
  import { useState } from 'react';
  import { useRouter } from 'next/navigation';
  import { useAuth } from '@/lib/auth-context';
  import { Button } from '@/components/ui/button';
  import { Input } from '@/components/ui/input';
  import { Label } from '@/components/ui/label';
  import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
  import { useToast } from '@/components/ui/use-toast';
  
  export default function LoginPage() {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [isLoading, setIsLoading] = useState(false);
    
    const { login } = useAuth();
    const router = useRouter();
    const { toast } = useToast();
  
    const handleSubmit = async (e: React.FormEvent) => {
      e.preventDefault();
      setIsLoading(true);
  
      try {
        await login(username, password);
        toast({
          title: 'Login successful',
          description: 'Welcome to HumanoidOps',
        });
        router.push('/fleet');
      } catch (error: any) {
        toast({
          title: 'Login failed',
          description: error.response?.data?.error?.message || 'Invalid credentials',
          variant: 'destructive',
        });
      } finally {
        setIsLoading(false);
      }
    };
  
    return (
      <div className="flex min-h-screen items-center justify-center bg-gray-50 dark:bg-gray-900">
        <Card className="w-full max-w-md">
          <CardHeader className="space-y-1 text-center">
            <CardTitle className="text-3xl font-bold">HumanoidOps</CardTitle>
            <CardDescription>
              Enter your credentials to access the fleet management dashboard
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleSubmit} className="space-y-4">
              <div className="space-y-2">
                <Label htmlFor="username">Username</Label>
                <Input
                  id="username"
                  type="text"
                  placeholder="Enter username"
                  value={username}
                  onChange={(e) => setUsername(e.target.value)}
                  required
                  disabled={isLoading}
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="password">Password</Label>
                <Input
                  id="password"
                  type="password"
                  placeholder="Enter password"
                  value={password}
                  onChange={(e) => setPassword(e.target.value)}
                  required
                  disabled={isLoading}
                />
              </div>
              <Button type="submit" className="w-full" disabled={isLoading}>
                {isLoading ? 'Logging in...' : 'Login'}
              </Button>
            </form>
          </CardContent>
        </Card>
      </div>
    );
  }
  ```
- [ ] Create `dashboard/app/(auth)/layout.tsx`
  ```typescript
  export default function AuthLayout({
    children,
  }: {
    children: React.ReactNode;
  }) {
    return <>{children}</>;
  }
  ```

#### End of Day: Protected Routes
- [ ] Create `dashboard/components/protected-route.tsx`
  ```typescript
  'use client';
  
  import { useEffect } from 'react';
  import { useRouter } from 'next/navigation';
  import { useAuth } from '@/lib/auth-context';
  
  export function ProtectedRoute({ children }: { children: React.ReactNode }) {
    const { isAuthenticated, isLoading } = useAuth();
    const router = useRouter();
  
    useEffect(() => {
      if (!isLoading && !isAuthenticated) {
        router.push('/login');
      }
    }, [isAuthenticated, isLoading, router]);
  
    if (isLoading) {
      return (
        <div className="flex h-screen items-center justify-center">
          <div className="text-lg">Loading...</div>
        </div>
      );
    }
  
    if (!isAuthenticated) {
      return null;
    }
  
    return <>{children}</>;
  }
  ```
- [ ] Update `dashboard/app/layout.tsx`
  ```typescript
  import { Inter } from 'next/font/google';
  import './globals.css';
  import { AuthProvider } from '@/lib/auth-context';
  import { Toaster } from '@/components/ui/toaster';
  
  const inter = Inter({ subsets: ['latin'] });
  
  export const metadata = {
    title: 'HumanoidOps - Fleet Management',
    description: 'Open-source operations platform for humanoid robots',
  };
  
  export default function RootLayout({
    children,
  }: {
    children: React.ReactNode;
  }) {
    return (
      <html lang="en">
        <body className={inter.className}>
          <AuthProvider>
            {children}
            <Toaster />
          </AuthProvider>
        </body>
      </html>
    );
  }
  ```
- [ ] Test login flow
  ```bash
  cd dashboard
  npm run dev
  # Visit http://localhost:3000/login
  # Login with admin/admin
  ```

---

### Day 3: Wednesday - Dashboard Layout

#### Morning: Layout Components
- [ ] 🔴 Create `dashboard/components/layout/header.tsx`
  ```typescript
  'use client';
  
  import Link from 'next/link';
  import { useAuth } from '@/lib/auth-context';
  import { Button } from '@/components/ui/button';
  import {
    DropdownMenu,
    DropdownMenuContent,
    DropdownMenuItem,
    DropdownMenuLabel,
    DropdownMenuSeparator,
    DropdownMenuTrigger,
  } from '@/components/ui/dropdown-menu';
  import { User, LogOut, Settings } from 'lucide-react';
  
  export function Header() {
    const { user, logout } = useAuth();
  
    const handleLogout = async () => {
      await logout();
    };
  
    return (
      <header className="border-b">
        <div className="flex h-16 items-center px-6">
          <div className="flex items-center space-x-4">
            <Link href="/fleet" className="text-2xl font-bold">
              HumanoidOps
            </Link>
          </div>
          
          <div className="ml-auto flex items-center space-x-4">
            {/* User menu */}
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="ghost" size="sm">
                  <User className="mr-2 h-4 w-4" />
                  {user?.username}
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuLabel>
                  <div className="flex flex-col">
                    <span>{user?.username}</span>
                    <span className="text-xs text-muted-foreground">
                      Role: {user?.role}
                    </span>
                  </div>
                </DropdownMenuLabel>
                <DropdownMenuSeparator />
                <DropdownMenuItem>
                  <Settings className="mr-2 h-4 w-4" />
                  Settings
                </DropdownMenuItem>
                <DropdownMenuItem onClick={handleLogout}>
                  <LogOut className="mr-2 h-4 w-4" />
                  Logout
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          </div>
        </div>
      </header>
    );
  }
  ```
- [ ] Create `dashboard/components/layout/sidebar.tsx`
  ```typescript
  'use client';
  
  import Link from 'next/link';
  import { usePathname } from 'next/navigation';
  import { cn } from '@/lib/utils';
  import { Home, AlertCircle, FileText, Settings, Users } from 'lucide-react';
  import { useAuth } from '@/lib/auth-context';
  
  const navigation = [
    { name: 'Fleet', href: '/fleet', icon: Home },
    { name: 'Alerts', href: '/alerts', icon: AlertCircle },
    { name: 'Audit Logs', href: '/audit', icon: FileText },
  ];
  
  const adminNavigation = [
    { name: 'Settings', href: '/settings', icon: Settings },
    { name: 'Users', href: '/users', icon: Users },
  ];
  
  export function Sidebar() {
    const pathname = usePathname();
    const { user } = useAuth();
  
    return (
      <aside className="w-64 border-r bg-gray-50 dark:bg-gray-900">
        <nav className="space-y-1 px-3 py-4">
          {navigation.map((item) => {
            const isActive = pathname === item.href;
            return (
              <Link
                key={item.name}
                href={item.href}
                className={cn(
                  'flex items-center rounded-md px-3 py-2 text-sm font-medium',
                  isActive
                    ? 'bg-gray-200 text-gray-900 dark:bg-gray-800 dark:text-white'
                    : 'text-gray-700 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800'
                )}
              >
                <item.icon className="mr-3 h-5 w-5" />
                {item.name}
              </Link>
            );
          })}
          
          {user?.role === 'admin' && (
            <>
              <div className="my-4 border-t" />
              {adminNavigation.map((item) => {
                const isActive = pathname === item.href;
                return (
                  <Link
                    key={item.name}
                    href={item.href}
                    className={cn(
                      'flex items-center rounded-md px-3 py-2 text-sm font-medium',
                      isActive
                        ? 'bg-gray-200 text-gray-900 dark:bg-gray-800 dark:text-white'
                        : 'text-gray-700 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800'
                    )}
                  >
                    <item.icon className="mr-3 h-5 w-5" />
                    {item.name}
                  </Link>
                );
              })}
            </>
          )}
        </nav>
      </aside>
    );
  }
  ```

#### Afternoon: Dashboard Layout
- [ ] 🔴 Create `dashboard/app/(dashboard)/layout.tsx`
  ```typescript
  import { ProtectedRoute } from '@/components/protected-route';
  import { Header } from '@/components/layout/header';
  import { Sidebar } from '@/components/layout/sidebar';
  
  export default function DashboardLayout({
    children,
  }: {
    children: React.ReactNode;
  }) {
    return (
      <ProtectedRoute>
        <div className="flex h-screen flex-col">
          <Header />
          <div className="flex flex-1 overflow-hidden">
            <Sidebar />
            <main className="flex-1 overflow-y-auto p-6">{children}</main>
          </div>
        </div>
      </ProtectedRoute>
    );
  }
  ```
- [ ] Create placeholder pages
  ```bash
  mkdir -p dashboard/app/\(dashboard\)/{fleet,alerts,audit,settings,users}
  
  # Create page.tsx in each folder with placeholder content
  ```

#### End of Day: Test Layout
- [ ] Test navigation between pages
- [ ] Test responsive behavior (collapse sidebar on mobile)
- [ ] Test dark mode toggle (if implementing)
- [ ] Commit layout
  ```bash
  git add .
  git commit -m "feat(dashboard): add layout with header and sidebar"
  ```

---

### Day 4: Thursday - React Query Setup & Hooks

#### Morning: React Query Configuration
- [ ] Install React Query
  ```bash
  cd dashboard
  npm install @tanstack/react-query @tanstack/react-query-devtools
  ```
- [ ] 🔴 Create `dashboard/lib/query-client.tsx`
  ```typescript
  'use client';
  
  import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
  import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
  import { useState } from 'react';
  
  export function QueryProvider({ children }: { children: React.ReactNode }) {
    const [queryClient] = useState(
      () =>
        new QueryClient({
          defaultOptions: {
            queries: {
              staleTime: 60 * 1000, // 1 minute
              refetchOnWindowFocus: false,
            },
          },
        })
    );
  
    return (
      <QueryClientProvider client={queryClient}>
        {children}
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    );
  }
  ```
- [ ] Update `dashboard/app/layout.tsx` to include QueryProvider
  ```typescript
  import { QueryProvider } from '@/lib/query-client';
  
  // Wrap children with QueryProvider
  <QueryProvider>
    {children}
  </QueryProvider>
  ```

#### Afternoon: Custom Hooks
- [ ] 🔴 Create `dashboard/hooks/use-robots.ts`
  ```typescript
  import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
  import { api } from '@/lib/api';
  import { Robot, PaginatedResponse } from '@/types';
  
  export function useRobots(params?: { status?: string; model?: string; page?: number; limit?: number }) {
    return useQuery({
      queryKey: ['robots', params],
      queryFn: () => api.getRobots(params),
    });
  }
  
  export function useRobot(id: string) {
    return useQuery({
      queryKey: ['robot', id],
      queryFn: () => api.getRobot(id),
      enabled: !!id,
    });
  }
  
  export function useCreateRobot() {
    const queryClient = useQueryClient();
    
    return useMutation({
      mutationFn: api.createRobot,
      onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: ['robots'] });
      },
    });
  }
  
  export function useUpdateRobot() {
    const queryClient = useQueryClient();
    
    return useMutation({
      mutationFn: ({ id, data }: { id: string; data: any }) => api.updateRobot(id, data),
      onSuccess: (_, variables) => {
        queryClient.invalidateQueries({ queryKey: ['robots'] });
        queryClient.invalidateQueries({ queryKey: ['robot', variables.id] });
      },
    });
  }
  
  export function useDeleteRobot() {
    const queryClient = useQueryClient();
    
    return useMutation({
      mutationFn: api.deleteRobot,
      onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: ['robots'] });
      },
    });
  }
  ```
- [ ] Create `dashboard/hooks/use-telemetry.ts`
  ```typescript
  import { useQuery } from '@tanstack/react-query';
  import { api } from '@/lib/api';
  
  export function useRobotTelemetry(
    robotId: string,
    params?: { start?: string; end?: string; limit?: number }
  ) {
    return useQuery({
      queryKey: ['telemetry', robotId, params],
      queryFn: () => api.getRobotTelemetry(robotId, params),
      enabled: !!robotId,
      refetchInterval: 5000, // Refetch every 5 seconds
    });
  }
  ```
- [ ] Create `dashboard/hooks/use-alerts.ts`
  ```typescript
  import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
  import { api } from '@/lib/api';
  
  export function useAlerts(params?: { acknowledged?: boolean; severity?: string; robot_id?: string }) {
    return useQuery({
      queryKey: ['alerts', params],
      queryFn: () => api.getAlerts(params),
      refetchInterval: 10000, // Refetch every 10 seconds
    });
  }
  
  export function useAcknowledgeAlert() {
    const queryClient = useQueryClient();
    
    return useMutation({
      mutationFn: api.acknowledgeAlert,
      onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: ['alerts'] });
      },
    });
  }
  ```

#### End of Day: Utility Hooks
- [ ] Create `dashboard/hooks/use-audit-logs.ts`
- [ ] Create `dashboard/lib/utils.ts` (if not exists)
  ```typescript
  import { type ClassValue, clsx } from 'clsx';
  import { twMerge } from 'tailwind-merge';
  
  export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs));
  }
  
  export function formatRelativeTime(dateString: string): string {
    const date = new Date(dateString);
    const now = new Date();
    const seconds = Math.floor((now.getTime() - date.getTime()) / 1000);
  
    if (seconds < 60) return `${seconds}s ago`;
    if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
    if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
    return `${Math.floor(seconds / 86400)}d ago`;
  }
  
  export function formatBatteryLevel(level: number): string {
    return `${Math.round(level)}%`;
  }
  
  export function getStatusColor(status: string): string {
    switch (status) {
      case 'online':
      case 'working':
        return 'text-green-600';
      case 'offline':
      case 'error':
        return 'text-red-600';
      case 'charging':
      case 'idle':
        return 'text-yellow-600';
      default:
        return 'text-gray-600';
    }
  }
  ```
- [ ] Commit hooks
  ```bash
  git add .
  git commit -m "feat(dashboard): add React Query and custom hooks"
  ```

---

### Day 5: Friday - User Management UI (Admin Only)

#### Morning: User List Page
- [ ] 🔴 Create `dashboard/app/(dashboard)/users/page.tsx`
  ```typescript
  'use client';
  
  import { useState } from 'react';
  import { useAuth } from '@/lib/auth-context';
  import { Button } from '@/components/ui/button';
  import {
    Table,
    TableBody,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
  } from '@/components/ui/table';
  import { Badge } from '@/components/ui/badge';
  import { Plus } from 'lucide-react';
  
  export default function UsersPage() {
    const { user } = useAuth();
  
    // Redirect if not admin
    if (user?.role !== 'admin') {
      return (
        <div className="text-center">
          <p className="text-red-600">You don't have permission to access this page.</p>
        </div>
      );
    }
  
    return (
      <div className="space-y-6">
        <div className="flex items-center justify-between">
          <h1 className="text-3xl font-bold">User Management</h1>
          <Button>
            <Plus className="mr-2 h-4 w-4" />
            Add User
          </Button>
        </div>
  
        <div className="rounded-md border">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Username</TableHead>
                <TableHead>Email</TableHead>
                <TableHead>Role</TableHead>
                <TableHead>Created</TableHead>
                <TableHead>Actions</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {/* TODO: Fetch and display users */}
              <TableRow>
                <TableCell className="text-muted-foreground" colSpan={5}>
                  User management coming in v0.2
                </TableCell>
              </TableRow>
            </TableBody>
          </Table>
        </div>
      </div>
    );
  }
  ```

#### Afternoon: Settings Page
- [ ] Create `dashboard/app/(dashboard)/settings/page.tsx`
  ```typescript
  'use client';
  
  export default function SettingsPage() {
    return (
      <div className="space-y-6">
        <h1 className="text-3xl font-bold">Settings</h1>
        
        <div className="rounded-md border p-6">
          <h2 className="text-xl font-semibold mb-4">System Configuration</h2>
          <p className="text-muted-foreground">
            System settings will be available in v0.2
          </p>
        </div>
      </div>
    );
  }
  ```

#### End of Week: Week 5 Review
- [ ] ✅ Week 5 deliverables complete:
  - [ ] Next.js configured with TypeScript
  - [ ] Type definitions created
  - [ ] API client implemented
  - [ ] Authentication context
  - [ ] Login page working
  - [ ] Dashboard layout (header + sidebar)
  - [ ] Protected routes
  - [ ] React Query setup
  - [ ] Custom hooks for data fetching
  - [ ] User management placeholder (admin)
- [ ] Test authentication flow end-to-end
- [ ] Test protected route redirection
- [ ] Create PR: Week 5 changes
- [ ] Merge and tag: `v0.1.0-week5`

---

## Week 6: Dashboard Components & Real-time Updates

**Goal**: Build fleet overview, robot cards, and implement WebSocket for real-time updates

### Day 1: Monday - Fleet Overview Page

#### Morning: Fleet Summary Component
- [ ] 🔴 Create `dashboard/components/fleet/fleet-summary.tsx`
  ```typescript
  'use client';
  
  import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
  import { Robot } from '@/types';
  
  interface FleetSummaryProps {
    robots: Robot[];
  }
  
  export function FleetSummary({ robots }: FleetSummaryProps) {
    const totalRobots = robots.length;
    const online = robots.filter((r) => r.operational_status === 'online').length;
    const offline = robots.filter((r) => r.operational_status === 'offline').length;
    const error = robots.filter((r) => r.operational_status === 'error').length;
    
    const avgBattery = robots.length > 0
      ? Math.round(robots.reduce((sum, r) => sum + (r.battery_level || 0), 0) / robots.length)
      : 0;
  
    return (
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Total Robots</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{totalRobots}</div>
          </CardContent>
        </Card>
        
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Online</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-green-600">{online}</div>
            <p className="text-xs text-muted-foreground">
              {totalRobots > 0 ? Math.round((online / totalRobots) * 100) : 0}% of fleet
            </p>
          </CardContent>
        </Card>
        
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Offline</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-red-600">{offline}</div>
            <p className="text-xs text-muted-foreground">
              {error > 0 && `${error} with errors`}
            </p>
          </CardContent>
        </Card>
        
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Avg Battery</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{avgBattery}%</div>
          </CardContent>
        </Card>
      </div>
    );
  }
  ```

#### Afternoon: Robot Card Component
- [ ] 🔴 Create `dashboard/components/fleet/robot-card.tsx`
  ```typescript
  'use client';
  
  import Link from 'next/link';
  import { Card, CardContent } from '@/components/ui/card';
  import { Badge } from '@/components/ui/badge';
  import { Robot } from '@/types';
  import { formatRelativeTime, formatBatteryLevel, getStatusColor } from '@/lib/utils';
  import { Battery, MapPin, Activity } from 'lucide-react';
  
  interface RobotCardProps {
    robot: Robot;
  }
  
  export function RobotCard({ robot }: RobotCardProps) {
    const statusColor = getStatusColor(robot.operational_status || 'unknown');
    
    const getBatteryIcon = (level: number) => {
      if (level > 80) return '🔋';
      if (level > 50) return '🔋';
      if (level > 20) return '🪫';
      return '🪫';
    };
  
    return (
      <Link href={`/robots/${robot.id}`}>
        <Card className="hover:border-gray-400 transition-colors cursor-pointer">
          <CardContent className="p-6">
            {/* Status indicator */}
            <div className="flex items-center justify-between mb-4">
              <Badge variant={robot.operational_status === 'online' ? 'default' : 'destructive'}>
                <Activity className="mr-1 h-3 w-3" />
                {robot.operational_status || 'unknown'}
              </Badge>
              <span className="text-2xl">{getBatteryIcon(robot.battery_level || 0)}</span>
            </div>
            
            {/* Robot info */}
            <h3 className="font-semibold text-lg mb-1">{robot.serial_number}</h3>
            <p className="text-sm text-muted-foreground mb-4">{robot.model}</p>
            
            {/* Metrics */}
            <div className="space-y-2">
              <div className="flex items-center justify-between text-sm">
                <span className="text-muted-foreground">Battery</span>
                <span className="font-medium">{formatBatteryLevel(robot.battery_level || 0)}</span>
              </div>
              
              {robot.location_name && (
                <div className="flex items-center text-sm text-muted-foreground">
                  <MapPin className="mr-1 h-3 w-3" />
                  {robot.location_name}
                </div>
              )}
              
              {robot.last_seen && (
                <div className="text-xs text-muted-foreground">
                  Last seen: {formatRelativeTime(robot.last_seen)}
                </div>
              )}
            </div>
          </CardContent>
        </Card>
      </Link>
    );
  }
  ```

#### End of Day: Status Filter Component
- [ ] Create `dashboard/components/fleet/status-filter.tsx`
  ```typescript
  'use client';
  
  import { Tabs, TabsList, TabsTrigger } from '@/components/ui/tabs';
  
  interface StatusFilterProps {
    value: string;
    onChange: (value: string) => void;
    counts: {
      all: number;
      online: number;
      offline: number;
      error: number;
    };
  }
  
  export function StatusFilter({ value, onChange, counts }: StatusFilterProps) {
    return (
      <Tabs value={value} onValueChange={onChange}>
        <TabsList>
          <TabsTrigger value="all">
            All ({counts.all})
          </TabsTrigger>
          <TabsTrigger value="online">
            Online ({counts.online})
          </TabsTrigger>
          <TabsTrigger value="offline">
            Offline ({counts.offline})
          </TabsTrigger>
          <TabsTrigger value="error">
            Error ({counts.error})
          </TabsTrigger>
        </TabsList>
      </Tabs>
    );
  }
  ```

---

### Day 2: Tuesday - Fleet Page Implementation

#### All Day: Fleet Overview Page
- [ ] 🔴 Create `dashboard/app/(dashboard)/fleet/page.tsx`
  ```typescript
  'use client';
  
  import { useState } from 'react';
  import { useRobots } from '@/hooks/use-robots';
  import { FleetSummary } from '@/components/fleet/fleet-summary';
  import { RobotCard } from '@/components/fleet/robot-card';
  import { StatusFilter } from '@/components/fleet/status-filter';
  import { Input } from '@/components/ui/input';
  import { Button } from '@/components/ui/button';
  import { Search, RefreshCw } from 'lucide-react';
  
  export default function FleetPage() {
    const [statusFilter, setStatusFilter] = useState('all');
    const [searchQuery, setSearchQuery] = useState('');
    
    const { data, isLoading, error, refetch } = useRobots({
      status: statusFilter !== 'all' ? statusFilter : undefined,
    });
  
    if (isLoading) {
      return (
        <div className="flex h-full items-center justify-center">
          <div className="text-lg">Loading robots...</div>
        </div>
      );
    }
  
    if (error) {
      return (
        <div className="flex h-full items-center justify-center">
          <div className="text-red-600">Failed to load robots</div>
        </div>
      );
    }
  
    const robots = data?.data || [];
    
    // Filter by search query
    const filteredRobots = robots.filter((robot) =>
      robot.serial_number.toLowerCase().includes(searchQuery.toLowerCase()) ||
      robot.model.toLowerCase().includes(searchQuery.toLowerCase())
    );
    
    // Calculate counts
    const counts = {
      all: robots.length,
      online: robots.filter((r) => r.operational_status === 'online').length,
      offline: robots.filter((r) => r.operational_status === 'offline').length,
      error: robots.filter((r) => r.operational_status === 'error').length,
    };
  
    return (
      <div className="space-y-6">
        <div className="flex items-center justify-between">
          <h1 className="text-3xl font-bold">Fleet Overview</h1>
          <Button variant="outline" size="sm" onClick={() => refetch()}>
            <RefreshCw className="mr-2 h-4 w-4" />
            Refresh
          </Button>
        </div>
  
        {/* Fleet summary */}
        <FleetSummary robots={robots} />
  
        {/* Filters */}
        <div className="flex items-center gap-4">
          <StatusFilter
            value={statusFilter}
            onChange={setStatusFilter}
            counts={counts}
          />
          
          <div className="relative flex-1 max-w-sm">
            <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
            <Input
              placeholder="Search robots..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="pl-10"
            />
          </div>
        </div>
  
        {/* Robot grid */}
        {filteredRobots.length === 0 ? (
          <div className="text-center py-12 text-muted-foreground">
            No robots found
          </div>
        ) : (
          <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
            {filteredRobots.map((robot) => (
              <RobotCard key={robot.id} robot={robot} />
            ))}
          </div>
        )}
      </div>
    );
  }
  ```
- [ ] Test fleet page with real data (requires backend running with robots)
- [ ] Test search functionality
- [ ] Test status filters
- [ ] Commit fleet overview
  ```bash
  git add .
  git commit -m "feat(dashboard): implement fleet overview page"
  ```

---

### Day 3: Wednesday - Robot Detail Page

#### Morning: Telemetry Charts
- [ ] Install chart library
  ```bash
  npm install recharts
  ```
- [ ] 🔴 Create `dashboard/components/charts/battery-chart.tsx`
  ```typescript
  'use client';
  
  import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
  import { Telemetry } from '@/types';
  
  interface BatteryChartProps {
    telemetry: Telemetry[];
  }
  
  export function BatteryChart({ telemetry }: BatteryChartProps) {
    const data = telemetry.map((t) => ({
      time: new Date(t.timestamp).toLocaleTimeString(),
      battery: Math.round(t.battery_level),
    }));
  
    return (
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis dataKey="time" />
          <YAxis domain={[0, 100]} />
          <Tooltip />
          <Line type="monotone" dataKey="battery" stroke="#3b82f6" strokeWidth={2} />
        </LineChart>
      </ResponsiveContainer>
    );
  }
  ```
- [ ] Create `dashboard/components/charts/temperature-chart.tsx`
  ```typescript
  'use client';
  
  import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
  import { Telemetry } from '@/types';
  
  interface TemperatureChartProps {
    telemetry: Telemetry[];
  }
  
  export function TemperatureChart({ telemetry }: TemperatureChartProps) {
    const data = telemetry
      .filter((t) => t.metrics?.cpu_temp !== undefined)
      .map((t) => ({
        time: new Date(t.timestamp).toLocaleTimeString(),
        temp: Math.round(t.metrics!.cpu_temp!),
      }));
  
    return (
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis dataKey="time" />
          <YAxis domain={[0, 100]} />
          <Tooltip />
          <Line type="monotone" dataKey="temp" stroke="#ef4444" strokeWidth={2} />
        </LineChart>
      </ResponsiveContainer>
    );
  }
  ```

#### Afternoon: Robot Detail Page
- [ ] 🔴 Create `dashboard/app/(dashboard)/robots/[id]/page.tsx`
  ```typescript
  'use client';
  
  import { use } from 'react';
  import Link from 'next/link';
  import { useRobot } from '@/hooks/use-robots';
  import { useRobotTelemetry } from '@/hooks/use-telemetry';
  import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
  import { Badge } from '@/components/ui/badge';
  import { Button } from '@/components/ui/button';
  import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
  import { BatteryChart } from '@/components/charts/battery-chart';
  import { TemperatureChart } from '@/components/charts/temperature-chart';
  import { ArrowLeft } from 'lucide-react';
  import { formatRelativeTime } from '@/lib/utils';
  
  export default function RobotDetailPage({ params }: { params: Promise<{ id: string }> }) {
    const { id } = use(params);
    const { data: robotData, isLoading: robotLoading } = useRobot(id);
    const { data: telemetryData, isLoading: telemetryLoading } = useRobotTelemetry(id, {
      limit: 100,
    });
  
    if (robotLoading) {
      return <div>Loading robot details...</div>;
    }
  
    const robot = robotData?.data;
    const telemetry = telemetryData?.data || [];
  
    if (!robot) {
      return <div>Robot not found</div>;
    }
  
    return (
      <div className="space-y-6">
        {/* Header */}
        <div className="flex items-center gap-4">
          <Link href="/fleet">
            <Button variant="ghost" size="sm">
              <ArrowLeft className="mr-2 h-4 w-4" />
              Back to Fleet
            </Button>
          </Link>
        </div>
  
        <div className="flex items-center justify-between">
          <div>
            <h1 className="text-3xl font-bold">{robot.serial_number}</h1>
            <p className="text-muted-foreground">{robot.model}</p>
          </div>
          <Badge variant={robot.operational_status === 'online' ? 'default' : 'destructive'}>
            {robot.operational_status || 'unknown'}
          </Badge>
        </div>
  
        {/* Overview cards */}
        <div className="grid gap-4 md:grid-cols-3">
          <Card>
            <CardHeader>
              <CardTitle className="text-sm">Battery Level</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{Math.round(robot.battery_level || 0)}%</div>
            </CardContent>
          </Card>
          
          <Card>
            <CardHeader>
              <CardTitle className="text-sm">Last Seen</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">
                {robot.last_seen ? formatRelativeTime(robot.last_seen) : 'Never'}
              </div>
            </CardContent>
          </Card>
          
          <Card>
            <CardHeader>
              <CardTitle className="text-sm">Location</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{robot.location_name || 'Unknown'}</div>
            </CardContent>
          </Card>
        </div>
  
        {/* Tabs */}
        <Tabs defaultValue="telemetry" className="space-y-4">
          <TabsList>
            <TabsTrigger value="telemetry">Telemetry</TabsTrigger>
            <TabsTrigger value="details">Details</TabsTrigger>
            <TabsTrigger value="alerts">Alerts</TabsTrigger>
          </TabsList>
  
          <TabsContent value="telemetry" className="space-y-4">
            <Card>
              <CardHeader>
                <CardTitle>Battery Level (Last 24h)</CardTitle>
              </CardHeader>
              <CardContent>
                {telemetryLoading ? (
                  <div>Loading chart...</div>
                ) : (
                  <BatteryChart telemetry={telemetry} />
                )}
              </CardContent>
            </Card>
            
            <Card>
              <CardHeader>
                <CardTitle>CPU Temperature (Last 24h)</CardTitle>
              </CardHeader>
              <CardContent>
                {telemetryLoading ? (
                  <div>Loading chart...</div>
                ) : (
                  <TemperatureChart telemetry={telemetry} />
                )}
              </CardContent>
            </Card>
          </TabsContent>
  
          <TabsContent value="details">
            <Card>
              <CardHeader>
                <CardTitle>Robot Details</CardTitle>
              </CardHeader>
              <CardContent>
                <dl className="grid grid-cols-2 gap-4">
                  <div>
                    <dt className="text-sm font-medium text-muted-foreground">Serial Number</dt>
                    <dd className="text-sm">{robot.serial_number}</dd>
                  </div>
                  <div>
                    <dt className="text-sm font-medium text-muted-foreground">Model</dt>
                    <dd className="text-sm">{robot.model}</dd>
                  </div>
                  <div>
                    <dt className="text-sm font-medium text-muted-foreground">Manufacturer</dt>
                    <dd className="text-sm">{robot.manufacturer || 'N/A'}</dd>
                  </div>
                  <div>
                    <dt className="text-sm font-medium text-muted-foreground">Firmware</dt>
                    <dd className="text-sm">{robot.firmware_version || 'N/A'}</dd>
                  </div>
                  <div>
                    <dt className="text-sm font-medium text-muted-foreground">Registered</dt>
                    <dd className="text-sm">{new Date(robot.registered_at).toLocaleString()}</dd>
                  </div>
                  <div>
                    <dt className="text-sm font-medium text-muted-foreground">Status</dt>
                    <dd className="text-sm">{robot.status}</dd>
                  </div>
                </dl>
              </CardContent>
            </Card>
          </TabsContent>
  
          <TabsContent value="alerts">
            <Card>
              <CardHeader>
                <CardTitle>Recent Alerts</CardTitle>
              </CardHeader>
              <CardContent>
                <p className="text-muted-foreground">Alert history will be displayed here</p>
              </CardContent>
            </Card>
          </TabsContent>
        </Tabs>
      </div>
    );
  }
  ```

#### End of Day: Test Robot Detail
- [ ] Test robot detail page with different robots
- [ ] Test charts with real telemetry data
- [ ] Test tab navigation
- [ ] Commit robot detail
  ```bash
  git add .
  git commit -m "feat(dashboard): add robot detail page with charts"
  ```

---

### Day 4: Thursday - WebSocket Implementation

#### Morning: WebSocket Hook
- [ ] 🔴 Create `dashboard/hooks/use-websocket.ts`
  ```typescript
  'use client';
  
  import { useEffect, useRef, useState } from 'react';
  
  interface WebSocketMessage {
    type: string;
    [key: string]: any;
  }
  
  export function useWebSocket(url: string) {
    const [isConnected, setIsConnected] = useState(false);
    const [lastMessage, setLastMessage] = useState<WebSocketMessage | null>(null);
    const wsRef = useRef<WebSocket | null>(null);
    const reconnectTimeoutRef = useRef<NodeJS.Timeout>();
  
    useEffect(() => {
      const connect = () => {
        try {
          const token = localStorage.getItem('auth_token');
          const wsUrl = `${url}?token=${token}`;
          
          const ws = new WebSocket(wsUrl);
          wsRef.current = ws;
  
          ws.onopen = () => {
            console.log('WebSocket connected');
            setIsConnected(true);
          };
  
          ws.onmessage = (event) => {
            try {
              const message = JSON.parse(event.data);
              setLastMessage(message);
            } catch (error) {
              console.error('Failed to parse WebSocket message:', error);
            }
          };
  
          ws.onerror = (error) => {
            console.error('WebSocket error:', error);
          };
  
          ws.onclose = () => {
            console.log('WebSocket disconnected');
            setIsConnected(false);
            
            // Attempt to reconnect after 5 seconds
            reconnectTimeoutRef.current = setTimeout(() => {
              console.log('Attempting to reconnect...');
              connect();
            }, 5000);
          };
        } catch (error) {
          console.error('Failed to connect WebSocket:', error);
        }
      };
  
      connect();
  
      return () => {
        if (reconnectTimeoutRef.current) {
          clearTimeout(reconnectTimeoutRef.current);
        }
        if (wsRef.current) {
          wsRef.current.close();
        }
      };
    }, [url]);
  
    return { isConnected, lastMessage };
  }
  ```

#### Afternoon: WebSocket Integration
- [ ] Update `dashboard/app/(dashboard)/fleet/page.tsx` to use WebSocket
  ```typescript
  'use client';
  
  import { useEffect } from 'react';
  import { useQueryClient } from '@tanstack/react-query';
  import { useWebSocket } from '@/hooks/use-websocket';
  
  export default function FleetPage() {
    const queryClient = useQueryClient();
    const WS_URL = process.env.NEXT_PUBLIC_WS_URL || 'ws://localhost:8080/api/v1/ws';
    const { isConnected, lastMessage } = useWebSocket(WS_URL);
  
    // Handle WebSocket messages
    useEffect(() => {
      if (!lastMessage) return;
  
      switch (lastMessage.type) {
        case 'telemetry_update':
        case 'status_change':
          // Invalidate robots query to refetch
          queryClient.invalidateQueries({ queryKey: ['robots'] });
          break;
        
        case 'alert_triggered':
          // Invalidate alerts query
          queryClient.invalidateQueries({ queryKey: ['alerts'] });
          // Could also show a toast notification
          break;
      }
    }, [lastMessage, queryClient]);
  
    // ... rest of component
    
    // Add connection indicator
    return (
      <div className="space-y-6">
        <div className="flex items-center justify-between">
          <h1 className="text-3xl font-bold">Fleet Overview</h1>
          <div className="flex items-center gap-2">
            <div className={`h-2 w-2 rounded-full ${isConnected ? 'bg-green-500' : 'bg-red-500'}`} />
            <span className="text-sm text-muted-foreground">
              {isConnected ? 'Connected' : 'Disconnected'}
            </span>
          </div>
        </div>
        {/* ... rest of page */}
      </div>
    );
  }
  ```

#### End of Day: Test Real-time Updates
- [ ] 🔴 End-to-end test: Start simulator → Watch dashboard update in real-time
  ```bash
  # Terminal 1: Start all services
  docker-compose up -d
  cd backend && go run cmd/api/main.go
  
  # Terminal 2: Start dashboard
  cd dashboard && npm run dev
  
  # Terminal 3: Run simulator
  cd agent && python -m humanoidops_agent.cli --num-robots 10
  
  # Verify dashboard updates automatically
  ```
- [ ] Test WebSocket reconnection (stop/start backend)
- [ ] Commit WebSocket integration
  ```bash
  git add .
  git commit -m "feat(dashboard): add WebSocket for real-time updates"
  ```

---

### Day 5: Friday - Alerts Page

#### Morning: Alert Components
- [ ] 🔴 Create `dashboard/components/alerts/alert-card.tsx`
  ```typescript
  'use client';
  
  import { Card, CardContent } from '@/components/ui/card';
  import { Badge } from '@/components/ui/badge';
  import { Button } from '@/components/ui/button';
  import { Alert as AlertType } from '@/types';
  import { AlertCircle, Check } from 'lucide-react';
  import { formatRelativeTime } from '@/lib/utils';
  
  interface AlertCardProps {
    alert: AlertType;
    onAcknowledge?: (id: string) => void;
  }
  
  export function AlertCard({ alert, onAcknowledge }: AlertCardProps) {
    const severityColor = {
      info: 'bg-blue-100 text-blue-800',
      warning: 'bg-yellow-100 text-yellow-800',
      critical: 'bg-red-100 text-red-800',
    }[alert.severity];
  
    return (
      <Card className={alert.acknowledged ? 'opacity-60' : ''}>
        <CardContent className="flex items-start gap-4 p-4">
          <AlertCircle className={`mt-1 h-5 w-5 ${alert.severity === 'critical' ? 'text-red-600' : 'text-yellow-600'}`} />
          
          <div className="flex-1 space-y-2">
            <div className="flex items-center justify-between">
              <div className="flex items-center gap-2">
                <span className="font-medium">{alert.robot_serial || alert.robot_id}</span>
                <Badge className={severityColor}>{alert.severity}</Badge>
              </div>
              <span className="text-sm text-muted-foreground">
                {formatRelativeTime(alert.triggered_at)}
              </span>
            </div>
            
            <p className="text-sm">{alert.message}</p>
            
            {alert.acknowledged && (
              <p className="text-xs text-muted-foreground">
                Acknowledged {formatRelativeTime(alert.acknowledged_at!)}
              </p>
            )}
          </div>
          
          {!alert.acknowledged && onAcknowledge && (
            <Button
              size="sm"
              variant="outline"
              onClick={() => onAcknowledge(alert.id)}
            >
              <Check className="mr-2 h-4 w-4" />
              Acknowledge
            </Button>
          )}
        </CardContent>
      </Card>
    );
  }
  ```

#### Afternoon: Alerts Page
- [ ] 🔴 Create `dashboard/app/(dashboard)/alerts/page.tsx`
  ```typescript
  'use client';
  
  import { useState } from 'react';
  import { useAlerts, useAcknowledgeAlert } from '@/hooks/use-alerts';
  import { AlertCard } from '@/components/alerts/alert-card';
  import { Tabs, TabsList, TabsTrigger } from '@/components/ui/tabs';
  import { useToast } from '@/components/ui/use-toast';
  
  export default function AlertsPage() {
    const [filter, setFilter] = useState('active');
    const { toast } = useToast();
    
    const { data, isLoading } = useAlerts({
      acknowledged: filter === 'acknowledged',
    });
    
    const acknowledgeMutation = useAcknowledgeAlert();
  
    const handleAcknowledge = async (id: string) => {
      try {
        await acknowledgeMutation.mutateAsync(id);
        toast({
          title: 'Alert acknowledged',
          description: 'The alert has been marked as acknowledged.',
        });
      } catch (error) {
        toast({
          title: 'Failed to acknowledge alert',
          description: 'Please try again.',
          variant: 'destructive',
        });
      }
    };
  
    if (isLoading) {
      return <div>Loading alerts...</div>;
    }
  
    const alerts = data?.data || [];
    const activeAlerts = alerts.filter((a) => !a.acknowledged);
    const acknowledgedAlerts = alerts.filter((a) => a.acknowledged);
  
    return (
      <div className="space-y-6">
        <div className="flex items-center justify-between">
          <h1 className="text-3xl font-bold">Alerts</h1>
          <div className="text-sm text-muted-foreground">
            {activeAlerts.length} active alert{activeAlerts.length !== 1 ? 's' : ''}
          </div>
        </div>
  
        <Tabs value={filter} onValueChange={setFilter}>
          <TabsList>
            <TabsTrigger value="active">Active ({activeAlerts.length})</TabsTrigger>
            <TabsTrigger value="acknowledged">Acknowledged ({acknowledgedAlerts.length})</TabsTrigger>
            <TabsTrigger value="all">All ({alerts.length})</TabsTrigger>
          </TabsList>
        </Tabs>
  
        <div className="space-y-4">
          {alerts.length === 0 ? (
            <div className="text-center py-12 text-muted-foreground">
              No alerts found
            </div>
          ) : (
            alerts.map((alert) => (
              <AlertCard
                key={alert.id}
                alert={alert}
                onAcknowledge={handleAcknowledge}
              />
            ))
          )}
        </div>
      </div>
    );
  }
  ```

#### End of Week: Week 6 Review
- [ ] ✅ Week 6 deliverables complete:
  - [ ] Fleet summary component
  - [ ] Robot card component
  - [ ] Status filters
  - [ ] Fleet overview page with search
  - [ ] Robot detail page
  - [ ] Telemetry charts (battery, temperature)
  - [ ] WebSocket integration
  - [ ] Real-time updates working
  - [ ] Alerts page with acknowledge functionality
- [ ] 🔴 Full integration test: Simulator → Real-time dashboard updates
- [ ] Test all pages on different screen sizes (responsive)
- [ ] Create PR: Week 6 changes
- [ ] Merge and tag: `v0.1.0-week6`

---

## Week 7: Alert Engine & Backend Integration

**Goal**: Implement alert engine in backend and complete WebSocket server

### Day 1: Monday - Alert Rule Engine (Backend)

#### Morning: Alert Engine Core
- [ ] 🔴 Create `backend/internal/alert/engine.go`
  ```go
  package alert
  
  import (
      "log"
      "time"
      
      "github.com/google/uuid"
      "gorm.io/gorm"
      
      "github.com/YOUR_USERNAME/humanoidops/internal/models"
  )
  
  type AlertEngine struct {
      db    *gorm.DB
      rules []AlertRule
  }
  
  type AlertRule func(robot *models.Robot, telemetry *models.Telemetry) *models.Alert
  
  func NewAlertEngine(db *gorm.DB) *AlertEngine {
      return &AlertEngine{
          db: db,
          rules: []AlertRule{
              checkRobotOffline,
              checkLowBattery,
              checkHighTemperature,
              checkErrorStatus,
          },
      }
  }
  
  func (e *AlertEngine) EvaluateAlerts(robotID uuid.UUID, telemetry *models.Telemetry) {
      // Fetch robot
      var robot models.Robot
      if err := e.db.First(&robot, robotID).Error; err != nil {
          log.Printf("Failed to fetch robot: %v", err)
          return
      }
      
      // Evaluate each rule
      for _, rule := range e.rules {
          if alert := rule(&robot, telemetry); alert != nil {
              // Check if alert already exists
              var existing models.Alert
              err := e.db.Where("robot_id = ? AND type = ? AND acknowledged = ?", 
                  alert.RobotID, alert.Type, false).First(&existing).Error
              
              if err == gorm.ErrRecordNotFound {
                  // Create new alert
                  if err := e.db.Create(alert).Error; err != nil {
                      log.Printf("Failed to create alert: %v", err)
                  } else {
                      log.Printf("🚨 Alert triggered: %s for robot %s", alert.Type, alert.RobotID)
                      // TODO: Broadcast via WebSocket
                  }
              }
          }
      }
      
      // Auto-resolve alerts if conditions cleared
      e.autoResolveAlerts(&robot, telemetry)
  }
  
  func (e *AlertEngine) autoResolveAlerts(robot *models.Robot, telemetry *models.Telemetry) {
      // Auto-acknowledge alerts if condition resolved
      // For example, if battery is back above 25%, resolve low battery alert
      if telemetry.BatteryLevel > 25.0 {
          e.db.Model(&models.Alert{}).
              Where("robot_id = ? AND type = ? AND acknowledged = ?", 
                  robot.ID, "low_battery", false).
              Update("acknowledged", true)
      }
      
      // If robot is online, resolve offline alert
      if telemetry.Status != "offline" {
          e.db.Model(&models.Alert{}).
              Where("robot_id = ? AND type = ? AND acknowledged = ?", 
                  robot.ID, "robot_offline", false).
              Update("acknowledged", true)
      }
  }
  ```

#### Afternoon: Alert Rules
- [ ] 🔴 Create `backend/internal/alert/rules.go`
  ```go
  package alert
  
  import (
      "fmt"
      "time"
      
      "github.com/YOUR_USERNAME/humanoidops/internal/models"
  )
  
  func checkRobotOffline(robot *models.Robot, telemetry *models.Telemetry) *models.Alert {
      if robot.LastSeen == nil {
          return nil
      }
      
      timeSinceLastSeen := time.Since(*robot.LastSeen)
      if timeSinceLastSeen > 30*time.Second {
          return &models.Alert{
              RobotID:  robot.ID,
              Type:     "robot_offline",
              Severity: "critical",
              Message:  fmt.Sprintf("Robot has been offline for %d seconds", int(timeSinceLastSeen.Seconds())),
          }
      }
      
      return nil
  }
  
  func checkLowBattery(robot *models.Robot, telemetry *models.Telemetry) *models.Alert {
      if telemetry.BatteryLevel < 20.0 {
          severity := "warning"
          if telemetry.BatteryLevel < 10.0 {
              severity = "critical"
          }
          
          return &models.Alert{
              RobotID:  robot.ID,
              Type:     "low_battery",
              Severity: severity,
              Message:  fmt.Sprintf("Battery level critically low (%.1f%%)", telemetry.BatteryLevel),
          }
      }
      
      return nil
  }
  
  func checkHighTemperature(robot *models.Robot, telemetry *models.Telemetry) *models.Alert {
      if telemetry.Metrics == nil {
          return nil
      }
      
      metrics := *telemetry.Metrics
      if cpuTemp, ok := metrics["cpu_temp"].(float64); ok {
          if cpuTemp > 80.0 {
              severity := "warning"
              if cpuTemp > 85.0 {
                  severity = "critical"
              }
              
              return &models.Alert{
                  RobotID:  robot.ID,
                  Type:     "high_temperature",
                  Severity: severity,
                  Message:  fmt.Sprintf("CPU temperature above threshold (%.1f°C)", cpuTemp),
              }
          }
      }
      
      return nil
  }
  
  func checkErrorStatus(robot *models.Robot, telemetry *models.Telemetry) *models.Alert {
      if telemetry.Status == "error" {
          errorCodes := ""
          if telemetry.Metrics != nil {
              if codes, ok := (*telemetry.Metrics)["error_codes"].([]interface{}); ok {
                  errorCodes = fmt.Sprintf(": %v", codes)
              }
          }
          
          return &models.Alert{
              RobotID:  robot.ID,
              Type:     "error_status",
              Severity: "critical",
              Message:  fmt.Sprintf("Robot reporting error status%s", errorCodes),
          }
      }
      
      return nil
  }
  ```

#### End of Day: Integrate Alert Engine
- [ ] Update `backend/internal/mqtt/handler.go` to call alert engine
  ```go
  import "github.com/YOUR_USERNAME/humanoidops/internal/alert"
  
  var alertEngine *alert.AlertEngine
  
  func init() {
      // Initialize in main.go instead
  }
  
  func HandleTelemetryMessage(client mqtt.Client, msg mqtt.Message) {
      // ... existing parsing code ...
      
      // Save telemetry
      if err := db.Create(&telemetry).Error; err != nil {
          log.Printf("❌ Failed to save telemetry: %v", err)
          return
      }
      
      // Update robot last_seen
      now := time.Now()
      db.Model(&models.Robot{}).
          Where("id = ?", robotID).
          Update("last_seen", now)
      
      // Evaluate alerts
      if alertEngine != nil {
          alertEngine.EvaluateAlerts(robotID, &telemetry)
      }
      
      log.Printf("✅ Saved telemetry for robot %s", robotID)
  }
  ```
- [ ] Update `backend/cmd/mqtt-handler/main.go`
  ```go
  // Initialize alert engine
  alertEngine := alert.NewAlertEngine(database.GetDB())
  mqtt.SetAlertEngine(alertEngine)
  ```
- [ ] Commit alert engine
  ```bash
  git add .
  git commit -m "feat(backend): implement alert rule engine"
  ```

---

### Day 2: Tuesday - WebSocket Server

#### Morning: WebSocket Manager
- [ ] 🔴 Create `backend/internal/websocket/manager.go`
  ```go
  package websocket
  
  import (
      "log"
      "sync"
      
      "github.com/gorilla/websocket"
  )
  
  type Manager struct {
      clients    map[*Client]bool
      broadcast  chan []byte
      register   chan *Client
      unregister chan *Client
      mu         sync.RWMutex
  }
  
  func NewManager() *Manager {
      return &Manager{
          clients:    make(map[*Client]bool),
          broadcast:  make(chan []byte),
          register:   make(chan *Client),
          unregister: make(chan *Client),
      }
  }
  
  func (m *Manager) Run() {
      for {
          select {
          case client := <-m.register:
              m.mu.Lock()
              m.clients[client] = true
              m.mu.Unlock()
              log.Printf("WebSocket client registered (total: %d)", len(m.clients))
          
          case client := <-m.unregister:
              m.mu.Lock()
              if _, ok := m.clients[client]; ok {
                  delete(m.clients, client)
                  close(client.send)
              }
              m.mu.Unlock()
              log.Printf("WebSocket client unregistered (total: %d)", len(m.clients))
          
          case message := <-m.broadcast:
              m.mu.RLock()
              for client := range m.clients {
                  select {
                  case client.send <- message:
                  default:
                      close(client.send)
                      delete(m.clients, client)
                  }
              }
              m.mu.RUnlock()
          }
      }
  }
  
  func (m *Manager) Broadcast(message []byte) {
      m.broadcast <- message
  }
  
  func (m *Manager) RegisterClient(client *Client) {
      m.register <- client
  }
  
  func (m *Manager) UnregisterClient(client *Client) {
      m.unregister <- client
  }
  ```
- [ ] Create `backend/internal/websocket/client.go`
  ```go
  package websocket
  
  import (
      "log"
      "time"
      
      "github.com/gorilla/websocket"
  )
  
  const (
      writeWait      = 10 * time.Second
      pongWait       = 60 * time.Second
      pingPeriod     = (pongWait * 9) / 10
      maxMessageSize = 512
  )
  
  type Client struct {
      manager *Manager
      conn    *websocket.Conn
      send    chan []byte
  }
  
  func NewClient(manager *Manager, conn *websocket.Conn) *Client {
      return &Client{
          manager: manager,
          conn:    conn,
          send:    make(chan []byte, 256),
      }
  }
  
  func (c *Client) ReadPump() {
      defer func() {
          c.manager.UnregisterClient(c)
          c.conn.Close()
      }()
      
      c.conn.SetReadDeadline(time.Now().Add(pongWait))
      c.conn.SetPongHandler(func(string) error {
          c.conn.SetReadDeadline(time.Now().Add(pongWait))
          return nil
      })
      
      for {
          _, _, err := c.conn.ReadMessage()
          if err != nil {
              if websocket.IsUnexpectedCloseError(err, websocket.CloseGoingAway, websocket.CloseAbnormalClosure) {
                  log.Printf("WebSocket error: %v", err)
              }
              break
          }
      }
  }
  
  func (c *Client) WritePump() {
      ticker := time.NewTicker(pingPeriod)
      defer func() {
          ticker.Stop()
          c.conn.Close()
      }()
      
      for {
          select {
          case message, ok := <-c.send:
              c.conn.SetWriteDeadline(time.Now().Add(writeWait))
              if !ok {
                  c.conn.WriteMessage(websocket.CloseMessage, []byte{})
                  return
              }
              
              if err := c.conn.WriteMessage(websocket.TextMessage, message); err != nil {
                  return
              }
          
          case <-ticker.C:
              c.conn.SetWriteDeadline(time.Now().Add(writeWait))
              if err := c.conn.WriteMessage(websocket.PingMessage, nil); err != nil {
                  return
              }
          }
      }
  }
  ```

#### Afternoon: WebSocket Handler
- [ ] Install gorilla/websocket
  ```bash
  cd backend
  go get github.com/gorilla/websocket
  ```
- [ ] 🔴 Create `backend/internal/handlers/websocket.go`
  ```go
  package handlers
  
  import (
      "log"
      "net/http"
      
      "github.com/gin-gonic/gin"
      "github.com/gorilla/websocket"
      
      ws "github.com/YOUR_USERNAME/humanoidops/internal/websocket"
      "github.com/YOUR_USERNAME/humanoidops/pkg/jwt"
  )
  
  var upgrader = websocket.Upgrader{
      ReadBufferSize:  1024,
      WriteBufferSize: 1024,
      CheckOrigin: func(r *http.Request) bool {
          return true // Allow all origins in development
      },
  }
  
  var wsManager *ws.Manager
  
  func InitWebSocket() *ws.Manager {
      wsManager = ws.NewManager()
      go wsManager.Run()
      return wsManager
  }
  
  func WebSocketHandler(c *gin.Context) {
      // Get token from query param
      token := c.Query("token")
      if token == "" {
          c.JSON(http.StatusUnauthorized, gin.H{"error": "Token required"})
          return
      }
      
      // Validate token
      _, err := jwt.ValidateToken(token)
      if err != nil {
          c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
          return
      }
      
      // Upgrade to WebSocket
      conn, err := upgrader.Upgrade(c.Writer, c.Request, nil)
      if err != nil {
          log.Printf("Failed to upgrade WebSocket: %v", err)
          return
      }
      
      // Create client
      client := ws.NewClient(wsManager, conn)
      wsManager.RegisterClient(client)
      
      // Start pumps
      go client.WritePump()
      go client.ReadPump()
  }
  
  func GetWebSocketManager() *ws.Manager {
      return wsManager
  }
  ```
- [ ] Update `backend/cmd/api/main.go` to add WebSocket endpoint
  ```go
  import wsHandlers "github.com/YOUR_USERNAME/humanoidops/internal/handlers"
  
  func main() {
      // ... existing setup ...
      
      // Initialize WebSocket
      wsHandlers.InitWebSocket()
      
      // ... existing routes ...
      
      // WebSocket endpoint
      router.GET("/api/v1/ws", wsHandlers.WebSocketHandler)
      
      // ... start server ...
  }
  ```

#### End of Day: Broadcast Alerts via WebSocket
- [ ] Update alert engine to broadcast alerts
  ```go
  // In alert/engine.go
  
  import (
      "encoding/json"
      ws "github.com/YOUR_USERNAME/humanoidops/internal/websocket"
  )
  
  type AlertEngine struct {
      db        *gorm.DB
      rules     []AlertRule
      wsManager *ws.Manager
  }
  
  func NewAlertEngine(db *gorm.DB, wsManager *ws.Manager) *AlertEngine {
      return &AlertEngine{
          db:        db,
          rules:     [...],
          wsManager: wsManager,
      }
  }
  
  func (e *AlertEngine) EvaluateAlerts(robotID uuid.UUID, telemetry *models.Telemetry) {
      // ... existing code ...
      
      if alert created {
          // Broadcast via WebSocket
          if e.wsManager != nil {
              message := map[string]interface{}{
                  "type": "alert_triggered",
                  "alert": alert,
              }
              msgBytes, _ := json.Marshal(message)
              e.wsManager.Broadcast(msgBytes)
          }
      }
  }
  ```
- [ ] Test WebSocket with alerts
  ```bash
  # Terminal 1: Start backend with WebSocket
  cd backend && go run cmd/api/main.go
  
  # Terminal 2: Start dashboard
  cd dashboard && npm run dev
  
  # Terminal 3: Run simulator (will trigger low battery alerts)
  cd agent && python -m humanoidops_agent.cli --num-robots 5
  
  # Watch alerts appear in dashboard in real-time
  ```

---

### Day 3: Wednesday - Telemetry Broadcasting

#### All Day: Real-time Telemetry Updates
- [ ] Update MQTT handler to broadcast telemetry via WebSocket
  ```go
  // In mqtt/handler.go
  
  func HandleTelemetryMessage(client mqtt.Client, msg mqtt.Message) {
      // ... existing parsing and saving ...
      
      // Broadcast telemetry update via WebSocket
      if wsManager != nil {
          message := map[string]interface{}{
              "type":     "telemetry_update",
              "robot_id": robotID.String(),
              "data": map[string]interface{}{
                  "battery_level": telemetry.BatteryLevel,
                  "status":        telemetry.Status,
                  "timestamp":     telemetry.Timestamp,
              },
          }
          msgBytes, _ := json.Marshal(message)
          wsManager.Broadcast(msgBytes)
      }
      
      // Broadcast status change if detected
      // Compare with previous status from database
  }
  ```
- [ ] Update dashboard to handle telemetry updates
  ```typescript
  // In dashboard/app/(dashboard)/fleet/page.tsx
  
  useEffect(() => {
    if (!lastMessage) return;
  
    switch (lastMessage.type) {
      case 'telemetry_update':
        // Update specific robot in cache
        queryClient.setQueryData(['robots'], (old: any) => {
          if (!old) return old;
          return {
            ...old,
            data: old.data.map((robot: Robot) =>
              robot.id === lastMessage.robot_id
                ? { ...robot, ...lastMessage.data }
                : robot
            ),
          };
        });
        break;
      
      case 'status_change':
        queryClient.invalidateQueries({ queryKey: ['robots'] });
        break;
      
      case 'alert_triggered':
        queryClient.invalidateQueries({ queryKey: ['alerts'] });
        // Show toast notification
        toast({
          title: 'New Alert',
          description: lastMessage.alert.message,
          variant: 'destructive',
        });
        break;
    }
  }, [lastMessage, queryClient, toast]);
  ```
- [ ] Test real-time updates
- [ ] Monitor WebSocket performance with many clients

---

### Day 4: Thursday - Performance Optimization

#### Morning: Database Query Optimization
- [ ] Add missing database indexes
  ```sql
  CREATE INDEX CONCURRENTLY idx_telemetry_robot_timestamp 
  ON telemetry(robot_id, timestamp DESC);
  
  CREATE INDEX CONCURRENTLY idx_alerts_robot_acknowledged 
  ON alerts(robot_id, acknowledged);
  ```
- [ ] Optimize robot list query
  ```go
  // Add select to limit columns
  db.Select("id, serial_number, model, status, last_seen").Find(&robots)
  ```
- [ ] Add database connection pooling configuration
  ```go
  sqlDB, err := DB.DB()
  sqlDB.SetMaxIdleConns(10)
  sqlDB.SetMaxOpenConns(100)
  sqlDB.SetConnMaxLifetime(time.Hour)
  ```

#### Afternoon: Frontend Performance
- [ ] Implement pagination for robot list
  ```typescript
  // In fleet page
  const [page, setPage] = useState(1);
  const limit = 20;
  
  const { data } = useRobots({ page, limit });
  ```
- [ ] Add loading skeletons
  ```typescript
  // Create skeleton component
  function RobotCardSkeleton() {
    return (
      <Card>
        <CardContent className="p-6 animate-pulse">
          <div className="h-4 bg-gray-200 rounded w-3/4 mb-4" />
          <div className="h-3 bg-gray-200 rounded w-1/2" />
        </CardContent>
      </Card>
    );
  }
  ```
- [ ] Optimize chart rendering (sample data if too many points)
  ```typescript
  function sampleData(data: any[], maxPoints: number = 100) {
    if (data.length <= maxPoints) return data;
    const step = Math.floor(data.length / maxPoints);
    return data.filter((_, i) => i % step === 0);
  }
  ```

#### End of Day: Load Testing
- [ ] Run load test with k6
  ```javascript
  // loadtests/websocket-load.js
  import ws from 'k6/ws';
  import { check } from 'k6';
  
  export let options = {
    stages: [
      { duration: '30s', target: 50 },
      { duration: '1m', target: 50 },
      { duration: '30s', target: 0 },
    ],
  };
  
  export default function () {
    const url = 'ws://localhost:8080/api/v1/ws?token=...';
    
    const res = ws.connect(url, function (socket) {
      socket.on('open', () => console.log('connected'));
      socket.on('message', (data) => console.log('Message received'));
      socket.on('close', () => console.log('disconnected'));
      
      socket.setTimeout(function () {
        socket.close();
      }, 60000);
    });
    
    check(res, { 'status is 101': (r) => r && r.status === 101 });
  }
  ```
- [ ] Monitor performance metrics
  - API latency (p95, p99)
  - Database query duration
  - WebSocket message delivery time
  - Memory usage

---

### Day 5: Friday - Audit Logs Page

#### Morning: Audit Log Components
- [ ] 🔴 Create `dashboard/app/(dashboard)/audit/page.tsx`
  ```typescript
  'use client';
  
  import { useState } from 'react';
  import { useQuery } from '@tanstack/react-query';
  import { api } from '@/lib/api';
  import {
    Table,
    TableBody,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
  } from '@/components/ui/table';
  import { Badge } from '@/components/ui/badge';
  import { Input } from '@/components/ui/input';
  import { Search } from 'lucide-react';
  
  export default function AuditLogsPage() {
    const [searchQuery, setSearchQuery] = useState('');
    
    const { data, isLoading } = useQuery({
      queryKey: ['audit-logs'],
      queryFn: () => api.getAuditLogs(),
    });
  
    if (isLoading) {
      return <div>Loading audit logs...</div>;
    }
  
    const logs = data?.data || [];
  
    return (
      <div className="space-y-6">
        <div className="flex items-center justify-between">
          <h1 className="text-3xl font-bold">Audit Logs</h1>
        </div>
  
        <div className="relative max-w-sm">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
          <Input
            placeholder="Search logs..."
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
            className="pl-10"
          />
        </div>
  
        <div className="rounded-md border">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Timestamp</TableHead>
                <TableHead>User</TableHead>
                <TableHead>Action</TableHead>
                <TableHead>Resource</TableHead>
                <TableHead>Result</TableHead>
                <TableHead>IP Address</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {logs.map((log: any) => (
                <TableRow key={log.id}>
                  <TableCell className="text-sm">
                    {new Date(log.timestamp).toLocaleString()}
                  </TableCell>
                  <TableCell>{log.username || 'System'}</TableCell>
                  <TableCell className="font-mono text-sm">{log.action}</TableCell>
                  <TableCell className="text-sm">
                    {log.resource_type} ({log.resource_id?.substring(0, 8)})
                  </TableCell>
                  <TableCell>
                    <Badge variant={log.result === 'success' ? 'default' : 'destructive'}>
                      {log.result}
                    </Badge>
                  </TableCell>
                  <TableCell className="text-sm text-muted-foreground">
                    {log.ip_address}
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </div>
      </div>
    );
  }
  ```

#### Afternoon: Final Testing & Polish
- [ ] Test all pages with different user roles
- [ ] Test error states (network errors, API errors)
- [ ] Add loading states everywhere
- [ ] Add empty states with helpful messages
- [ ] Test dark mode (if implemented)
- [ ] Test responsive design on mobile

#### End of Week: Week 7 Review
- [ ] ✅ Week 7 deliverables complete:
  - [ ] Alert rule engine implemented
  - [ ] Alert rules (offline, battery, temp, error)
  - [ ] WebSocket server working
  - [ ] WebSocket manager broadcasting messages
  - [ ] Real-time alert notifications
  - [ ] Real-time telemetry updates
  - [ ] Performance optimizations
  - [ ] Audit logs page
  - [ ] Load testing completed
- [ ] 🔴 Full end-to-end test with 50 robots
- [ ] Performance check: Dashboard responsive with 100 robots
- [ ] Create PR: Week 7 changes
- [ ] Merge and tag: `v0.1.0-week7`

---

## Week 8: Testing, Documentation & Final Release

**Goal**: Complete testing, finalize documentation, and prepare for v0.1 release

### Day 1: Monday - Comprehensive Testing

#### Morning: Unit Tests
- [ ] 🔴 Backend unit tests
  ```bash
  cd backend
  go test ./... -v -coverprofile=coverage.out
  go tool cover -html=coverage.out -o coverage.html
  
  # Check coverage >80%
  go tool cover -func=coverage.out | grep total
  ```
- [ ] Add missing unit tests
  - [ ] Alert engine rules
  - [ ] JWT utilities
  - [ ] Repository layer
  - [ ] Service layer

#### Afternoon: Integration Tests
- [ ] 🔴 Backend integration tests
  ```bash
  # Create integration test suite
  cd backend/tests/integration
  go test -v
  ```
- [ ] Key integration tests:
  - [ ] Robot registration → Database → API retrieval
  - [ ] Telemetry MQTT → Database → API query
  - [ ] Alert trigger → Database → WebSocket broadcast
  - [ ] Authentication flow
  - [ ] RBAC enforcement

#### End of Day: Frontend Tests
- [ ] Dashboard component tests
  ```bash
  cd dashboard
  npm test
  ```
- [ ] E2E tests with Playwright (optional but recommended)
  ```bash
  npx playwright test
  ```
- [ ] Key E2E tests:
  - [ ] Login flow
  - [ ] Fleet page navigation
  - [ ] Robot detail page
  - [ ] Alert acknowledgment
  - [ ] Real-time updates

---

### Day 2: Tuesday - Security & Performance Testing

#### Morning: Security Testing
- [ ] Run dependency vulnerability scans
  ```bash
  # Go
  cd backend
  go list -json -m all | nancy sleuth
  # Or use Snyk
  snyk test
  
  # Python
  cd agent
  safety check
  
  # Node.js
  cd dashboard
  npm audit
  ```
- [ ] Fix all critical and high vulnerabilities
- [ ] Security checklist:
  - [ ] All secrets in environment variables
  - [ ] No hardcoded credentials in code
  - [ ] TLS enforced in production config
  - [ ] JWT tokens expire
  - [ ] CORS properly configured
  - [ ] SQL injection prevention (parameterized queries)
  - [ ] XSS prevention (React escaping)
  - [ ] RBAC working correctly

#### Afternoon: Performance Testing
- [ ] 🔴 Load test: 100 robots at 1Hz for 10 minutes
  ```bash
  cd agent
  python -m humanoidops_agent.cli --num-robots 100 --interval 1
  ```
- [ ] Monitor metrics:
  - [ ] API p95 latency < 200ms
  - [ ] Dashboard load time < 2s
  - [ ] Database CPU < 80%
  - [ ] Memory usage stable (no leaks)
  - [ ] WebSocket connections stable
- [ ] Stress test: 1000 robots
  ```bash
  python -m humanoidops_agent.cli --num-robots 1000 --interval 1
  ```
- [ ] Document performance characteristics

#### End of Day: Bug Fixes
- [ ] Create GitHub issues for all bugs found
- [ ] Prioritize and fix critical bugs
- [ ] Retest after fixes

---

### Day 3: Wednesday - Documentation

#### Morning: API Documentation
- [ ] 🔴 Complete OpenAPI specification
  ```bash
  # Ensure all endpoints documented
  # Add request/response examples
  # Add error responses
  ```
- [ ] Generate API documentation website
  ```bash
  # Using Swagger UI or similar
  # Host at /api/docs
  ```
- [ ] Test all API examples in documentation

#### Afternoon: User Documentation
- [ ] 🔴 Update `README.md`
  ```markdown
  # HumanoidOps v0.1
  
  Open-source operations platform for humanoid robot fleets.
  
  ## Features
  
  - ✅ Robot identity & registration
  - ✅ Real-time telemetry monitoring (MQTT/HTTPS)
  - ✅ Fleet dashboard with live updates
  - ✅ Automated alerting (offline, battery, errors)
  - ✅ Audit logging for compliance
  - ✅ REST API & Python SDK
  - ✅ WebSocket real-time updates
  
  ## Quick Start
  
  ### Prerequisites
  
  - Docker & Docker Compose
  - (Optional) Go 1.21+, Python 3.10+, Node.js 18+ for development
  
  ### Run with Docker Compose
  
  1. Clone repository:
     ```bash
     git clone https://github.com/YOUR_USERNAME/humanoidops.git
     cd humanoidops
     ```
  
  2. Start services:
     ```bash
     docker-compose up -d
     ```
  
  3. Create admin user:
     ```bash
     ./scripts/create-admin.sh
     ```
  
  4. Access dashboard:
     ```
     http://localhost:3000
     Username: admin
     Password: admin
     ```
  
  5. Run robot simulator:
     ```bash
     docker-compose exec agent python -m humanoidops_agent.cli --num-robots 10
     ```
  
  ## Architecture
  
  [Architecture diagram]
  
  See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for details.
  
  ## Documentation
  
  - [User Guide](docs/guides/USER_GUIDE.md)
  - [API Reference](docs/api/README.md)
  - [Developer Guide](docs/guides/DEVELOPMENT.md)
  - [Deployment Guide](docs/guides/DEPLOYMENT.md)
  
  ## License
  
  Apache 2.0 - see [LICENSE](LICENSE)
  ```
- [ ] Create `docs/guides/USER_GUIDE.md`
  ```markdown
  # User Guide
  
  ## Fleet Overview
  
  The fleet overview page shows all registered robots...
  
  ### Robot Status Indicators
  
  - 🟢 **Online**: Robot sending telemetry regularly
  - 🔴 **Offline**: No telemetry for >30 seconds
  - 🟡 **Warning**: Battery low or other warnings
  
  ### Filtering Robots
  
  Use the status filter tabs to view:
  - All robots
  - Only online robots
  - Only offline robots
  - Robots with errors
  
  ## Robot Details
  
  Click on any robot card to view detailed information...
  
  [Continue with all features...]
  ```
- [ ] Create `docs/guides/DEPLOYMENT.md`
  ```markdown
  # Deployment Guide
  
  ## Production Deployment
  
  ### Using Docker Compose
  
  1. Update docker-compose.production.yml...
  2. Set environment variables...
  3. Generate SSL certificates...
  4. Start services...
  
  ### Security Checklist
  
  - [ ] Change default passwords
  - [ ] Generate strong JWT secret (32+ chars)
  - [ ] Enable TLS for all services
  - [ ] Configure firewall rules
  - [ ] Set up regular backups
  - [ ] Configure log aggregation
  - [ ] Set up monitoring alerts
  
  [Continue with details...]
  ```
- [ ] Create `docs/guides/DEVELOPMENT.md`
  ```markdown
  # Development Guide
  
  ## Local Development Setup
  
  ### Backend Development
  
  ```bash
  cd backend
  go run cmd/api/main.go
  ```
  
  ### Frontend Development
  
  ```bash
  cd dashboard
  npm run dev
  ```
  
  ### Agent Development
  
  ```bash
  cd agent
  python -m humanoidops_agent.cli
  ```
  
  ## Testing
  
  [Test commands and guidelines...]
  
  ## Code Style
  
  [Style guidelines...]
  ```

#### End of Day: Architecture Documentation
- [ ] Create architecture diagrams (C4 model)
- [ ] Document design decisions (ADRs)
- [ ] Create sequence diagrams for key flows
- [ ] Update `docs/ARCHITECTURE.md` with all diagrams

---

### Day 4: Thursday - Beta Testing

#### Morning: Beta Testing Preparation
- [ ] Create beta testing checklist
- [ ] Prepare test scenarios document
- [ ] Set up feedback collection (GitHub Discussions or form)
- [ ] Deploy to staging environment

#### Afternoon: Beta Testing
- [ ] 🔴 Invite 2+ external beta testers
- [ ] Provide test scenarios:
  1. Deploy HumanoidOps locally
  2. Register 10 robots
  3. Run simulator for 30 minutes
  4. Test all dashboard features
  5. Test alert acknowledgment
  6. Review audit logs
- [ ] Collect feedback
- [ ] Fix critical issues
- [ ] Document known issues

#### End of Day: Release Preparation
- [ ] Create `CHANGELOG.md`
  ```markdown
  # Changelog
  
  ## [0.1.0] - 2024-01-XX
  
  ### Added
  - Robot identity and registration system
  - Real-time telemetry ingestion via MQTT and HTTPS
  - Fleet dashboard with live updates
  - Automated alert engine (offline, battery, temperature, errors)
  - Complete audit logging
  - REST API with OpenAPI specification
  - Python SDK for robot integration
  - Robot simulator for testing
  - WebSocket real-time updates
  - Role-based access control (admin, operator, viewer)
  
  ### Security
  - JWT authentication
  - TLS support for all communications
  - bcrypt password hashing
  - RBAC implementation
  - Comprehensive audit logging
  
  ### Performance
  - Supports 1000+ concurrent robots
  - Dashboard loads in <2s with 100 robots
  - API p95 latency <200ms
  - TimescaleDB for efficient time-series storage
  ```
- [ ] Create release notes
- [ ] Tag release candidate: `v0.1.0-rc1`

---

### Day 5: Friday - Release Day! 🚀

#### Morning: Pre-Release Checklist
- [ ] ✅ All acceptance criteria met (see PRD.md)
  - [ ] Admin can register 10 simulated robots via API ✅
  - [ ] Robots send telemetry via MQTT and data appears in DB within 1s ✅
  - [ ] Dashboard displays all robots with correct status ✅
  - [ ] Real-time updates work ✅
  - [ ] Alert triggers when robot goes offline ✅
  - [ ] User can acknowledge alert ✅
  - [ ] Audit log shows all actions ✅
  - [ ] API documentation complete ✅
  - [ ] Python SDK works ✅
  - [ ] Security: Zero critical vulnerabilities ✅
  - [ ] Performance: Dashboard <2s with 100 robots ✅
  - [ ] Tests pass (>80% coverage) ✅
  - [ ] Documentation complete ✅
  - [ ] Docker Compose setup works ✅

#### Afternoon: Release Tasks
- [ ] 🔴 Final smoke test on clean machine
  ```bash
  # On fresh Ubuntu VM or container
  git clone https://github.com/YOUR_USERNAME/humanoidops.git
  cd humanoidops
  docker-compose up -d
  # Test all features
  ```
- [ ] Tag final release: `v0.1.0`
  ```bash
  git tag -a v0.1.0 -m "Release v0.1.0 - MVP"
  git push origin v0.1.0
  ```
- [ ] Create GitHub Release
  - [ ] Release title: "HumanoidOps v0.1.0 - MVP Release"
  - [ ] Copy CHANGELOG content
  - [ ] Attach documentation PDF (optional)
  - [ ] Mark as "Latest release"
- [ ] Publish Python SDK to PyPI (optional)
  ```bash
  cd agent
  python -m build
  twine upload dist/*
  ```
- [ ] Publish Docker images to Docker Hub (optional)
  ```bash
  docker build -t humanoidops/api:0.1.0 -f backend/docker/api.Dockerfile backend/
  docker push humanoidops/api:0.1.0
  ```

#### End of Day: Launch Announcements
- [ ] 🔴 Update README with badges:
  ```markdown
  ![Version](https://img.shields.io/badge/version-0.1.0-blue)
  ![License](https://img.shields.io/badge/license-Apache%202.0-green)
  ![Build](https://img.shields.io/github/actions/workflow/status/YOUR_USERNAME/humanoidops/backend.yml)
  ```
- [ ] Write launch blog post (optional)
- [ ] Share on social media:
  - [ ] Reddit (r/robotics, r/golang, r/reactjs)
  - [ ] Hacker News (Show HN)
  - [ ] Twitter/LinkedIn
  - [ ] Dev.to or Medium
- [ ] Set up GitHub Discussions for community
- [ ] Set up issue templates for bug reports and feature requests
- [ ] Plan v0.2 features based on feedback

---

## Post-Release: Week 8 Review

### Deliverables Checklist

#### ✅ Core Features
- [x] Robot identity & registration
- [x] Telemetry ingestion (MQTT + HTTPS)
- [x] Fleet dashboard
- [x] Real-time updates (WebSocket)
- [x] Alert engine with 4+ rules
- [x] Audit logging
- [x] REST API (all endpoints)
- [x] Python SDK
- [x] Robot simulator

#### ✅ Security
- [x] JWT authentication
- [x] RBAC (3 roles)
- [x] Password hashing (bcrypt)
- [x] TLS support
- [x] Audit trail
- [x] Zero critical vulnerabilities

#### ✅ Performance
- [x] 100+ robots supported
- [x] Dashboard <2s load time
- [x] API p95 <200ms
- [x] 30-day data retention
- [x] WebSocket stable with 50+ connections

#### ✅ Testing
- [x] Unit tests >80% coverage (backend)
- [x] Unit tests >70% coverage (frontend)
- [x] Integration tests
- [x] E2E tests (key flows)
- [x] Load testing
- [x] Security scanning

#### ✅ Documentation
- [x] README with quick start
- [x] API documentation (OpenAPI)
- [x] User guide
- [x] Developer guide
- [x] Deployment guide
- [x] Architecture documentation
- [x] CHANGELOG

#### ✅ Release
- [x] v0.1.0 tagged
- [x] GitHub Release created
- [x] Beta tested with 2+ external users
- [x] Known issues documented
- [x] Community setup (Discussions, issues)

---

## Next Steps (Post v0.1)

### Immediate (Week 9-10)
- [ ] Monitor GitHub Issues for bug reports
- [ ] Respond to community questions
- [ ] Fix critical bugs in patch releases (v0.1.1, v0.1.2)
- [ ] Collect feature requests for v0.2

### v0.2 Planning (Future)
Based on PRD future features:
- [ ] OTA firmware updates
- [ ] Tele-operation (secure mode)
- [ ] Predictive diagnostics (ML)
- [ ] Webhook support (Slack, PagerDuty)
- [ ] Advanced analytics
- [ ] Map visualization
- [ ] Multi-tenant support

### Community Building
- [ ] Create ROADMAP.md
- [ ] Set up Discord server (optional)
- [ ] Write tutorial blog posts
- [ ] Create video walkthrough
- [ ] Present at robotics meetups/conferences

---

## 🎉 Congratulations!

**You've completed the HumanoidOps v0.1 MVP!**

### What You Built:
- ✅ Production-ready fleet operations platform
- ✅ 1000s of lines of code (Go, Python, TypeScript)
- ✅ Comprehensive test suite
- ✅ Complete documentation
- ✅ Open-source contribution to robotics ecosystem

### Impact:
- 🤖 Enables organizations to manage robot fleets at scale
- 🔒 Security-first architecture for critical infrastructure
- 🌐 Open-source alternative to vendor-locked solutions
- 📊 Real-time visibility into robot operations
