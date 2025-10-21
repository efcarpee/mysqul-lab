# 🐋 Docker Compose Lab — Ghost + MySQL + 🤖 OpenWebUI + Ollama  
> *by François “Cocina” Carpe — Trinity College, Cloud-Native Computing (2025)*  

---

## 📘 Overview

This project demonstrates **multi-container orchestration** using **Docker Compose**, **persistent data storage**, and **local LLMs**.  
You’ll deploy two independent stacks:

| Part | Stack | Description |
|------|--------|-------------|
| 🧱 **Part 1** | Ghost + MySQL | Learn how a CMS works with a backend database and how data persists using volumes |
| 🧠 **Part 2** | Ollama + OpenWebUI | Explore local LLM deployment with Docker and use OpenWebUI as a ChatGPT-like interface |

---

## 🧰 Prerequisites

- 🐳 Docker Desktop installed and running  
- 💻 Git installed  
- ⚙️ PowerShell or terminal access  
- 🗂️ At least 10 GB free disk space  
- 🌐 Internet access (for pulling images/models)  
- 🧑‍💻 Basic terminal familiarity  

---

## ⚙️ Part 1 — Ghost + MySQL

### Step 1: Create your workspace
```bash
cd "C:\Users\<yourname>\OneDrive\Documentos\Cloud-Native"
mkdir ghost-mysql-lab
cd ghost-mysql-lab
Step 2: Create the docker-compose.yaml
yaml
Copiar código
services:
  mysql:
    image: mysql:8.0
    container_name: ghost-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: ghost
      MYSQL_USER: ghost
      MYSQL_PASSWORD: ghostpass
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -prootpass || exit 1"]
      interval: 5s
      timeout: 3s
      retries: 20

  ghost:
    image: ghost:6-alpine
    container_name: ghost-app
    depends_on:
      mysql:
        condition: service_healthy
    restart: unless-stopped
    ports:
      - "8080:2368"
    environment:
      url: http://localhost:8080
      database__client: mysql
      database__connection__host: mysql
      database__connection__user: ghost
      database__connection__password: ghostpass
      database__connection__database: ghost
    volumes:
      - ghost_content:/var/lib/ghost/content

volumes:
  mysql_data:
  ghost_content:
Step 3: Build and start
bash
Copiar código
docker compose up -d
-d = detached mode (runs in background).

The first time might take a few minutes while downloading images.

Check if running:

bash
Copiar código
docker compose ps
Expected:

bash
Copiar código
ghost-app     Up 10s (healthy)  0.0.0.0:8080->2368/tcp
ghost-mysql   Up 10s (healthy)  3306/tcp
Step 4: Open Ghost
Visit http://localhost:8080/ghost
Create your admin account and publish a test post.

If it fails to load:

bash
Copiar código
docker compose logs --tail=100
docker compose restart ghost
Step 5: Test persistence
Stop and restart containers:

bash
Copiar código
docker compose down
docker compose up -d
Your post and blog name should still exist ✅

💾 Why Data Persists
Your content lives inside Docker volumes:

mysql_data → database tables

ghost_content → uploads, media, themes

View volumes:

bash
Copiar código
docker volume ls
⚠️ Common Ghost Errors
Issue	Cause	Fix
Port conflict	8080 already in use	Change to 8081:2368
“ECONNREFUSED”	MySQL not ready	Restart ghost or add depends_on
“no config file provided”	Wrong directory	Run from inside ghost-mysql-lab
MySQL access denied	Wrong env vars	Verify user/password
Containers stuck	MySQL corruption	docker compose down -v then rebuild

🤖 Part 2 — Ollama + OpenWebUI
Step 1: Create new folder
bash
Copiar código
cd "C:\Users\<yourname>\OneDrive\Documentos\Cloud-Native"
mkdir ollama-webui-lab
cd ollama-webui-lab
Step 2: Add docker-compose.yaml
yaml
Copiar código
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    depends_on:
      - ollama
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    ports:
      - "3001:8080"
    volumes:
      - openwebui_data:/app/backend/data

volumes:
  ollama_data:
  openwebui_data:
Step 3: Start containers
bash
Copiar código
docker compose up -d
Verify:

bash
Copiar código
docker compose ps
Expected:

bash
Copiar código
open-webui   Up   0.0.0.0:3001->8080/tcp
ollama       Up   0.0.0.0:11434->11434/tcp
Step 4: Access interface
Go to http://localhost:3001
Create a local account (stored locally).

Step 5: Pull a model
Inside OpenWebUI → Settings → Models → “Pull a model”
Enter:

makefile
Copiar código
llama3.2:1b
Wait for ~1.3GB download.

If canceled:

bash
Copiar código
docker exec -it ollama ollama pull llama3.2:1b
If DNS errors appear:
Add inside ollama: block:

yaml
Copiar código
dns:
  - 8.8.8.8
  - 1.1.1.1
Then rebuild:

bash
Copiar código
docker compose down
docker compose up -d
Step 6: Chat with your model
Back to main UI → select llama3.2:1b.
Try asking:

Explain Docker Compose in simple terms.
Write a Python “Hello World” script.

You now have a local ChatGPT running on Docker 🧠🔥

⚠️ Common Ollama Errors
Problem	Fix
Port 3000 busy	Change to 3002:8080
Download canceled	Run manual pull
DNS failure	Add Google DNS 8.8.8.8
Out of space	docker system prune -a
Container restarts	Allocate more RAM (6–8GB)

🧹 Cleanup
bash
Copiar código
docker compose down             # Stop and remove containers
docker compose down -v          # Also remove volumes
docker system prune -a          # Clean images and cache
📸 Screenshots to Submit
Section	Description
📰 Ghost Blog	
🐳 Docker Desktop (Ghost + MySQL)	
🤖 OpenWebUI Chat	
🐋 Docker Desktop (Ollama + WebUI)	

🧩 Reflection Questions
1️⃣ What does -d do?
Runs containers in detached (background) mode.

2️⃣ Why does data persist?
Docker volumes (mysql_data, ghost_content) store data outside container lifecycle.

3️⃣ Check resource usage:

bash
Copiar código
docker system df
4️⃣ Similarities:
Both YAML files use services, ports, volumes, and depends_on.

5️⃣ Differences:

Ghost uses MySQL (database app)

OpenWebUI uses Ollama (AI inference service)

6️⃣ Why run LLMs locally?

🔒 Privacy

💸 No API fees

⚡ Fast response

🌐 Works offline
