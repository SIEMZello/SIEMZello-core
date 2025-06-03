# SIEMZello API Documentation

**Version:** 1.0.0  
**Description:** AI-Powered Threat Detection SIEM System API  
**Base URL:** `http://localhost:8000` (or your configured host)

---

## 📋 Table of Contents

1. [Getting Started](#getting-started)
2. [API Architecture](#api-architecture)
3. [Authentication](#authentication)
4. [Agents Management](#agents-management)
5. [Users Management](#users-management)
6. [Events & Logs](#events--logs)
7. [Alerts System](#alerts-system)
8. [Network Monitoring](#network-monitoring)
9. [Server Monitoring](#server-monitoring)
10. [Reports Generation](#reports-generation)
11. [Dashboard Endpoints](#dashboard-endpoints)
12. [Error Handling](#error-handling)
13. [Frontend Integration](#frontend-integration)
14. [Testing Guide](#testing-guide)
15. [Troubleshooting](#troubleshooting)
16. [Development Workflow](#development-workflow)
17. [Performance Considerations](#performance-considerations)

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- FastAPI application running
- Virtual environment activated

### Quick Start
```bash
# Start the API server
cd SIEMZello-api
.\venv\Scripts\Activate.ps1
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Access API documentation
http://localhost:8000/docs
```

### Health Check
```http
GET /health
```
**Response:**
```json
{
  "status": "ok"
}
```

---

## 🏗️ API Architecture

### Router Structure
| Router | Prefix | Description |
|--------|--------|-------------|
| `auth` | `/auth` | Authentication and login |
| `users` | `/api/users` | User management |
| `agents` | `/api/agents` | Monitoring agents |
| `events` | `/api/events` | Security events |
| `reports` | `/api/reports` | Report generation |
| `alerts` | `/alerts` | Alert management |
| `dashboard` | `/dashboard` | Dashboard data |
| `servers` | `/api/servers` | Server monitoring |
| `network` | `/api/network` | Network monitoring |

### Key Features
- **RESTful Design**: Standard HTTP methods (GET, POST, PUT, DELETE, PATCH)
- **Input Validation**: Pydantic schemas for request/response validation
- **Error Handling**: Comprehensive error responses with HTTP status codes
- **CORS Enabled**: Cross-origin requests supported for frontend integration
- **Mock Data**: Development-ready with mock data (replace with database in production)

---

## 🔐 Authentication

> **Note:** Authentication endpoints are available but not fully implemented in current version.

### Login
```http
POST /auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password123"
}
```

---

## 🤖 Agents Management

Monitoring agents are systems/servers that SIEMZello monitors for security events.

### Agent Data Model
```json
{
  "id": 1,
  "hostname": "web-server-01",
  "ip": "192.168.1.10",
  "mac": "00:11:22:33:44:55",
  "status": "active",
  "ssh_pass": "secure123",
  "sudo_pass": "sudo123",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### Get All Agents
```http
GET /api/agents/
```
**Response:** Array of agent objects

### Get Agent by ID
```http
GET /api/agents/{agent_id}
```
**Example:** `GET /api/agents/1`

### Create New Agent
```http
POST /api/agents/
Content-Type: application/json

{
  "hostname": "new-server-01",
  "ip": "192.168.1.20",
  "mac": "00:11:22:33:44:66",
  "status": "active",
  "ssh_pass": "secure123",
  "sudo_pass": "sudo123"
}
```
**Validation Rules:**
- `hostname`: Required, 1-255 characters
- `ip`: Required, valid IPv4 format
- `mac`: Required, valid MAC address format (XX:XX:XX:XX:XX:XX)
- `status`: Either "active" or "inactive"

### Update Agent
```http
PUT /api/agents/{agent_id}
Content-Type: application/json

{
  "hostname": "updated-server-01",
  "status": "inactive"
}
```
**Note:** All fields are optional in updates

### Delete Agent
```http
DELETE /api/agents/{agent_id}
```
**Response:**
```json
{
  "success": true,
  "message": "Agent 'web-server-01' deleted successfully"
}
```

### Update Agent Status
```http
PATCH /api/agents/{agent_id}/status?status=inactive
```

### Agent Statistics
```http
GET /api/agents/stats/overview
```
**Response:**
```json
{
  "total": 5,
  "active": 3,
  "inactive": 2
}
```

### Check if Agents Exist
```http
GET /api/agents/stats/exists
```
**Response:**
```json
{
  "has_agents": true
}
```

### Error Handling
- **400 Bad Request**: Duplicate hostname/IP, invalid data format
- **404 Not Found**: Agent ID doesn't exist
- **422 Unprocessable Entity**: Invalid input data

---

## 👥 Users Management

### User Data Model
```json
{
  "id": "user-001",
  "username": "admin",
  "name": "Admin User",
  "email": "admin@example.com",
  "role": "Administrator",
  "status": "active",
  "last_login": "2024-01-15T10:30:00Z",
  "created_at": "2024-01-15T08:00:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### User Roles
- `Administrator`: Full system access
- `Security Analyst`: Security monitoring and analysis
- `System Administrator`: Infrastructure management

### Key Endpoints
```http
GET /api/users/                    # List all users
POST /api/users/                   # Create new user
GET /api/users/{user_id}           # Get user details
PUT /api/users/{user_id}           # Update user
DELETE /api/users/{user_id}        # Delete user
GET /api/users/stats/overview      # User statistics
GET /api/users/profile/me          # Current user profile
GET /api/users/auth-logs           # Authentication logs
```

---

## 📊 Events & Logs

Security events are the core data that SIEMZello processes and analyzes.

### Event Data Model
```json
{
  "id": "event-001",
  "timestamp": "2024-01-15T10:30:00Z",
  "event_type": "authentication",
  "severity": "medium",
  "source": "web-server-01",
  "description": "Failed login attempt",
  "details": {
    "username": "admin",
    "source_ip": "192.168.1.100",
    "failure_reason": "invalid_password"
  }
}
```

### Event Types
- `authentication`: Login/logout events
- `network`: Network traffic anomalies
- `system`: System configuration changes
- `security`: Security policy violations
- `application`: Application-specific events

### Severity Levels
- `low`: Informational events
- `medium`: Events requiring attention
- `high`: Potential security incidents
- `critical`: Active security threats

### Key Endpoints
```http
GET /api/events/                   # Get events with filtering
GET /api/events/{event_id}         # Get specific event
GET /api/events/search             # Advanced event search
GET /api/events/stats              # Event statistics
GET /api/events/export             # Export events
```

### Event Filtering
```http
GET /api/events/?event_type=authentication&severity=high&page=1&limit=20
```

### Search Events
```http
GET /api/events/search?query=failed+login&page=1&limit=10
```

---

## 🚨 Alerts System

Alerts are triggered when suspicious activities or security events are detected.

### Alert Data Model
```json
{
  "id": "alert-001",
  "title": "Multiple Failed Login Attempts",
  "source": "Authentication System",
  "description": "User 'admin' failed login 5 times in 10 minutes",
  "priority": "high",
  "status": "active",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### Alert Priorities
- `low`: Minor security events
- `medium`: Moderate threats
- `high`: Serious security incidents
- `critical`: Immediate action required

### Alert Status
- `active`: Alert needs attention
- `acknowledged`: Alert has been seen
- `resolved`: Alert has been handled
- `false_positive`: Alert was not a real threat

### Key Endpoints
```http
GET /alerts/                       # Get all alerts
GET /alerts/active                 # Get active alerts
GET /alerts/priority/{priority}    # Get alerts by priority
POST /alerts/{alert_id}/acknowledge # Acknowledge alert
POST /alerts/{alert_id}/resolve    # Mark alert as resolved
```

---

## 🌐 Network Monitoring

Monitor network traffic, connections, and detect network-based security threats.

### Network Traffic Stats
```http
GET /api/network/traffic/stats
```
**Response:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "bandwidth_utilization": 45.2,
  "total_bytes_in": 1048576000,
  "total_bytes_out": 524288000,
  "total_packets_in": 1000000,
  "total_packets_out": 800000,
  "protocol_breakdown": {
    "HTTP": 1200,
    "HTTPS": 800,
    "SSH": 150,
    "FTP": 50
  }
}
```

### Network Connections
```http
GET /api/network/connections?page=1&per_page=20
```

### Network Alerts
```http
GET /api/network/alerts?severity=high&hours=24
```

### Real-time Network Data
```http
GET /api/network/realtime
```

### Key Network Endpoints
```http
GET /api/network/traffic/stats      # Traffic statistics
GET /api/network/traffic/history    # Historical traffic data  
GET /api/network/traffic/protocols  # Protocol breakdown
GET /api/network/connections        # Active connections
GET /api/network/devices            # Network devices
GET /api/network/alerts             # Network security alerts
GET /api/network/security/overview  # Security overview
```

---

## 🖥️ Server Monitoring

Monitor server health, performance metrics, and system status.

### Server Data Model
```json
{
  "id": "srv-001",
  "hostname": "web-server-01",
  "ip_address": "192.168.1.10",
  "status": "online",
  "os_type": "Ubuntu 22.04 LTS",
  "cpu_cores": 4,
  "memory_total": 16384,
  "disk_total": 500,
  "uptime": 86400,
  "last_seen": "2024-01-15T10:30:00Z"
}
```

### Server Status Values
- `online`: Server is running normally
- `offline`: Server is not responding
- `maintenance`: Server is under maintenance
- `error`: Server has issues

### Server Metrics
```http
GET /api/servers/{server_id}/metrics
```
**Response:**
```json
{
  "server_id": "srv-001",
  "hostname": "web-server-01",
  "timestamp": "2024-01-15T10:30:00Z",
  "cpu_usage": 45.2,
  "memory_usage": 62.8,
  "disk_usage": 78.5,
  "network_in": 1024000,
  "network_out": 512000,
  "load_average": 1.25,
  "process_count": 180
}
```

### Key Server Endpoints
```http
GET /api/servers/                  # List all servers
GET /api/servers/{server_id}       # Get server details
GET /api/servers/{server_id}/metrics # Server performance metrics
GET /api/servers/{server_id}/health  # Server health status
GET /api/servers/stats             # Server statistics
GET /api/servers/health/overview   # Health overview
GET /api/servers/metrics/realtime  # Real-time metrics
```

---

## 📄 Reports Generation

Generate security reports in various formats for compliance and analysis.

### Report Types
- `security_summary`: Comprehensive security overview
- `auth_failures`: Authentication failure analysis
- `compliance`: Compliance status report
- `network_activity`: Network traffic analysis

### Generate Report
```http
POST /api/reports/generate
Content-Type: application/json

{
  "title": "Weekly Security Report",
  "description": "Security summary for the past week",
  "report_type": "security_summary",
  "format": "pdf",
  "date_range": "7d"
}
```

### Report Status
- `pending`: Report request received
- `generating`: Report is being created
- `completed`: Report is ready for download
- `failed`: Report generation failed

### Key Report Endpoints
```http
GET /api/reports/                  # List all reports
POST /api/reports/generate         # Generate new report
GET /api/reports/{report_id}       # Get report details
DELETE /api/reports/{report_id}    # Delete report
GET /api/reports/download/{report_id} # Download report
GET /api/reports/templates/        # Available templates
GET /api/reports/stats/overview    # Report statistics
```

---

## 📈 Dashboard Endpoints

Provide aggregated data for dashboard visualizations.

### Dashboard Data Example
```http
GET /dashboard/overview
```
**Response:**
```json
{
  "alerts": {
    "total": 25,
    "critical": 3,
    "high": 8,
    "medium": 10,
    "low": 4
  },
  "agents": {
    "total": 12,
    "active": 10,
    "inactive": 2
  },
  "events_24h": 1847,
  "threats_detected": 5,
  "uptime_percentage": 99.2
}
```

---

## ⚠️ Error Handling

### HTTP Status Codes
- **200 OK**: Successful request
- **201 Created**: Resource created successfully
- **400 Bad Request**: Invalid input data
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource not found
- **422 Unprocessable Entity**: Validation error
- **500 Internal Server Error**: Server error

### Error Response Format
```json
{
  "detail": "Agent with hostname 'web-server-01' already exists"
}
```

### Validation Errors
```json
{
  "detail": [
    {
      "loc": ["body", "ip"],
      "msg": "Invalid IP address format",
      "type": "value_error"
    }
  ]
}
```

---

## 🔗 Frontend Integration

### API Client Configuration
The frontend automatically detects the correct API base URL:

```typescript
// Smart API client that adapts to your network setup
const getApiBaseUrl = () => {
  if (typeof window !== 'undefined') {
    const currentHost = window.location.hostname;
    if (currentHost === 'localhost') {
      return 'http://localhost:8000';
    }
    return `http://${currentHost}:8000`;
  }
  return 'http://localhost:8000';
};
```

### Network Access
- **Local Development**: `http://localhost:8000`
- **Network Access**: `http://192.168.100.16:8000`
- **API Documentation**: `http://192.168.100.16:8000/docs`

### CORS Configuration
The API accepts requests from any origin for development:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 🧪 Testing Guide

### API Testing Tools

#### 1. Interactive API Documentation
Visit the auto-generated FastAPI docs:
```
http://localhost:8000/docs        # Swagger UI
http://localhost:8000/redoc       # ReDoc
```

#### 2. Built-in Test Page
Use the frontend test page for comprehensive API testing:
```
http://localhost:3000/test/api
```

This page provides:
- Real-time API connectivity testing
- Current API base URL display
- Interactive endpoint testing
- Network configuration validation

#### 3. PowerShell Testing Scripts

**Test Agent Endpoints:**
```powershell
# Get all agents
$response = Invoke-RestMethod -Uri "http://localhost:8000/api/agents/" -Method GET
$response | ConvertTo-Json -Depth 3

# Create new agent
$agentData = @{
    hostname = "test-server-$(Get-Random)"
    ip = "192.168.1.$(Get-Random -Minimum 10 -Maximum 254)"
    mac = "00:11:22:33:44:55"
    status = "active"
    ssh_pass = "testpass123"
    sudo_pass = "testpass123"
} | ConvertTo-Json

$newAgent = Invoke-RestMethod -Uri "http://localhost:8000/api/agents/" -Method POST -Body $agentData -ContentType "application/json"
Write-Host "Created agent with ID: $($newAgent.id)"
```

**Test Network Endpoints:**
```powershell
# Get network statistics
$networkStats = Invoke-RestMethod -Uri "http://localhost:8000/api/network/stats" -Method GET
$networkStats | ConvertTo-Json -Depth 2

# Get network connections
$connections = Invoke-RestMethod -Uri "http://localhost:8000/api/network/connections" -Method GET
Write-Host "Found $($connections.total) network connections"
```

#### 4. Automated Testing with curl

**Health Check Script:**
```bash
#!/bin/bash
# Health check for all major endpoints
endpoints=(
    "/health"
    "/api/agents/stats/overview"
    "/api/users/stats"
    "/api/network/stats"
    "/api/servers/stats"
    "/dashboard/overview"
)

for endpoint in "${endpoints[@]}"; do
    echo "Testing $endpoint..."
    response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8000$endpoint)
    if [ $response -eq 200 ]; then
        echo "✓ $endpoint - OK"
    else
        echo "✗ $endpoint - Failed ($response)"
    fi
done
```

### Testing Different Network Configurations

#### Local Development
```bash
# Backend on localhost
http://localhost:8000/docs

# Frontend on localhost
http://localhost:3000/test/api
```

#### Network Testing
```bash
# Replace with your actual IP
export LOCAL_IP="192.168.100.16"

# Backend accessible from network
http://$LOCAL_IP:8000/docs

# Frontend accessible from network
http://$LOCAL_IP:3000/test/api
```

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### 1. "Failed to fetch" Errors

**Symptoms:**
- Frontend can't connect to backend
- CORS errors in browser console
- Network timeout errors

**Solutions:**
```powershell
# Check if backend is running
netstat -an | findstr :8000

# Test backend health
curl http://localhost:8000/health

# Check CORS configuration in app/main.py
# Ensure allow_origins includes your frontend URL
```

#### 2. Network Access Issues

**Problem:** Can't access API from other devices on network

**Solution:**
```powershell
# Start backend on all interfaces
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Start frontend on all interfaces
npm run dev -- --host 0.0.0.0 --port 3000
```

#### 3. Smart URL Detection Not Working

**Problem:** Frontend uses wrong API base URL

**Debug Steps:**
1. Check current detection logic:
```javascript
console.log('Current hostname:', window.location.hostname);
console.log('Detected API URL:', getApiBaseUrl());
```

2. Override detection for testing:
```javascript
localStorage.setItem('API_BASE_URL_OVERRIDE', 'http://192.168.1.100:8000');
```

#### 4. Mock Data Issues

**Problem:** Endpoints return outdated or inconsistent mock data

**Solution:**
```python
# Restart the FastAPI server to reset mock data
# Or implement data persistence with SQLite for development:

# In app/core/database.py
import sqlite3
from datetime import datetime

def init_db():
    conn = sqlite3.connect('dev_database.db')
    # Create tables for persistent mock data
    conn.execute('''
        CREATE TABLE IF NOT EXISTS agents (
            id INTEGER PRIMARY KEY,
            hostname TEXT,
            ip TEXT,
            status TEXT,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    conn.commit()
    conn.close()
```
