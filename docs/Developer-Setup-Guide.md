# SIEMZello Developer Setup Guide

**Quick Reference:** Complete setup instructions for SIEMZello development environment

---

## 🚀 Quick Start

### Prerequisites
- **Python 3.8+** (Backend)
- **Node.js 18+** (Frontend)
- **PowerShell 7+** (Windows development)
- **Git** (Version control)

### 1-Minute Setup

```powershell
# Clone and setup backend
cd "c:\Users\MONSTERV2\Desktop\My Fac (INSAT)\RT3\1PPP\SIEMZello-api"
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt

# Start backend (Terminal 1)
python .\start_server.py

# Setup frontend (Terminal 2)
cd "..\SIEMZello-ui\siemzello-ui"
npm install
npm run dev -- --host 0.0.0.0 --port 3000

# Test the connection
curl http://localhost:8000/health
# Open browser: http://localhost:3000/test/api
```

---

## 🔧 Detailed Setup

### Backend Setup (FastAPI)

#### 1. Environment Setup
```powershell
# Navigate to API directory
cd "c:\Users\MONSTERV2\Desktop\My Fac (INSAT)\RT3\1PPP\SIEMZello-api"

# Create virtual environment
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Verify Python version
python --version  # Should be 3.8+
```

#### 2. Install Dependencies
```powershell
# Install all required packages
pip install -r requirements.txt

# Verify installation
pip list | findstr fastapi
pip list | findstr uvicorn
```

#### 3. Start Development Server
```powershell
# Method 1: Using the startup script
python .\start_server.py

# Method 2: Direct uvicorn command
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Method 3: Background service
Start-Process python -ArgumentList "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"
```

#### 4. Verify Backend
```powershell
# Test health endpoint
Invoke-RestMethod -Uri "http://localhost:8000/health"

# Test API documentation
Start-Process "http://localhost:8000/docs"

# Test specific endpoint
Invoke-RestMethod -Uri "http://localhost:8000/api/agents/stats/overview"
```

### Frontend Setup (Next.js)

#### 1. Navigate to Frontend
```powershell
cd "c:\Users\MONSTERV2\Desktop\My Fac (INSAT)\RT3\1PPP\SIEMZello-ui\siemzello-ui"
```

#### 2. Install Dependencies
```powershell
# Install packages using npm
npm install

# Or using pnpm (if preferred)
pnpm install

# Verify installation
npm list next
npm list react
```

#### 3. Start Development Server
```powershell
# Method 1: Standard development
npm run dev

# Method 2: Network-accessible
npm run dev -- --host 0.0.0.0 --port 3000

# Method 3: Custom port
npm run dev -- --port 3001
```

#### 4. Verify Frontend
```powershell
# Open main application
Start-Process "http://localhost:3000"

# Test API integration
Start-Process "http://localhost:3000/test/api"

# Check dashboard
Start-Process "http://localhost:3000/dashboard"
```

---

## 🌐 Network Configuration

### Local Development
- **Backend:** `http://localhost:8000`
- **Frontend:** `http://localhost:3000`
- **API Docs:** `http://localhost:8000/docs`

### Network Access (for testing from other devices)
```powershell
# Get your local IP address
$localIP = (Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias "Wi-Fi").IPAddress
Write-Host "Your local IP: $localIP"

# Start backend on all interfaces
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Start frontend on all interfaces
npm run dev -- --host 0.0.0.0 --port 3000

# Access from other devices
# Backend: http://$localIP:8000/docs
# Frontend: http://$localIP:3000
```

