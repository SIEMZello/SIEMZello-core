# SIEMZello Project Report
## AI-Powered Security Information and Event Management System

*Student Project Report - Final Submission*

---

## 📋 Project Overview

**SIEMZello** is a comprehensive Security Information and Event Management (SIEM) system that leverages artificial intelligence for advanced threat detection and security monitoring. The project consists of multiple interconnected components working together to provide real-time security analysis, monitoring, and response capabilities.

### 🎯 Project Objectives
- Develop an AI-powered threat detection system
- Create a real-time security monitoring dashboard
- Implement automated log analysis and anomaly detection
- Build a scalable SIEM architecture with modern web technologies
- Provide comprehensive security insights through data visualization

---

## 🏗️ System Architecture

The SIEMZello system follows a microservices architecture with the following major components:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   SIEMZello-ui  │    │  SIEMZello-api  │    │  SIEMZello-ai   │
│   (Frontend)    │◄──►│   (Backend)     │◄──►│  (AI Engine)    │
│                 │    │                 │    │                 │
│ • Next.js       │    │ • FastAPI       │    │ • ML Models     │
│ • React         │    │ • SQLite        │    │ • Anomaly       │
│ • TailwindCSS   │    │ • REST APIs     │    │   Detection     │
│ • TypeScript    │    │ • Kafka         │    │ • Classification│
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                        │                        │
        └────────────────────────┼────────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  SIEMZello-core │
                    │ (Documentation) │
                    │                 │
                    │ • Architecture  │
                    │ • API Docs      │
                    │ • Setup Guides  │
                    └─────────────────┘
```

---

## 🔧 Major Components Developed

### 1. **SIEMZello-ui (Frontend Dashboard)**

**Technology Stack:**
- **Next.js 15.2.4** - React framework with App Router
- **React 19** - UI library with modern hooks
- **TypeScript** - Type-safe development
- **TailwindCSS** - Utility-first styling
- **shadcn/ui** - Beautiful component library
- **Recharts** - Data visualization
- **SQLite (better-sqlite3)** - Local database

**Key Features Implemented:**
- **Real-time Agent Monitoring Dashboard**
  - Live CPU, memory, disk, and network metrics
  - Auto-refresh every 15 seconds
  - Agent connectivity testing
  - System overview with aggregated statistics

- **Agent Management Interface**
  - Add, edit, delete monitoring agents
  - Agent status tracking (online/offline)
  - SSH connectivity validation
  - Agent statistics and health monitoring

- **Security Analytics Dashboard**
  - Threat detection visualization
  - Alert management system
  - Event explorer with filtering
  - Security trends analysis

- **Comprehensive Admin Interface**
  - User management
  - System settings
  - Database management
  - Report generation

**Database Schema:**
```sql
-- Agents table for monitored systems
CREATE TABLE agents (
    id INTEGER PRIMARY KEY,
    hostname TEXT UNIQUE,
    ip TEXT UNIQUE, 
    mac TEXT UNIQUE,
    ssh_user TEXT,
    ssh_pass TEXT,
    sudo_pass TEXT,
    interface TEXT,
    status TEXT DEFAULT 'active',
    created_at DATETIME,
    updated_at DATETIME
);

