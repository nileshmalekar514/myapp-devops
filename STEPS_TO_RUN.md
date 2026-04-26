# Practical 1 — Step-by-Step Guide

Jenkins Pipeline → GitHub → Docker → Kubernetes

---

## PHASE 1: Install Tools (1 hour)

### Step 1.1 — Install Java (required by Jenkins)
1. Open browser → go to: https://adoptium.net/
2. Click "Latest LTS Release" → Download MSI for Windows x64
3. Run the .msi file → Next → Next → Install
4. **Verify:** Open Command Prompt → type:
   ```
   java -version
   ```
   Should show: `openjdk version "21.x.x"`

---

### Step 1.2 — Install Docker Desktop
1. Open browser → go to: https://www.docker.com/products/docker-desktop/
2. Click "Download for Windows"
3. Run `Docker Desktop Installer.exe`
4. Check "Use WSL 2 instead of Hyper-V" → OK
5. Wait for install → **Restart computer when asked**
6. After restart → Open Docker Desktop from Start Menu
7. Accept terms → Skip sign-in
8. Wait until you see green "Engine running" at bottom-left
9. **Verify:** Open CMD → type:
   ```
   docker --version
   ```
   Should show: `Docker version 24.x.x`

---

### Step 1.3 — Install Jenkins
1. Go to: https://www.jenkins.io/download/
2. Click "Windows" → Download `jenkins.msi`
3. Run installer → Next → Next
4. Choose "Run as Local System" (easier)
5. Port: **8080** (default) → Next → Install
6. Jenkins starts automatically
7. Open browser → go to: http://localhost:8080
8. Page asks for "Administrator Password"
9. Open File Explorer → go to:
   ```
   C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
   ```
10. Open file with Notepad → Copy the password
11. Paste in browser → Continue
12. Click "Install Suggested Plugins" → wait 5-10 min
13. Create First Admin User:
    - Username: `admin`
    - Password: `admin123`
    - Full name: `Your Name`
    - Email: `your@email.com`
14. Click Save → Start using Jenkins
15. You should see Jenkins Dashboard

---

### Step 1.4 — Install kubectl
1. Open PowerShell as Administrator
2. Run:
   ```
   curl.exe -LO "https://dl.k8s.io/release/v1.29.0/bin/windows/amd64/kubectl.exe"
   ```
3. Move kubectl.exe to `C:\Windows\System32\` (or any folder in PATH)
4. **Verify:** CMD → type:
   ```
   kubectl version --client
   ```

---

### Step 1.5 — Install Minikube
1. Open PowerShell as Administrator
2. Run:
   ```
   New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
   Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
   ```
3. Add to PATH:
   ```
   $oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
   [Environment]::SetEnvironmentVariable('Path', $oldPath + ';C:\minikube', [EnvironmentVariableTarget]::Machine)
   ```
4. Close and reopen PowerShell
5. **Start Minikube:**
   ```
   minikube start --driver=docker
   ```
6. Wait 5 minutes → should see "Done! kubectl is now configured"

---

## PHASE 2: Create Accounts (10 min)

### Step 2.1 — GitHub Account
1. Go to: https://github.com/signup
2. Sign up with email
3. Verify email

### Step 2.2 — Docker Hub Account
1. Go to: https://hub.docker.com/signup
2. Sign up with email
3. Verify email

---

## PHASE 3: Push Code to GitHub (10 min)

### Step 3.1 — Install Git
1. Go to: https://git-scm.com/download/win
2. Download → Install with default options

### Step 3.2 — Create GitHub Repository
1. Login to GitHub → Click "+" → New repository
2. Repository name: `myapp-devops`
3. Public → Create repository

### Step 3.3 — Push Practical 1 Code
1. Open Command Prompt
2. Navigate to Practical 1 folder:
   ```
   cd "C:\Users\HP\OneDrive\Documents\practical sem 2_tulsiramji\DevOps Practicals\Practical 1"
   ```
3. Run these commands one by one:
   ```
   git init
   git add .
   git config user.email "your@email.com"
   git config user.name "Your Name"
   git commit -m "Initial commit - Practical 1"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/myapp-devops.git
   git push -u origin main
   ```
4. Enter GitHub username and password (or token)

---

## PHASE 4: Configure Jenkins (20 min)

### Step 4.1 — Install Jenkins Plugins
1. Open Jenkins: http://localhost:8080
2. Go to: **Manage Jenkins** → **Plugins** → **Available**
3. Search and install:
   - `Docker Pipeline`
   - `Kubernetes CLI`
   - `GitHub Integration`
4. Click "Install without restart"

### Step 4.2 — Add GitHub Credentials
1. **Manage Jenkins** → **Credentials** → **System** → **Global credentials**
2. Click "Add Credentials"
3. Kind: Username with password
4. Username: your GitHub username
5. Password: GitHub Personal Access Token (create at github.com → Settings → Developer settings → Personal access tokens)
6. ID: `github-creds`
7. Click Create

### Step 4.3 — Add Docker Hub Credentials
1. Same place: Add Credentials
2. Kind: Username with password
3. Username: your Docker Hub username
4. Password: Docker Hub password
5. ID: `dockerhub-creds`
6. Click Create

---

## PHASE 5: Create and Run Pipeline (10 min)

### Step 5.1 — Update Jenkinsfile
1. Open `Jenkinsfile` in Practical 1 folder
2. Replace `yourusername/myapp` with `YOUR-DOCKERHUB-USERNAME/myapp`
3. Replace GitHub URL with your repo URL
4. Save → push to GitHub:
   ```
   git add Jenkinsfile
   git commit -m "Update Jenkinsfile"
   git push
   ```

### Step 5.2 — Create Pipeline Job in Jenkins
1. Jenkins Dashboard → **New Item**
2. Name: `myapp-pipeline`
3. Choose: **Pipeline** → OK
4. Scroll to **Pipeline** section:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL: `https://github.com/YOUR-USERNAME/myapp-devops.git`
   - Credentials: select `github-creds`
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`
5. Click **Save**

### Step 5.3 — Run the Pipeline
1. On the pipeline page → click **Build Now** (left side)
2. Watch the build execute (bottom left "Build History")
3. Click on build #1 → **Console Output**
4. Watch each stage:
   - Checkout ✓
   - Build ✓
   - Docker Build ✓
   - Docker Push ✓
   - Deploy to Kubernetes ✓

### Step 5.4 — Verify Deployment
1. Open Command Prompt
2. Run:
   ```
   kubectl get pods
   ```
3. You should see pods Running
4. Run:
   ```
   minikube service myapp-service --url
   ```
5. Open the URL in browser → see "Hello from Jenkins-Docker-Kubernetes Pipeline!"

---

## SCREENSHOTS TO TAKE FOR PRACTICAL SUBMISSION

1. Jenkins Dashboard with pipeline created
2. Pipeline build console output (success - green)
3. Docker Hub page showing the pushed image
4. CMD showing `kubectl get pods` with Running status
5. Browser showing the deployed Flask app

---

## TROUBLESHOOTING

| Problem | Solution |
|---------|----------|
| Docker not starting | Enable WSL2 in Windows Features |
| Jenkins port busy | Change port in jenkins.xml |
| Minikube fails | Enable virtualization in BIOS |
| kubectl not found | Add to PATH and restart CMD |
| Git push asks password | Use Personal Access Token instead |

---

**Tip:** Take screenshots at each step. If anything fails, screenshot the error and ask for help.