### CORS Configuration
The backend is configured to accept requests from any origin:
```python
# In app/main.py
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Allows all origins
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 🛠️ Development Tools

### Recommended VS Code Extensions
```json
{
  "recommendations": [
    "ms-python.python",
    "ms-python.flake8",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

### PowerShell Aliases for Quick Development
```powershell
# Add to your PowerShell profile
function Start-SIEMBackend {
    Set-Location "c:\Users\MONSTERV2\Desktop\My Fac (INSAT)\RT3\1PPP\SIEMZello-api"
    .\venv\Scripts\Activate.ps1
    python .\start_server.py
}

function Start-SIEMFrontend {
    Set-Location "c:\Users\MONSTERV2\Desktop\My Fac (INSAT)\RT3\1PPP\SIEMZello-ui\siemzello-ui"
    npm run dev -- --host 0.0.0.0 --port 3000
}

function Test-SIEMConnection {
    Write-Host "Testing backend health..."
    try {
        $response = Invoke-RestMethod -Uri "http://localhost:8000/health"
        Write-Host "✓ Backend: $($response.status)" -ForegroundColor Green
    } catch {
        Write-Host "✗ Backend: Not responding" -ForegroundColor Red
    }
    
    Write-Host "Testing frontend..."
    try {
        $response = Invoke-WebRequest -Uri "http://localhost:3000" -Method Head
        Write-Host "✓ Frontend: Available" -ForegroundColor Green
    } catch {
        Write-Host "✗ Frontend: Not responding" -ForegroundColor Red
    }
}

# Usage:
# Start-SIEMBackend
# Start-SIEMFrontend  
# Test-SIEMConnection
```

---

## 🐛 Common Issues & Solutions

### Issue: Backend won't start
```powershell
# Check if port 8000 is in use
netstat -an | findstr :8000

# Kill process using port 8000
$process = Get-Process -Id (Get-NetTCPConnection -LocalPort 8000).OwningProcess
Stop-Process -Id $process.Id -Force

# Restart backend
python .\start_server.py
```

### Issue: Frontend build errors
```powershell
# Clear node modules and reinstall
Remove-Item -Recurse -Force node_modules
Remove-Item package-lock.json
npm install

# Clear Next.js cache
npx next clean
npm run dev
```

### Issue: API connection failed
```powershell
# Check API client configuration
# The frontend should automatically detect the correct API URL
# For debugging, check browser console for API base URL

# Manual override (for testing)
# localStorage.setItem('API_BASE_URL_OVERRIDE', 'http://192.168.1.100:8000');
```

### Issue: Python virtual environment issues
```powershell
# Recreate virtual environment
Remove-Item -Recurse -Force venv
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

---

## 📝 Development Workflow

### Daily Development Routine
1. **Start Backend:**
   ```powershell
   cd SIEMZello-api
   .\venv\Scripts\Activate.ps1
   python .\start_server.py
   ```

2. **Start Frontend:**
   ```powershell
   cd SIEMZello-ui\siemzello-ui
   npm run dev -- --host 0.0.0.0
   ```

3. **Test Connection:**
   ```powershell
   # Quick test
   curl http://localhost:8000/health
   # Open browser: http://localhost:3000/test/api
   ```

### Making Changes

#### Backend Changes (Python/FastAPI):
1. Edit files in `app/api/` or `app/models/`
2. Server auto-reloads with `--reload` flag
3. Test endpoint: `http://localhost:8000/docs`
4. Verify with frontend integration

#### Frontend Changes (React/Next.js):
1. Edit files in `app/`, `components/`, or `lib/`
2. Next.js hot-reloads automatically
3. Test in browser: `http://localhost:3000`
4. Check console for errors

### Git Workflow
```powershell
# Create feature branch
git checkout -b feature/new-endpoint

# Make changes and test
# ... development work ...

# Stage and commit changes
git add .
git commit -m "Add new endpoint for X feature"

# Push and create PR
git push origin feature/new-endpoint
```

---

## 🔍 Monitoring & Debugging

### Backend Monitoring
```powershell
# Monitor FastAPI logs
Get-Content -Path "logs/api.log" -Wait

# Check system resources
Get-Process | Where-Object {$_.ProcessName -eq "python"}
```

### Frontend Monitoring
```powershell
# Monitor Next.js output
# Check terminal where npm run dev is running

# Browser developer tools
# F12 > Console for JavaScript errors
# F12 > Network for API call monitoring
```

### Performance Monitoring
```powershell
# Check API response times
Measure-Command { Invoke-RestMethod -Uri "http://localhost:8000/api/agents/" }

# Monitor memory usage
Get-Process python | Format-Table Name, CPU, WorkingSet
```

---

*Last updated: June 3, 2025*