-- Logs table for security events
CREATE TABLE logs (
    id INTEGER PRIMARY KEY,
    agent_id INTEGER,
    category TEXT,
    attack_probability REAL,
    attack_type TEXT,
    full_message TEXT,
    created_at DATETIME,
    FOREIGN KEY (agent_id) REFERENCES agents(id)
);
```

### 2. **SIEMZello-api (Backend API Server)**

**Technology Stack:**
- **FastAPI** - Modern Python web framework
- **SQLite** - Database with Python integration
- **Kafka** - Message streaming for log processing
- **Pydantic** - Data validation and serialization
- **CORS middleware** - Cross-origin resource sharing

**API Endpoints Implemented:**
- **Agent Management APIs**
  - `GET /api/agents/` - List all agents
  - `POST /api/agents/` - Create new agent
  - `PUT /api/agents/{id}` - Update agent
  - `DELETE /api/agents/{id}` - Delete agent
  - `GET /api/agents/stats/overview` - Agent statistics

- **Logs Management APIs**
  - `GET /api/agents/logs` - Get all logs with pagination
  - `GET /api/agents/{id}/logs` - Get logs for specific agent
  - `GET /api/agents/logs/count` - Get total logs count

- **Authentication APIs**
  - `POST /auth/login` - User authentication
  - User session management

- **Dashboard APIs**
  - `GET /dashboard/overview` - Dashboard statistics
  - Real-time metrics endpoints

**Key Features:**
- **Automated Agent Deployment**
  - Ansible playbook execution for agent setup
  - SSH key deployment and configuration
  - Remote system configuration

- **Real-time Data Processing**
  - Kafka consumers for log processing
  - Automated threat analysis pipeline
  - Attack probability calculation

- **Database Operations**
  - CRUD operations for all entities
  - Data validation and error handling
  - Mock data generation for development

### 3. **SIEMZello-ai (AI Engine)**

**Technology Stack:**
- **FastAPI** - AI service API
- **Scikit-learn** - Machine learning models
- **Pandas** - Data processing
- **NumPy** - Numerical computations
- **Custom ML Models** - Domain-specific analyzers

**AI Models Developed:**

#### **Network Traffic Analyzer**
- **Two-stage detection system:**
  1. **Detection Model**: Binary classification (normal vs. attack)
  2. **Classification Model**: Multi-class attack type identification
- **Features**: Protocol analysis, traffic pattern detection, anomaly scoring

#### **Memory Analysis Engine**
- **Anomaly detection** for memory usage patterns
- **Process behavior analysis**
- **Memory leak detection**
- **Resource consumption monitoring**

#### **Disk Activity Analyzer**
- **I/O pattern analysis**
- **Suspicious disk activity detection**
- **File system monitoring**
- **Performance anomaly detection**

#### **Process Monitoring System**
- **Process behavior analysis**
- **CPU usage anomaly detection**
- **Privilege escalation detection**
- **Malicious process identification**

**API Endpoints:**
```python
# Network traffic analysis
POST /api/v1/predict/network

# System resource analysis
POST /api/v1/predict/memory
POST /api/v1/predict/disk  
POST /api/v1/predict/process

# Integrated SIEM analysis
POST /api/v1/siem/analyze
```

### 4. **Data Consumers & Processing Pipeline**

**Kafka Integration:**
- **ProcessConsumer**: Analyzes system process logs
- **NetworkConsumer**: Handles network traffic data
- **MemConsumer**: Processes memory usage logs
- **DiskConsumer**: Monitors disk activity

**Processing Workflow:**
1. **Log Ingestion**: Agents send logs to Kafka topics
2. **AI Analysis**: Consumers forward data to AI models
3. **Threat Scoring**: AI returns attack probability scores
4. **Database Storage**: Results stored with enriched metadata
5. **Real-time Alerts**: High-risk events trigger notifications

### 5. **Real-time Monitoring System**

**Agent Metrics Collection:**
- **Live system metrics** from monitored agents
- **HTTP Basic Auth** security (siem:pass)
- **Metrics endpoint**: `{agent_ip}:5050/metrics`
- **Data structure**:
  ```typescript
  interface AgentMetrics {
    timestamp: string;
    cpu: { total_usage: number };
    memory: { percent: number; total: number; used: number };
    disk: { percent: number; read_bytes: number; write_bytes: number };
    network: { bytes_sent: number; bytes_recv: number };
  }
  ```

**Real-time Features:**
- **15-second auto-refresh** of all metrics
- **Agent connectivity monitoring**
- **System health aggregation**
- **Performance trend analysis**

---

## 🎨 User Interface Design

### **Modern Dashboard Interface**
- **Dark/Light theme support** with system preference detection
- **Responsive design** for desktop and mobile devices
- **Interactive charts** using Recharts library
- **Real-time updates** without page refresh

### **Key Dashboard Sections**
1. **Overview Tab**: System-wide statistics and health metrics
2. **Servers Tab**: Individual agent monitoring and management
3. **Analytics Tab**: Security trends and threat analysis
4. **Alerts Tab**: Active security alerts and incidents
5. **Reports Tab**: Security reports and compliance data

### **Component Library**
- **shadcn/ui components**: Professional, accessible UI components
- **Custom charts**: CPU, memory, disk, and network visualizations
- **Data tables**: Sortable, filterable agent and log listings
- **Form components**: Agent creation and configuration forms

---

## 🗄️ Database Design

### **SQLite Database Schema**
The system uses SQLite for simplicity and portability:

```sql
-- Main agents table
CREATE TABLE agents (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    hostname TEXT NOT NULL UNIQUE,
    ssh_user TEXT NOT NULL,
    ip TEXT NOT NULL UNIQUE,
    mac TEXT NOT NULL UNIQUE,
    status TEXT NOT NULL DEFAULT 'active',
    ssh_pass TEXT NOT NULL,
    sudo_pass TEXT NOT NULL,
    interface TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Security logs table
CREATE TABLE logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    agent_id INTEGER NOT NULL,
    category TEXT NOT NULL,
    attack_probability REAL NOT NULL,
    attack_type TEXT NOT NULL,
    full_message TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (agent_id) REFERENCES agents(id) ON DELETE CASCADE
);

