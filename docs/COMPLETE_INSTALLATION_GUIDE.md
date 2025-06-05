# SIEMZello-core Complete Documentation

## 📋 Table of Contents
1. [System Overview](#system-overview)
2. [Architecture Components](#architecture-components)
3. [Network Configuration](#network-configuration)
4. [Installation Guide](#installation-guide)
5. [Component Setup](#component-setup)
6. [Configuration Management](#configuration-management)
7. [Troubleshooting](#troubleshooting)
8. [API Documentation](#api-documentation)

---

## 🏗️ System Overview

SIEMZello is a distributed SIEM system with AI-powered threat detection capabilities. The system consists of multiple components deployed across different machines for optimal performance and security.

### 🌐 Network Topology

```
┌─────────────────────────────────────────────────────────────┐
│                    SIEM Infrastructure                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   SIEM VM       │    │  Frontend Host  │    │ Monitored Agents│
│ (Kafka + API)   │◄──►│ (UI + AI API)   │◄──►│  (Port 5050)    │
│                 │    │                 │    │                 │
│ • Kafka Broker  │    │ • Next.js UI    │    │ • Metrics API   │
│ • SIEMZello-api │    │ • SIEMZello-ai  │    │ • Log Agents    │
│ • Port 8000     │    │ • Port 3000     │    │ • SSH Access    │
│ • Log Processing│    │ • Port 5000     │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                        │                        │
        └────────────────────────┼────────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  Agent Setup    │
                    │   (setup.zip)   │
                    │                 │
                    │ • Ansible       │
                    │ • Configuration │
                    │ • Auto Deploy   │
                    └─────────────────┘
```

---

## 🔧 Architecture Components

### 1. **SIEMZello-ui (Frontend)**
**Location**: Frontend Host Machine  
**Ports**: 3000 (UI), 5000 (AI API)

**Components:**
- **Next.js Dashboard**: Real-time security monitoring interface
- **React Components**: Interactive charts, tables, forms
- **TypeScript**: Type-safe frontend development
- **TailwindCSS**: Modern UI styling
- **SQLite Database**: Local agent management

**Key Features:**
- Real-time agent monitoring
- Security analytics dashboard
- Agent management interface
- AI-powered threat visualization

### 2. **SIEMZello-api (Backend)**
**Location**: SIEM VM  
**Ports**: 8000 (API)

**Components:**
- **FastAPI Server**: REST API endpoints
- **SQLite Database**: Agent and log storage
- **Kafka Integration**: Message processing
- **Authentication**: JWT-based security

**Key Features:**
- Agent CRUD operations
- Log management and analysis
- Real-time data processing
- Secure API endpoints

### 3. **SIEMZello-ai (AI Engine)**
**Location**: Frontend Host Machine  
**Ports**: 5000 (AI API)

**Components:**
- **ML Models**: Threat detection algorithms
- **FastAPI Server**: AI inference endpoints
- **Scikit-learn**: Machine learning framework
- **Pandas/NumPy**: Data processing

**Key Features:**
- Network traffic analysis
- Memory anomaly detection
- Disk activity monitoring
- Process behavior analysis

### 4. **Kafka Message Broker**
**Location**: SIEM VM  
**Ports**: 9092 (Kafka)

**Components:**
- **Apache Kafka**: Message streaming
- **Topics**: Process, memory, disk, network logs
- **Consumers**: AI analysis pipeline
- **Producers**: Agent log collectors

### 5. **Agent Setup System**
**Location**: Distributed via setup.zip  
**Components:**
- **Ansible Playbooks**: Automated deployment
- **Configuration Files**: Agent settings
- **Metrics Collector**: System monitoring
- **Log Forwarder**: Kafka integration

---

## 🌐 Network Configuration

### **Critical Configuration Requirements**

#### **1. SIEM VM IP Configuration**
```yaml
# File: agent.yml (inside setup.zip)
---
- hosts: all
  vars:
    siem_vm_ip: "192.168.100.49"  # ⚠️ MUST BE UPDATED FOR EACH INSTALLATION
    kafka_brokers:
      - "{{ siem_vm_ip }}:9092"
    api_endpoint: "http://{{ siem_vm_ip }}:8000"
    
  tasks:
    - name: Configure log forwarding
      template:
        src: kafka-config.j2
        dest: /etc/kafka/producer.conf
        vars:
          kafka_bootstrap_servers: "{{ siem_vm_ip }}:9092"
```

#### **2. Port Configuration Matrix**

| Service | Machine | Port | Protocol | Description |
|---------|---------|------|----------|-------------|
| SIEMZello-ui | Frontend Host | 3000 | HTTP | Web dashboard |
| SIEMZello-ai | Frontend Host | 5000 | HTTP | AI inference API |
| SIEMZello-api | SIEM VM | 8000 | HTTP | Backend API |
| Kafka | SIEM VM | 9092 | TCP | Message broker |
| Agent Metrics | Monitored Agents | 5050 | HTTP | System metrics |
| SSH | Monitored Agents | 22 | SSH | Remote access |

#### **3. Network Access Requirements**
```bash
# Frontend Host needs access to:
- SIEM VM:8000 (API access)
- Monitored Agents:5050 (metrics collection)
- Monitored Agents:22 (SSH deployment)

# SIEM VM needs access to:
- Frontend Host:5000 (AI API calls)
- Monitored Agents:* (log collection)

# Monitored Agents need access to:
- SIEM VM:9092 (Kafka log forwarding)
```

---

## 📦 Installation Guide

### **Step 1: Prerequisites**

#### **SIEM VM Requirements**
```bash
# Operating System: Ubuntu 20.04+ / CentOS 7+
# RAM: 8GB minimum, 16GB recommended
# Disk: 100GB minimum
# Network: Static IP address

# Install Docker and Docker Compose
sudo apt update
sudo apt install -y docker.io docker-compose python3 python3-pip

# Install Kafka (via Docker)
sudo docker run -d \
  --name kafka \
  -p 9092:9092 \
  -e KAFKA_ZOOKEEPER_CONNECT=localhost:2181 \
  confluentinc/cp-kafka:latest
```

#### **Frontend Host Requirements**
```bash
# Operating System: Windows 10+ / Ubuntu 20.04+
# RAM: 4GB minimum, 8GB recommended
# Network: Access to SIEM VM and monitored agents

# Install Node.js and Python
# Windows:
choco install nodejs python

# Ubuntu:
sudo apt install -y nodejs npm python3 python3-pip
```

### **Step 2: Clone Repository**
```bash
# Create project directory
mkdir -p /opt/siemzello
cd /opt/siemzello

# Clone all components
git clone https://github.com/SIEMZello/SIEMZello-core.git
git clone https://github.com/SIEMZello/SIEMZello-ui.git
git clone https://github.com/SIEMZello/SIEMZello-api.git
git clone https://github.com/SIEMZello/SIEMZello-ai.git
```

### **Step 3: Component Installation**

#### **Install SIEMZello-api (SIEM VM)**
```bash
cd /opt/siemzello/SIEMZello-api

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
vim .env  # Edit configuration

# Start the API server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

#### **Install SIEMZello-ui (Frontend Host)**
```bash
cd /opt/siemzello/SIEMZello-ui/siemzello-ui

# Install Node.js dependencies
npm install

# Configure environment
cp .env.local.example .env.local
vim .env.local  # Set SIEM VM IP

# Set API endpoint
echo "NEXT_PUBLIC_API_URL=http://SIEM_VM_IP:8000" >> .env.local

# Start the frontend
npm run dev -- --host 0.0.0.0 --port 3000
```

#### **Install SIEMZello-ai (Frontend Host)**
```bash
cd /opt/siemzello/SIEMZello-ai

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start the AI API
uvicorn main:app --host 0.0.0.0 --port 5000 --reload
```

---

## ⚙️ Configuration Management

### **1. Agent Deployment Configuration**

#### **Before Agent Installation - Critical Step**
```bash
# Extract setup.zip
unzip setup.zip -d agent-setup/
cd agent-setup/

# Edit agent.yml - MANDATORY STEP
vim agent.yml

# Update the SIEM VM IP address:
siem_vm_ip: "192.168.100.49"  # Replace with actual SIEM VM IP
kafka_servers: 
  - "192.168.100.49:9092"     # Replace with actual SIEM VM IP
api_base_url: "http://192.168.100.49:8000"  # Replace with actual SIEM VM IP
```

#### **Agent.yml Configuration Template**
```yaml
---
# SIEMZello Agent Configuration
# ⚠️ CRITICAL: Update siem_vm_ip before deployment!

- hosts: all
  become: yes
  vars:
    # ========================================
    # NETWORK CONFIGURATION - UPDATE REQUIRED
    # ========================================
    siem_vm_ip: "192.168.100.49"  # ⚠️ CHANGE THIS IP!
    frontend_host_ip: "192.168.100.50"  # ⚠️ CHANGE THIS IP!
    
    # Derived variables (auto-configured)
    kafka_bootstrap_servers: "{{ siem_vm_ip }}:9092"
    api_endpoint: "http://{{ siem_vm_ip }}:8000"
    ai_endpoint: "http://{{ frontend_host_ip }}:5000"
    
    # Agent Configuration
    agent_user: "siem"
    agent_password: "pass"
    metrics_port: 5050
    log_level: "INFO"
    
  tasks:
    - name: Create SIEM user
      user:
        name: "{{ agent_user }}"
        password: "{{ agent_password | password_hash('sha512') }}"
        groups: sudo
        shell: /bin/bash
        
    - name: Install Python and dependencies
      package:
        name: 
          - python3
          - python3-pip
          - python3-venv
        state: present
        
    - name: Create agent directory
      file:
        path: /opt/siemzello-agent
        state: directory
        owner: "{{ agent_user }}"
        group: "{{ agent_user }}"
        mode: '0755'
        
    - name: Install metrics collector
      copy:
        src: metrics_collector.py
        dest: /opt/siemzello-agent/metrics.py
        owner: "{{ agent_user }}"
        group: "{{ agent_user }}"
        mode: '0755'
        
    - name: Configure Kafka producer
      template:
        src: kafka_config.j2
        dest: /opt/siemzello-agent/kafka.conf
        vars:
          bootstrap_servers: "{{ kafka_bootstrap_servers }}"
          
    - name: Create systemd service
      template:
        src: siemzello-agent.service.j2
        dest: /etc/systemd/system/siemzello-agent.service
        
    - name: Start and enable agent service
      systemd:
        name: siemzello-agent
        enabled: yes
        state: started
        daemon_reload: yes
        
    - name: Configure firewall for metrics port
      ufw:
        rule: allow
        port: "{{ metrics_port }}"
        proto: tcp
```

### **2. Environment Configuration Files**

#### **SIEMZello-api (.env)**
```bash
# Database Configuration
DATABASE_URL=sqlite:///./data/agents.db

# Kafka Configuration  
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
KAFKA_TOPICS_PROCESS=process-logs
KAFKA_TOPICS_MEMORY=memory-logs
KAFKA_TOPICS_DISK=disk-logs
KAFKA_TOPICS_NETWORK=network-logs

# AI API Configuration
AI_ENDPOINTS_PROCESS=http://FRONTEND_HOST_IP:5000/api/v1/predict/process
AI_ENDPOINTS_MEMORY=http://FRONTEND_HOST_IP:5000/api/v1/predict/memory
AI_ENDPOINTS_DISK=http://FRONTEND_HOST_IP:5000/api/v1/predict/disk
AI_ENDPOINTS_NETWORK=http://FRONTEND_HOST_IP:5000/api/v1/predict/network

# Security Configuration
JWT_SECRET_KEY=your-secret-key-here
JWT_ALGORITHM=HS256
JWT_EXPIRATION_TIME=24  # hours

# Logging Configuration
LOG_LEVEL=INFO
LOG_FILE=/var/log/siemzello-api.log
```

#### **SIEMZello-ui (.env.local)**
```bash
# API Configuration
NEXT_PUBLIC_API_URL=http://SIEM_VM_IP:8000
NEXT_PUBLIC_AI_API_URL=http://localhost:5000

# Database Configuration
DATABASE_PATH=./data/agents.db

# Development Configuration
NODE_ENV=development
NEXT_TELEMETRY_DISABLED=1

# Metrics Configuration
METRICS_REFRESH_INTERVAL=15000  # 15 seconds
AGENT_TIMEOUT=5000  # 5 seconds

# Authentication Configuration
NEXTAUTH_SECRET=your-nextauth-secret
NEXTAUTH_URL=http://localhost:3000
```

---

## 🚀 Component Setup

### **1. Setup.zip Deployment Process**

#### **Preparation Phase**
```bash
# 1. Extract setup.zip
unzip setup.zip -d /tmp/agent-setup
cd /tmp/agent-setup

# 2. Inventory Configuration
echo "[agents]" > inventory.ini
echo "192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa" >> inventory.ini
echo "192.168.1.11 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa" >> inventory.ini

# 3. Update agent.yml with correct SIEM VM IP
sed -i 's/siem_vm_ip: ".*"/siem_vm_ip: "192.168.100.49"/' agent.yml

# 4. Verify configuration
grep "siem_vm_ip" agent.yml
```

#### **Deployment Phase**
```bash
# 1. Test connectivity
ansible all -i inventory.ini -m ping

# 2. Deploy agents
ansible-playbook -i inventory.ini agent.yml

# 3. Verify deployment
ansible all -i inventory.ini -m shell -a "systemctl status siemzello-agent"

# 4. Test metrics endpoint
curl http://192.168.1.10:5050/metrics
curl http://192.168.1.11:5050/metrics
```

### **2. Service Management**

#### **Start All Services (Production)**
```bash
#!/bin/bash
# start-siemzello.sh

# Start Kafka (SIEM VM)
sudo docker start kafka

# Start API (SIEM VM)
cd /opt/siemzello/SIEMZello-api
source venv/bin/activate
nohup uvicorn app.main:app --host 0.0.0.0 --port 8000 > api.log 2>&1 &

# Start AI API (Frontend Host)
cd /opt/siemzello/SIEMZello-ai  
source venv/bin/activate
nohup uvicorn main:app --host 0.0.0.0 --port 5000 > ai.log 2>&1 &

# Start Frontend (Frontend Host)
cd /opt/siemzello/SIEMZello-ui/siemzello-ui
nohup npm run build && npm start > ui.log 2>&1 &

echo "SIEMZello services started successfully!"
```

#### **Stop All Services**
```bash
#!/bin/bash
# stop-siemzello.sh

# Stop processes
pkill -f "uvicorn.*8000"  # API
pkill -f "uvicorn.*5000"  # AI API  
pkill -f "next.*3000"     # Frontend

# Stop Kafka
sudo docker stop kafka

echo "SIEMZello services stopped successfully!"
```

### **3. Health Check Script**
```bash
#!/bin/bash
# health-check.sh

echo "=== SIEMZello Health Check ==="

# Check API
if curl -f http://localhost:8000/health > /dev/null 2>&1; then
    echo "✅ API (Port 8000): Healthy"
else
    echo "❌ API (Port 8000): Down"
fi

# Check AI API
if curl -f http://localhost:5000/health > /dev/null 2>&1; then
    echo "✅ AI API (Port 5000): Healthy"
else
    echo "❌ AI API (Port 5000): Down"
fi

# Check Frontend
if curl -f http://localhost:3000 > /dev/null 2>&1; then
    echo "✅ Frontend (Port 3000): Healthy"
else
    echo "❌ Frontend (Port 3000): Down"
fi

# Check Kafka
if sudo docker ps | grep -q kafka; then
    echo "✅ Kafka: Running"
else
    echo "❌ Kafka: Down"
fi

echo "=== End Health Check ==="
```

---

## 🔧 Troubleshooting

### **Common Issues and Solutions**

#### **1. Agent Connection Issues**
```bash
# Problem: Agents showing as offline
# Solution: Check network connectivity and configuration

# Test agent connectivity
curl http://AGENT_IP:5050/metrics

# Check agent logs
ssh siem@AGENT_IP
sudo journalctl -u siemzello-agent -f

# Verify Kafka connectivity
telnet SIEM_VM_IP 9092
```

#### **2. API Connection Issues**
```bash
# Problem: Frontend cannot connect to API
# Solution: Check network and configuration

# Test API connectivity
curl http://SIEM_VM_IP:8000/health

# Check API logs
tail -f /opt/siemzello/SIEMZello-api/api.log

# Verify environment variables
cat /opt/siemzello/SIEMZello-ui/siemzello-ui/.env.local | grep API_URL
```

#### **3. AI API Issues**
```bash
# Problem: AI predictions failing
# Solution: Check AI service and models

# Test AI API
curl -X POST http://localhost:5000/api/v1/predict/network \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'

# Check AI service logs
tail -f /opt/siemzello/SIEMZello-ai/ai.log

# Verify model files
ls -la /opt/siemzello/SIEMZello-ai/*/models/
```

### **4. Setup.zip Configuration Issues**

#### **Problem: Wrong SIEM VM IP in agent.yml**
```bash
# Solution: Update configuration before deployment

# Check current configuration
grep -n "siem_vm_ip" agent.yml

# Update configuration
sed -i 's/siem_vm_ip: ".*"/siem_vm_ip: "192.168.100.49"/' agent.yml

# Verify update
grep "siem_vm_ip" agent.yml

# Re-run deployment
ansible-playbook -i inventory.ini agent.yml
```

#### **Problem: Agents cannot reach Kafka**
```bash
# Solution: Verify Kafka configuration and network

# Test Kafka connectivity from agent
telnet SIEM_VM_IP 9092

# Check Kafka configuration in agent
ssh siem@AGENT_IP
cat /opt/siemzello-agent/kafka.conf

# Update Kafka configuration
sudo sed -i 's/bootstrap.servers=.*/bootstrap.servers=SIEM_VM_IP:9092/' /opt/siemzello-agent/kafka.conf
sudo systemctl restart siemzello-agent
```

### **5. Network Configuration Validation**

#### **Validate Network Connectivity**
```bash
#!/bin/bash
# network-validation.sh

SIEM_VM_IP="192.168.100.49"
FRONTEND_HOST_IP="192.168.100.50"
AGENT_IPS=("192.168.1.10" "192.168.1.11")

echo "=== Network Connectivity Validation ==="

# Test SIEM VM connectivity
echo "Testing SIEM VM ($SIEM_VM_IP):"
nc -zv $SIEM_VM_IP 8000  # API
nc -zv $SIEM_VM_IP 9092  # Kafka

# Test Frontend Host connectivity
echo "Testing Frontend Host ($FRONTEND_HOST_IP):"
nc -zv $FRONTEND_HOST_IP 3000  # UI
nc -zv $FRONTEND_HOST_IP 5000  # AI API

# Test Agent connectivity
for agent_ip in "${AGENT_IPS[@]}"; do
    echo "Testing Agent ($agent_ip):"
    nc -zv $agent_ip 22    # SSH
    nc -zv $agent_ip 5050  # Metrics
done

echo "=== Validation Complete ==="
```

---

## 📚 API Documentation

### **SIEMZello-api Endpoints**

#### **Agent Management**
```bash
# Get all agents
GET http://SIEM_VM_IP:8000/api/agents/

# Create agent
POST http://SIEM_VM_IP:8000/api/agents/
Content-Type: application/json
{
  "hostname": "server-01",
  "ip": "192.168.1.10",
  "mac": "00:11:22:33:44:55",
  "ssh_user": "siem",
  "ssh_pass": "pass",
  "sudo_pass": "pass",
  "interface": "eth0",
  "status": "active"
}

# Update agent
PUT http://SIEM_VM_IP:8000/api/agents/1
Content-Type: application/json
{
  "status": "inactive"
}

# Delete agent
DELETE http://SIEM_VM_IP:8000/api/agents/1

# Get agent statistics
GET http://SIEM_VM_IP:8000/api/agents/stats/overview
```

#### **Log Management**
```bash
# Get all logs
GET http://SIEM_VM_IP:8000/api/agents/logs?limit=100&offset=0

# Get logs for specific agent
GET http://SIEM_VM_IP:8000/api/agents/1/logs?limit=50

# Get logs count
GET http://SIEM_VM_IP:8000/api/agents/logs/count

# Get today's logs count
GET http://SIEM_VM_IP:8000/api/agents/logs/count/today
```

### **SIEMZello-ai Endpoints**

#### **AI Predictions**
```bash
# Network traffic analysis
POST http://FRONTEND_HOST_IP:5000/api/v1/predict/network
Content-Type: application/json
{
  "timestamp": "2024-01-01T12:00:00",
  "source_ip": "192.168.1.100",
  "destination_ip": "203.0.113.1",
  "protocol": "TCP",
  "bytes_sent": 1024,
  "bytes_received": 8192
}

# Memory analysis
POST http://FRONTEND_HOST_IP:5000/api/v1/predict/memory
Content-Type: application/json
{
  "PID": 1234,
  "ts": 1640995200,
  "CMD": "python3",
  "RDDSK": "100",
  "WRDSK": "50"
}

# Process analysis
POST http://FRONTEND_HOST_IP:5000/api/v1/predict/process
Content-Type: application/json
{
  "ts": 1640995200,
  "CMD": "suspicious_process",
  "CPU": 95.5,
  "PID": 1234
}

# Disk analysis
POST http://FRONTEND_HOST_IP:5000/api/v1/predict/disk
Content-Type: application/json
{
  "ts": 1640995200,
  "PID": 1234,
  "CMD": "dd",
  "RDDSK": "1000000",
  "WRDSK": "1000000"
}
```

---

## 🎯 Quick Reference Commands

### **Daily Operations**
```bash
# Start all services
./start-siemzello.sh

# Check system health
./health-check.sh

# View logs
tail -f /opt/siemzello/SIEMZello-api/api.log
tail -f /opt/siemzello/SIEMZello-ai/ai.log

# Restart specific service
sudo systemctl restart siemzello-agent  # On agent machines
```

### **Agent Deployment**
```bash
# Extract and configure setup.zip
unzip setup.zip
cd agent-setup
vim agent.yml  # Update SIEM VM IP
ansible-playbook -i inventory.ini agent.yml
```

### **Network Testing**
```bash
# Test all endpoints
curl http://SIEM_VM_IP:8000/health
curl http://FRONTEND_HOST_IP:3000
curl http://FRONTEND_HOST_IP:5000/health
curl http://AGENT_IP:5050/metrics
```

---

## 📞 Support and Maintenance

### **Log Locations**
- **API Logs**: `/opt/siemzello/SIEMZello-api/api.log`
- **AI Logs**: `/opt/siemzello/SIEMZello-ai/ai.log`
- **Frontend Logs**: `/opt/siemzello/SIEMZello-ui/siemzello-ui/ui.log`
- **Agent Logs**: `/var/log/siemzello-agent.log` (on each agent)

### **Configuration Files**
- **API Config**: `/opt/siemzello/SIEMZello-api/.env`
- **Frontend Config**: `/opt/siemzello/SIEMZello-ui/siemzello-ui/.env.local`
- **Agent Config**: `/opt/siemzello-agent/kafka.conf` (on each agent)

### **Database Files**
- **Frontend DB**: `/opt/siemzello/SIEMZello-ui/siemzello-ui/data/agents.db`
- **API DB**: `/opt/siemzello/SIEMZello-api/data/agents.db`

---

*This documentation covers the complete SIEMZello system deployment and configuration. Always update the SIEM VM IP in agent.yml before deploying the setup.zip to ensure proper connectivity between all components.*
