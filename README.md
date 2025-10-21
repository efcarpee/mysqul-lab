# 🐋 Docker Compose Lab — Ghost + MySQL + 🧠 OpenWebUI + Ollama

> 🚀 A full hands-on lab exploring multi-container apps, data persistence, and local LLMs — built for Cloud-Native Computing at Trinity College.

---

## 🧱 Project Overview

This lab is divided into **two parts**:

| Part | Stack | Description |
|------|--------|-------------|
| **1** | 📰 **Ghost + MySQL** | Learn how a CMS works with a backend database and how data persists with Docker Volumes. |
| **2** | 🤖 **OpenWebUI + Ollama** | Run local LLMs (like `llama3.2:1b`) inside Docker and chat through a ChatGPT-style interface. |

---

## 🧰 Tech Stack
- 🐳 **Docker Desktop**
- ⚙️ **Docker Compose**
- 🧱 **MySQL 8**
- 👻 **Ghost 6-alpine**
- 🤖 **Ollama + OpenWebUI**
- 💾 **Persistent Volumes** for database and model storage
- 💻 **Windows 10 / 11 + PowerShell**

---

## ⚙️ Setup Instructions

### 📰 Part 1 — Ghost + MySQL
1. Create a new folder:
   ```bash
   mkdir ghost-mysql-lab && cd ghost-mysql-lab