-- Metrics history table
CREATE TABLE metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    agent_id INTEGER NOT NULL,
    cpu_total_usage REAL NOT NULL,
    disk_usage REAL NOT NULL,
    memory_usage REAL NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (agent_id) REFERENCES agents(id) ON DELETE CASCADE
);
```

---

## 🔐 Security Features

### **Authentication & Authorization**
- **JWT-based authentication** for API access
- **Role-based access control** (RBAC)
- **Secure password handling** with bcrypt hashing
- **Session management** with token expiration

### **Network Security**
- **SSH key-based authentication** for agent communication
- **Basic HTTP Authentication** for metrics endpoints
- **CORS configuration** for cross-origin security
- **Input validation** and sanitization

### **Data Protection**
- **SQL injection prevention** through parameterized queries
- **XSS protection** through proper data escaping
- **Sensitive data encryption** for stored credentials
- **Audit logging** for all administrative actions

---

## 🚀 Deployment & Configuration

### **Development Environment Setup**

**Frontend (SIEMZello-ui):**
```bash
cd SIEMZello-ui/siemzello-ui
pnpm install
pnpm dev  # Runs on http://localhost:3000
```

**Backend API (SIEMZello-api):**
```bash
cd SIEMZello-api
pip install -r requirements.txt
uvicorn app.main:app --reload  # Runs on http://localhost:8000
```

**AI Engine (SIEMZello-ai):**
```bash
cd SIEMZello-ai
pip install -r requirements.txt
uvicorn main:app --reload --port 5000 # Runs on http://localhost:5000
```

### **Production Considerations**
- **Docker containerization** for easy deployment
- **Kubernetes orchestration** for scalability
- **Load balancing** for high availability
- **Database clustering** for performance
- **SSL/TLS encryption** for secure communications

---

## 📊 Performance & Scalability

### **System Performance**
- **Real-time processing** with sub-second response times
- **Concurrent user support** through async operations
- **Efficient database queries** with proper indexing
- **Memory optimization** in ML model inference

### **Scalability Features**
- **Microservices architecture** for horizontal scaling
- **Kafka message queues** for reliable data processing
- **Database sharding** capabilities
- **CDN integration** for static asset delivery

---

## 🧪 Testing & Quality Assurance

### **Testing Implementation**
- **Unit tests** for individual components
- **Integration tests** for API endpoints
- **End-to-end tests** for user workflows
- **Performance testing** for load handling

### **Code Quality**
- **TypeScript** for type safety in frontend
- **Pydantic validation** for API data validation
- **ESLint & Prettier** for code formatting
- **Error handling** and logging throughout

---

## 📈 Monitoring & Analytics

### **System Monitoring**
- **Real-time agent health monitoring**
- **Performance metrics collection**
- **Error rate tracking**
- **Resource utilization monitoring**

### **Security Analytics**
- **Attack pattern recognition**
- **Threat trend analysis**
- **Compliance reporting**
- **Incident response metrics**

---

## 🎓 Learning Outcomes & Technical Skills

### **Technologies Mastered**
- **Modern Web Development**: Next.js, React, TypeScript
- **Backend Development**: FastAPI, Python, REST APIs
- **Database Management**: SQLite, SQL queries, schema design
- **Machine Learning**: Scikit-learn, anomaly detection, classification
- **DevOps**: Docker, message queues, microservices
- **Security**: Authentication, authorization, secure communications

### **Software Engineering Practices**
- **Version Control**: Git workflow and collaboration
- **API Design**: RESTful services and documentation
- **Code Organization**: Modular architecture and clean code
- **Error Handling**: Robust error management and logging
- **Documentation**: Comprehensive project documentation

---

## 🔮 Future Enhancements

### **Planned Improvements**
1. **Enhanced AI Models**: Deep learning for advanced threat detection
2. **Real-time Alerting**: SMS, email, and Slack notifications
3. **Historical Analytics**: Long-term trend analysis and reporting
4. **Mobile Application**: iOS and Android apps for monitoring
5. **Integration APIs**: Third-party security tool integration
6. **Compliance Features**: GDPR, HIPAA, and SOX compliance modules

### **Scalability Roadmap**
- **Cloud deployment** on AWS/Azure/GCP
- **Kubernetes orchestration** for container management
- **Elasticsearch integration** for log search and analytics
- **GraphQL APIs** for efficient data fetching
- **WebSocket connections** for real-time updates

---

## 💡 Challenges & Solutions

### **Technical Challenges Overcome**
1. **Real-time Data Synchronization**: Solved with auto-refresh hooks and efficient state management
2. **Cross-platform Compatibility**: Addressed through responsive design and progressive enhancement
3. **Performance Optimization**: Implemented through efficient queries and caching strategies
4. **Security Implementation**: Ensured through comprehensive authentication and data validation

### **Development Challenges**
- **Complex State Management**: Resolved with React hooks and context
- **API Integration**: Handled through robust error handling and retry logic
- **Data Visualization**: Implemented with Recharts and custom components
- **Deployment Coordination**: Managed through clear documentation and scripts

---

## 📚 Documentation & Resources

### **Project Documentation**
- **API Documentation**: Comprehensive endpoint documentation
- **Architecture Guide**: System design and component interaction
- **Setup Instructions**: Step-by-step installation guide
- **User Manual**: End-user operation instructions

### **Code Organization**
```
SIEMZello/
├── SIEMZello-ui/          # Frontend application
│   ├── app/               # Next.js app router pages
│   ├── components/        # Reusable UI components
│   ├── lib/              # Utilities and API clients
│   └── hooks/            # Custom React hooks
├── SIEMZello-api/         # Backend API server
│   ├── app/              # FastAPI application
│   ├── core/             # Core functionality
│   └── models/           # Data models and schemas
├── SIEMZello-ai/          # AI engine service
│   ├── api/              # AI API endpoints
│   ├── models/           # ML model implementations
│   └── schemas/          # Data validation schemas
└── SIEMZello-core/        # Documentation and guides
    └── docs/             # Project documentation
```

---

## 🏆 Project Summary

The **SIEMZello** project represents a comprehensive Security Information and Event Management system that successfully integrates modern web technologies with artificial intelligence for advanced threat detection. The system demonstrates proficiency in:

- **Full-stack development** with modern frameworks
- **AI/ML implementation** for security analytics
- **Database design** and management
- **API development** and integration
- **Real-time systems** and monitoring
- **Security best practices** and implementation

The project showcases the ability to design, develop, and deploy a complex, multi-component system that addresses real-world cybersecurity challenges while maintaining high standards of code quality, performance, and user experience.

---

*This project was developed as part of the computer science curriculum, demonstrating advanced technical skills in software engineering, cybersecurity, and artificial intelligence.*

**Project Status**: ✅ **COMPLETED - Early Version Ready for Deployment**

**Development Period**: Academic Year 2024-2025

**Technologies Used**: Next.js, React, TypeScript, FastAPI, Python, SQLite, Kafka, Scikit-learn, TailwindCSS, shadcn/ui
