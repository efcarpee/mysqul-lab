🐋 Docker Compose Lab — Ghost + MySQL + 🤖 OpenWebUI + Ollama

by François Carpe — Trinity College, Cloud-Native Computing (2025)

📘 Overview

This project demonstrates multi-container orchestration with Docker Compose, persistent data via volumes, and running local LLMs. It contains two independent stacks:

Part	Stack	What you’ll learn
🧱 Part 1	Ghost + MySQL	CMS + database, service dependencies, volumes & persistence
🧠 Part 2	OpenWebUI + Ollama	Local LLM serving, model management, troubleshooting networking/ports
🧰 Prerequisites

Docker Desktop running

Git installed

Terminal (PowerShell on Windows)

≥ 10 GB free disk (models + images)

Internet access

Basic CLI familiarity

⚙️ Part 1 — Ghost + MySQL
1) Create workspace
cd "C:\Users\<yourname>\OneDrive\Documentos\Cloud-Native"
mkdir ghost-mysql-lab
cd ghost-mysql-lab

2) Create docker-compose.yaml
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
      - "8080:2368"        # use "8081:2368" if 8080 is busy
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


If you change the host port to 8081, also change url: http://localhost:8081.

3) Start services
docker compose up -d
docker compose ps


You should see ghost-app and ghost-mysql Up with 0.0.0.0:8080->2368/tcp.

4) Initialize Ghost

Open http://localhost:8080/ghost

Create admin, set blog title, publish a test post

5) Test persistence
docker compose down
docker compose up -d


Return to /ghost and confirm your user, title, and post still exist ✅

🛠 Common Ghost issues
Symptom	Cause	Fix
Browser can’t reach site	Port 8080 in use	Change to "8081:2368" and update url
ECONNREFUSED in ghost logs	MySQL not ready	Wait 20–30s or docker compose restart ghost
no configuration file provided	Wrong directory	Run commands inside ghost-mysql-lab
MySQL auth errors	Env mismatch	Ensure MySQL creds match Ghost DB env
Stuck/DB corrupt	Dirty data	docker compose down -v (⚠️ deletes data) and rerun
💾 Why data persists

mysql_data keeps database files

ghost_content keeps uploads/themes
Volumes live outside containers, so data survives down/up.

🤖 Part 2 — OpenWebUI + Ollama
1) Create workspace
cd "C:\Users\<yourname>\OneDrive\Documentos\Cloud-Native"
mkdir ollama-webui-lab
cd ollama-webui-lab

2) Create docker-compose.yaml
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
      - "3001:8080"        # pick 3002 if 3001 is busy
    volumes:
      - openwebui_data:/app/backend/data

volumes:
  ollama_data:
  openwebui_data:

3) Start services
docker compose up -d
docker compose ps


Expect open-webui at 0.0.0.0:3001->8080/tcp and ollama at 11434.

4) Access UI and create account

Open http://localhost:3001

Create a local admin account (stored locally only)

5) Pull a model

Via UI: Settings → Models → “Pull a model” → llama3.2:1b → Download (~1.3 GB)

If UI fails (download canceled), use CLI:

docker exec -it ollama ollama pull llama3.2:1b
docker exec -it ollama ollama list   # confirm model presence


If DNS errors: add to ollama service

dns:
  - 8.8.8.8
  - 1.1.1.1


Then:

docker compose down
docker compose up -d

6) Chat!

Reload the page → select llama3.2:1b (top bar) → start chatting:

“Explain Docker Compose in simple terms.”

“Write a Python ‘Hello World’.”

🛠 Common OpenWebUI/Ollama issues
Symptom	Cause	Fix
Bind for 0.0.0.0:3000 failed	Port in use	Use "3001:8080" or "3002:8080"
Download canceled	Network/DNS	Use CLI pull; add dns:; check firewall/proxy
Name resolution failure	DNS	Add dns: 8.8.8.8, 1.1.1.1
No space left	Disk full	docker system prune -a (⚠️ removes unused images)
Container restarts	Low RAM	Increase Docker memory (6–8 GB recommended)
📸 Screenshots to include

Ghost blog with custom title

Docker Desktop showing ghost-app and ghost-mysql running

OpenWebUI chat with 2–3 exchanges

Docker Desktop showing open-webui and ollama running

Suggested structure:

ghost-mysql-lab/
  docker-compose.yaml
  README.md
  screenshots/
    ghost_blog.png
    docker_ghost_mysql.png

ollama-webui-lab/
  docker-compose.yaml
  README.md
  screenshots/
    openwebui_chat.png
    docker_openwebui_ollama.png

🧩 Reflection (sample answers)

What does -d do? Runs containers in detached mode (background).

Why does Ghost data persist? Named volumes (mysql_data, ghost_content) live outside containers.

Resource usage: run docker system df and report Images and Volumes space used.

Similarities: both stacks use multiple services, port mappings, and volumes.

Differences: Ghost ↔ MySQL (DB); OpenWebUI ↔ Ollama (LLM serving).

When to prefer local LLMs: privacy, offline use, predictable costs, low latency demos.



