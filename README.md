# rag-enterprise-chatbot
🤖 RAG-Based Enterprise Knowledge Base Chatbot | Spring Boot + Python FastAPI + React | AI-powered Q&amp;A from company documents using LangChain, ChromaDB &amp; GPT-4
<div align="center">

# 🤖 RAG Enterprise Chatbot

**An AI-powered knowledge base chatbot that answers questions from your uploaded documents.**

Upload PDFs, Word docs, or text files — the system ingests, embeds, and serves intelligent answers with source citations and confidence scores.

[![Java](https://img.shields.io/badge/Java-17-orange?style=flat-square&logo=openjdk)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2-brightgreen?style=flat-square&logo=springboot)](https://spring.io/projects/spring-boot)
[![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat-square&logo=python)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109-009688?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)](https://react.dev/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)](https://docs.docker.com/compose/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=flat-square&logo=postgresql)](https://www.postgresql.org/)

</div>

---

## ✨ Features

- 📄 **Multi-format document ingestion** — PDF, DOCX, TXT, MD, CSV (up to 50MB)
- 🔍 **Semantic search** using ChromaDB + sentence-transformers embeddings
- 🧠 **Flexible LLM backend** — OpenAI GPT-4 Turbo, Anthropic Claude 3.5 Sonnet, or local Llama
- 💬 **Persistent chat sessions** with full message history
- 📌 **Source citations + confidence scores** in every response
- 🔐 **JWT authentication** with role-based access (User / Admin)
- 🛠️ **Admin dashboard** — manage users, documents, and view platform stats
- 🐳 **One-command Docker deployment**

---

## 🏗️ Architecture

```
┌─────────────┐     JWT      ┌──────────────────┐     HTTP     ┌───────────────────┐
│  React SPA  │ ──────────▶  │  Spring Boot API  │ ──────────▶ │ Python ML Service │
│   :3000     │              │   :8080 (/api)    │             │  FastAPI  :8000   │
└─────────────┘              └──────────────────┘             └───────────────────┘
                                      │                                  │
                                      ▼                                  ▼
                              ┌──────────────┐                 ┌─────────────────┐
                              │  PostgreSQL   │                 │ ChromaDB Vector │
                              │    :5432      │                 │     Store       │
                              └──────────────┘                 └─────────────────┘
```

The **Spring Boot backend** handles auth, session management, and document metadata. All RAG operations — ingestion, embedding, similarity search, and LLM calls — are handled by the **Python ML service**.

### RAG Pipeline

```
User question → JWT auth → Python ML service
    → Embed with SentenceTransformer
    → ChromaDB similarity search → Top-K chunks
    → Build prompt: [System] + [Context chunks] + [Question]
    → LLM API (OpenAI / Anthropic / Local)
    → Return { answer, sources[], confidence }
    → Save to PostgreSQL → Display in React UI
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Tailwind CSS, React Query, Axios |
| Backend API | Java 17, Spring Boot 3.2, Spring Security |
| ML Service | Python 3.11, FastAPI, LangChain |
| Vector Store | ChromaDB (persistent) |
| LLM | OpenAI GPT-4 Turbo / Anthropic Claude 3.5 Sonnet |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` |
| Database | PostgreSQL 15 |
| Auth | JWT (access + refresh tokens, BCrypt) |
| Containers | Docker + Docker Compose |

---

## 🚀 Quick Start

### Option 1 — Docker (Recommended)

> **Prerequisites:** Docker and Docker Compose installed.

```bash
# 1. Clone the repo
git clone https://github.com/your-username/rag-chatbot.git
cd rag-chatbot

# 2. Set up environment
cp python-service/.env.example python-service/.env
# Edit python-service/.env — add your OPENAI_API_KEY or ANTHROPIC_API_KEY

# 3. Launch all services
docker-compose up --build
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:8080/api |
| ML Service Docs | http://localhost:8000/docs |

---

### Option 2 — Manual Setup

**Prerequisites:** Java 17, Python 3.11, Node 20, and PostgreSQL installed.

#### 1. Database

```bash
psql -U postgres -c "CREATE DATABASE ragchatbot;"
# or use the provided init script:
psql -U postgres -f docs/init.sql
```

#### 2. Python ML Service

```bash
cd python-service
cp .env.example .env          # Add your API key
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

#### 3. Spring Boot Backend

```bash
cd backend
# Edit src/main/resources/application.properties — set DB password and python service URL
mvn spring-boot:run
```

#### 4. React Frontend

```bash
cd frontend
npm install
npm start
```

---

## ⚙️ Configuration

### Python Service (`python-service/.env`)

```env
# LLM Provider — openai | anthropic | local
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=...

# RAG settings
EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2
CHUNK_SIZE=500
CHUNK_OVERLAP=50
TOP_K_RESULTS=5
SIMILARITY_THRESHOLD=0.3

# ChromaDB
CHROMA_PERSIST_DIR=./chroma_db
CHROMA_COLLECTION_NAME=enterprise_docs
```

### Backend (`application.properties`)

| Property | Description |
|---|---|
| `spring.datasource.*` | PostgreSQL connection and credentials |
| `app.jwt.secret` | 256-bit signing secret — **change in production** |
| `app.jwt.expiration` | Access token TTL in ms (default: 24h) |
| `app.python.service.url` | URL of the Python ML service |
| `app.cors.allowed-origins` | Allowed frontend origins |

---

## 🔑 Default Admin Account

After running `docs/init.sql`, a default admin is seeded:

| Field | Value |
|---|---|
| Email | `admin@enterprise.com` |
| Password | `admin123` |

> ⚠️ **Change this password immediately after first login.**

To promote a self-registered user to admin:

```sql
UPDATE users SET role = 'ADMIN' WHERE email = 'your@email.com';
```

---

## 📡 API Reference

All routes are prefixed with `/api`.

<details>
<summary><strong>Auth</strong></summary>

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register a new user |
| POST | `/auth/login` | Login and receive JWT tokens |

</details>

<details>
<summary><strong>Chat</strong></summary>

| Method | Endpoint | Description |
|---|---|---|
| POST | `/chat/sessions` | Create a new chat session |
| GET | `/chat/sessions` | List all sessions for current user |
| POST | `/chat/sessions/{id}/messages` | Send a message (triggers RAG) |
| GET | `/chat/sessions/{id}/messages` | Get session message history |
| DELETE | `/chat/sessions/{id}` | Delete a session |

</details>

<details>
<summary><strong>Documents (Admin upload/delete)</strong></summary>

| Method | Endpoint | Description |
|---|---|---|
| POST | `/documents/upload` | Upload and ingest a document |
| GET | `/documents` | List all ingested documents |
| DELETE | `/documents/{id}` | Delete document + embeddings |

</details>

<details>
<summary><strong>Admin</strong></summary>

| Method | Endpoint | Description |
|---|---|---|
| GET | `/admin/stats` | Platform-wide statistics |
| GET | `/admin/users` | List all users |
| PUT | `/admin/users/{id}/toggle` | Enable or disable a user |

</details>

---

## 📁 Project Structure

```
rag-chatbot/
├── backend/                          # Spring Boot API Gateway
│   └── src/main/java/com/enterprise/ragchatbot/
│       ├── controller/               # Auth, Chat, Document, Admin controllers
│       ├── service/                  # Business logic
│       ├── model/                    # JPA entities (User, ChatSession, ChatMessage, Document)
│       ├── repository/               # Spring Data JPA
│       ├── security/                 # JWT filter + utility
│       └── config/                   # Security, CORS, async, exception handling
│
├── python-service/                   # ML Microservice (FastAPI)
│   └── app/
│       ├── routes/api.py             # /ingest, /query, /delete, /health
│       ├── services/
│       │   ├── ingestion.py          # Document parsing + LangChain chunking
│       │   ├── vector_store.py       # ChromaDB operations
│       │   └── llm.py                # OpenAI / Anthropic / Local LLM abstraction
│       └── config.py                 # Pydantic settings
│
├── frontend/                         # React SPA
│   └── src/
│       ├── pages/                    # Login, Register, Chat, Admin
│       ├── components/               # Sidebar, ChatInput, MessageBubble
│       ├── context/AuthContext.jsx   # Global auth state
│       └── services/api.js           # Axios API client
│
├── docs/init.sql                     # DB init script with default admin
└── docker-compose.yml
```

---

## 🗄️ Database Schema

```
users              chat_sessions         chat_messages          documents
──────────         ─────────────         ─────────────          ─────────
id (PK)            id (PK)               id (PK)                id (PK)
email              title                 session_id (FK)        original_name
password           user_id (FK)          role                   file_type
full_name          created_at            content (TEXT)         status
role               updated_at            sources                chunk_count
enabled                                  confidence             uploaded_by (FK)
created_at                               created_at             created_at
```

---

## 📦 Supported Document Formats

| Format | Extension |
|---|---|
| PDF | `.pdf` |
| Word Document | `.docx` |
| Plain Text | `.txt` |
| Markdown | `.md` |
| CSV | `.csv` |

Max file size: **50MB** (configurable in `application.properties`).

---

## 🔒 Security

- Passwords hashed with **BCrypt**
- Stateless **JWT** authentication with access + refresh token pattern
- Role-based access control — `USER` and `ADMIN` enforced at the endpoint level via Spring Security
- CORS restricted to configured frontend origins only
- H2 in-memory DB for tests — no real PostgreSQL needed for `mvn test`

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
