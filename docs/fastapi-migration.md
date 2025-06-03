# FastAPI Backend Migration - Complete Implementation Guide

## Overview

This document describes the complete migration of the SIEMZello agent management system from Next.js API routes to a FastAPI backend, including the implementation of conditional dashboard access based on agent configuration.

## Architecture Changes

### Before: Next.js API Routes
- API endpoints: `/api/agents/`, `/api/agents/stats`
- Client-side database operations using SQLite
- Direct fetch calls from React components
- Boolean status field for agents

### After: FastAPI Backend
- API endpoints: `http://localhost:8000/api/agents/`, `http://localhost:8000/api/agents/stats`
- Server-side database operations with service layer
- Centralized API client with error handling
- Enum status field ("active" | "inactive")
- Conditional dashboard access based on agent count

## Implementation Details

### 1. FastAPI Backend Components

#### Agent Schemas (`/app/models/schemas.py`)
```python
from pydantic import BaseModel
from typing import Optional
from enum import Enum
from datetime import datetime

class AgentStatus(str, Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"

class AgentBase(BaseModel):
    hostname: str
    ip: str
    mac: str
    status: AgentStatus
    ssh_pass: str
    sudo_pass: str

class AgentCreate(AgentBase):
    pass

class AgentUpdate(BaseModel):
    hostname: Optional[str] = None
    ip: Optional[str] = None
    mac: Optional[str] = None
    status: Optional[AgentStatus] = None
    ssh_pass: Optional[str] = None
    sudo_pass: Optional[str] = None

class Agent(AgentBase):
    id: int
    created_at: datetime
    updated_at: datetime

class AgentStats(BaseModel):
    total: int
    active: int
    inactive: int
```

#### Database Service (`/app/services/agents.py`)
- SQLite database operations
- Connection management with proper closing
- CRUD operations (Create, Read, Update, Delete)
- Statistics calculation
- Error handling and validation

#### API Routes (`/app/api/agents.py`)
```python
from fastapi import APIRouter, HTTPException, Depends
from typing import List
from ..models.schemas import Agent, AgentCreate, AgentUpdate, AgentStats
from ..services.agents import AgentService

router = APIRouter(prefix="/api/agents", tags=["agents"])

@router.get("/stats", response_model=AgentStats)
async def get_agent_stats():
    return AgentService.get_stats()

@router.get("/", response_model=List[Agent])
async def get_agents():
    return AgentService.get_all()

@router.post("/", response_model=Agent)
async def create_agent(agent: AgentCreate):
    return AgentService.create(agent)

@router.get("/{agent_id}", response_model=Agent)
async def get_agent(agent_id: int):
    agent = AgentService.get_by_id(agent_id)
    if not agent:
        raise HTTPException(status_code=404, detail="Agent not found")
    return agent

@router.put("/{agent_id}", response_model=Agent)
async def update_agent(agent_id: int, agent_update: AgentUpdate):
    agent = AgentService.update(agent_id, agent_update)
    if not agent:
        raise HTTPException(status_code=404, detail="Agent not found")
    return agent

@router.delete("/{agent_id}")
async def delete_agent(agent_id: int):
    success = AgentService.delete(agent_id)
    if not success:
        raise HTTPException(status_code=404, detail="Agent not found")
    return {"message": "Agent deleted successfully"}
```

### 2. Frontend API Client

#### Centralized HTTP Client (`/lib/api-client.ts`)
```typescript
class ApiClient {
  private baseURL: string;

  constructor(baseURL: string = 'http://localhost:8000') {
    this.baseURL = baseURL;
  }

  private async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
      ...options,
    };

    try {
      const response = await fetch(url, config);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      return await response.json();
    } catch (error) {
      console.error(`API request failed: ${error}`);
      throw error;
    }
  }
}

export const agentApi = {
  getAll: (): Promise<Agent[]> => apiClient.get('/api/agents/'),
  getById: (id: number): Promise<Agent> => apiClient.get(`/api/agents/${id}`),
  create: (agent: AgentCreate): Promise<Agent> => apiClient.post('/api/agents/', agent),
  update: (id: number, agent: AgentUpdate): Promise<Agent> => 
    apiClient.put(`/api/agents/${id}`, agent),
  delete: (id: number): Promise<void> => apiClient.delete(`/api/agents/${id}`),
  getStats: (): Promise<AgentStats> => apiClient.get('/api/agents/stats'),
};
```

### 3. Dashboard Access Control

