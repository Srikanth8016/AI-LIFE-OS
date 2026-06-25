# AI LifeOS

> Your personal AI-powered operating system — tasks, notes, calendar, finances, documents, emails, habits, health, and knowledge in one place, powered by local AI.

## What is AI LifeOS?

AI LifeOS combines the best of Notion + ChatGPT + Google Calendar + Todoist + Memory + RAG into a single, unified personal productivity platform. Unlike cloud AI tools, it runs 100% locally using Ollama — your data never leaves your machine.

## Key Features

- **AI Assistant** — Natural language task and reminder creation
- **Smart Notes** — Auto-summarize, tag, and link related notes
- **AI Search** — Semantic search over all your personal data via vector DB
- **Tasks & Calendar** — Auto-create tasks, priorities, and calendar events
- **Document Chat** — Upload PDF/Word/Excel and ask questions
- **AI Memory** — Remembers your preferences, projects, habits, and goals
- **Daily Brief** — Morning summary of meetings, tasks, emails, and weather
- **Expense Tracker** — Auto-categorize spending with monthly reports
- **Voice Assistant** — Speak commands naturally
- **Knowledge Base** — Save and search YouTube, articles, blogs, GitHub, papers

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| Frontend | Next.js, TypeScript, Tailwind CSS, ShadCN, React Query, Framer Motion, Zustand |
| Backend | FastAPI, SQLAlchemy, Alembic, Pydantic, Celery, Redis, JWT |
| Database | PostgreSQL, Qdrant, Redis |
| AI | Ollama (llama3, mistral, phi4, qwen2.5, gemma), LangChain, Sentence Transformers |
| Storage | MinIO |
| Notifications | Web Push, Email |

## Project Structure

```
AI-LifeOS/
├── docs/           # Project documentation
├── frontend/       # Next.js frontend
├── backend/        # FastAPI backend
├── docker/         # Docker configurations
├── nginx/          # Nginx reverse proxy config
└── README.md
```

## Documentation

| File | Description |
|------|-------------|
| [01-project-overview.md](docs/01-project-overview.md) | Vision, goals, and problem statement |
| [02-features.md](docs/02-features.md) | Detailed feature specifications |
| [03-architecture.md](docs/03-architecture.md) | System architecture and design |
| [04-database-design.md](docs/04-database-design.md) | Database schema and collections |
| [05-api-design.md](docs/05-api-design.md) | REST API endpoints |
| [06-ui-pages.md](docs/06-ui-pages.md) | UI pages and components |
| [07-development-roadmap.md](docs/07-development-roadmap.md) | Milestones and sprint plan |
| [08-deployment.md](docs/08-deployment.md) | Deployment and infrastructure guide |

## Getting Started

```bash
# Clone the repo
git clone https://github.com/yourusername/AI-LifeOS.git
cd AI-LifeOS

# Start backend
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload

# Start frontend
cd frontend
npm install
npm run dev
```

## License

MIT