#### Agents Guard Hook (`/hooks/use-agents-guard.ts`)
```typescript
export function useAgentsGuard(): UseAgentsGuardReturn {
  const [hasAgents, setHasAgents] = useState(false);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const checkAgents = async () => {
    try {
      setLoading(true);
      setError(null);
      const stats = await agentApi.getStats();
      setHasAgents(stats.total > 0);
    } catch (err) {
      console.error('Error checking agents:', err);
      setError('Failed to check agent status');
      setHasAgents(false);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    checkAgents();
  }, []);

  return { hasAgents, loading, error, refetch: checkAgents };
}
```

#### Dashboard Layout with Conditional Access (`/app/dashboard/layout.tsx`)
```typescript
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  const { hasAgents, loading, error } = useAgentsGuard();

  // Show loading state while checking for agents
  if (loading) {
    return <LoadingSpinner />;
  }

  // Show error state if failed to check agents
  if (error) {
    return <ErrorState error={error} />;
  }

  // Show no agents state if no agents are configured
  if (!hasAgents) {
    return <NoAgentsState />;
  }

  // Show full dashboard with sidebar if agents exist
  return (
    <SidebarProvider>
      <div className="flex min-h-screen w-full">
        <DashboardSidebar />
        <main className="flex-1 flex flex-col w-0 min-w-0">
          <DashboardHeader />
          {children}
        </main>
      </div>
    </SidebarProvider>
  );
}
```

## Key Features Implemented

### 1. Agent Management
- ✅ **Create Agents**: POST `/api/agents/` with full validation
- ✅ **Read Agents**: GET `/api/agents/` and GET `/api/agents/{id}`
- ✅ **Update Agents**: PUT `/api/agents/{id}` with partial updates
- ✅ **Delete Agents**: DELETE `/api/agents/{id}` with confirmation
- ✅ **Agent Statistics**: GET `/api/agents/stats` for dashboard metrics

### 2. Data Validation
- ✅ **Pydantic Validation**: Server-side validation for all agent operations
- ✅ **Type Safety**: Full TypeScript integration with API client
- ✅ **Error Handling**: Comprehensive error handling with user feedback
- ✅ **Status Enum**: Proper enum handling for agent status

### 3. Dashboard Access Control
- ✅ **Conditional Sidebar**: Hide sidebar when no agents configured
- ✅ **No Agents State**: Show dedicated page for first-time users
- ✅ **Loading States**: Proper loading indicators during API calls
- ✅ **Error States**: User-friendly error messages and recovery

### 4. UI/UX Improvements
- ✅ **Responsive Design**: Mobile-friendly agent management interface
- ✅ **Real-time Updates**: Live statistics and status indicators
- ✅ **Search and Filter**: Enhanced agent list with search capabilities
- ✅ **Confirmation Dialogs**: Safe deletion with user confirmation

## API Endpoints Reference

### Base URL
- **Development**: `http://localhost:8000`
- **Production**: Configure in environment variables

### Agents Endpoints

| Method | Endpoint | Description | Request Body | Response |
|--------|----------|-------------|--------------|----------|
| GET | `/api/agents/` | Get all agents | - | `Agent[]` |
| GET | `/api/agents/stats` | Get agent statistics | - | `AgentStats` |
| GET | `/api/agents/{id}` | Get agent by ID | - | `Agent` |
| POST | `/api/agents/` | Create new agent | `AgentCreate` | `Agent` |
| PUT | `/api/agents/{id}` | Update agent | `AgentUpdate` | `Agent` |
| DELETE | `/api/agents/{id}` | Delete agent | - | `{message: string}` |

### Example API Calls

#### Create Agent
```bash
curl -X POST "http://localhost:8000/api/agents/" \
  -H "Content-Type: application/json" \
  -d '{
    "hostname": "server-01",
    "ip": "192.168.1.100",
    "mac": "00:11:22:33:44:55",
    "status": "active",
    "ssh_pass": "secure_password",
    "sudo_pass": "sudo_password"
  }'
```

#### Get Statistics
```bash
curl -X GET "http://localhost:8000/api/agents/stats"
```

## Setup Instructions

### 1. Backend Setup (FastAPI)

1. **Navigate to backend directory**:
   ```bash
   cd SIEMZello-api
   ```

2. **Activate virtual environment**:
   ```bash
   # Windows
   .\venv\Scripts\Activate.ps1
   
   # Linux/Mac
   source venv/bin/activate
   ```

3. **Install dependencies**:
   ```bash
   pip install fastapi uvicorn python-multipart
   ```

4. **Start the server**:
   ```bash
   uvicorn app.main:app --reload --port 8000
   ```

### 2. Frontend Setup (Next.js)

1. **Navigate to frontend directory**:
   ```bash
   cd SIEMZello-ui/siemzello-ui
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Start development server**:
   ```bash
   npm run dev
   ```

### 3. Access the Application

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Documentation**: http://localhost:8000/docs

## Testing the Implementation

### 1. Test Backend API
```bash
# Test agent statistics
curl -X GET "http://localhost:8000/api/agents/stats"

# Test creating an agent
curl -X POST "http://localhost:8000/api/agents/" \
  -H "Content-Type: application/json" \
  -d '{"hostname":"test","ip":"192.168.1.1","mac":"00:11:22:33:44:55","status":"active","ssh_pass":"pass","sudo_pass":"sudo"}'

# Test getting all agents
curl -X GET "http://localhost:8000/api/agents/"
```

### 2. Test Frontend Integration
1. Visit http://localhost:3000/dashboard
2. If no agents exist: Should show "No Agents State" page
3. If agents exist: Should show full dashboard with sidebar
4. Visit http://localhost:3000/agents to manage agents

### 3. Test Dashboard Behavior
1. **No Agents**: Dashboard shows only "Add your first agent" message
2. **With Agents**: Full dashboard with sidebar navigation
3. **Agent Operations**: Create, edit, delete agents through UI
4. **Real-time Updates**: Statistics update immediately after changes

## Migration Benefits

### 1. Performance Improvements
- **Centralized API**: Single source of truth for data operations
- **Better Caching**: FastAPI built-in response caching
- **Async Operations**: Non-blocking database operations
- **Connection Pooling**: Efficient database connection management

### 2. Development Experience
- **Type Safety**: Full TypeScript integration
- **API Documentation**: Auto-generated OpenAPI docs
- **Error Handling**: Comprehensive error messages
- **Hot Reload**: Both backend and frontend support hot reload

### 3. Security Enhancements
- **Input Validation**: Pydantic validation prevents invalid data
- **SQL Injection Protection**: Parameterized queries
- **CORS Configuration**: Proper cross-origin request handling
- **Error Sanitization**: No sensitive data in error responses

### 4. Maintainability
- **Separation of Concerns**: Clear API/UI boundaries
- **Service Layer**: Business logic separated from API routes
- **Consistent Patterns**: Standardized CRUD operations
- **Testing Ready**: Easy to unit test API endpoints

## Future Enhancements

### 1. Authentication & Authorization
- JWT token-based authentication
- Role-based access control
- API key management for agents

### 2. Real-time Features
- WebSocket connections for live updates
- Agent health monitoring
- Real-time event streaming

### 3. Monitoring & Logging
- Request/response logging
- Performance metrics
- Error tracking and alerting

### 4. Database Improvements
- Migration to PostgreSQL
- Database connection pooling
- Backup and recovery procedures

## Troubleshooting

### Common Issues

1. **CORS Errors**
   - Ensure FastAPI CORS middleware is configured
   - Check frontend API base URL configuration

2. **Connection Refused**
   - Verify FastAPI server is running on port 8000
   - Check firewall settings

3. **Database Errors**
   - Ensure SQLite database file permissions
   - Check database file path in configuration

4. **Type Errors**
   - Update frontend types to match backend schemas
   - Ensure agent status uses enum values

### Debug Commands

```bash
# Check FastAPI server status
curl -X GET "http://localhost:8000/docs"

# Check database contents
sqlite3 SIEMZello-api/data/agents.db ".tables"
sqlite3 SIEMZello-api/data/agents.db "SELECT * FROM agents;"

# Test frontend API client
# Open browser console at http://localhost:3000
# Run: await fetch('http://localhost:8000/api/agents/stats')
```

## Conclusion

The migration to FastAPI backend provides a robust, scalable foundation for the SIEMZello agent management system. The implementation includes:

- ✅ Complete API migration from Next.js to FastAPI
- ✅ Conditional dashboard access based on agent configuration
- ✅ Type-safe frontend integration with centralized API client
- ✅ Comprehensive error handling and validation
- ✅ User-friendly no-agents state for first-time users
- ✅ Real-time statistics and status updates

The system is now ready for production deployment and can be easily extended with additional features like authentication, real-time monitoring, and advanced security capabilities.
