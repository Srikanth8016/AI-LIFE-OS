# AI LifeOS — Feature Documentation

## Feature Index

1. [AI Assistant](#1-ai-assistant)
2. [Smart Notes](#2-smart-notes)
3. [AI Search](#3-ai-search)
4. [Tasks & Reminders](#4-tasks--reminders)
5. [Calendar](#5-calendar)
6. [Document Chat](#6-document-chat)
7. [AI Memory](#7-ai-memory)
8. [Daily Brief](#8-daily-brief)
9. [Expense Tracker](#9-expense-tracker)
10. [Voice Assistant](#10-voice-assistant)
11. [Knowledge Base](#11-knowledge-base)

---

## 1. AI Assistant

### Description
The AI Assistant is the central intelligence layer of AI LifeOS. It's a conversational interface powered by a local LLM (via Ollama) that understands your full context — your notes, tasks, calendar, expenses, and memory — and lets you interact with your entire life through natural language.

### User Stories

- As a user, I want to create a task by typing "remind me to call mom on Friday at 3pm" so that I don't have to navigate to the tasks screen.
- As a user, I want to ask "what did I write about machine learning last month?" so that I can find relevant notes without searching manually.
- As a user, I want to say "summarize my week" so that I get a quick overview of what I accomplished.
- As a user, I want the assistant to remember my preferences so that I don't have to repeat context in every conversation.

### Example Interactions

```
User: "Create a task to review the Q3 budget proposal by Thursday, high priority"
AI: "Done! I've created a high-priority task: 'Review Q3 budget proposal' due Thursday, Oct 19. 
     Want me to also block 2 hours on your calendar for it?"

User: "What are my top priorities for today?"
AI: "Good morning! Here's what needs your attention today:
     🔴 High priority: Review Q3 budget proposal (due Thursday)
     🟡 Medium priority: Follow up with Sarah on the design mockups
     📅 Meeting at 2pm: Team standup
     💡 Note from yesterday: You mentioned wanting to finalize the API design"

User: "How much did I spend on food last month?"
AI: "Last month you spent $342 on food — $180 at restaurants and $162 on groceries. 
     That's 18% above your $290 monthly food budget. Want me to show the breakdown?"
```

### AI Behavior
- Uses the selected local model (default: llama3) via Ollama API
- Retrieves relevant context from notes, tasks, calendar, and memory via RAG
- Maintains conversation history within a session
- Stores important facts from conversations in AI Memory
- Supports follow-up questions and multi-turn dialogue
- Falls back gracefully when the LLM is unavailable

### Acceptance Criteria
- [ ] User can open a chat interface and send messages
- [ ] AI responds within 10 seconds for standard queries
- [ ] AI can create tasks, notes, and events from natural language
- [ ] AI retrieves and references user's notes and memory in responses
- [ ] Conversation history persists across sessions
- [ ] User can select which LLM model to use
- [ ] AI gracefully handles out-of-scope queries

---

## 2. Smart Notes

### Description
Smart Notes is an AI-enhanced note-taking system. Notes are automatically embedded into the vector database, enabling semantic search, AI summarization, auto-tagging, and automatic linking of similar notes.

### User Stories

- As a user, I want my notes automatically tagged so that I don't have to manually organize them.
- As a user, I want to see notes similar to the one I'm reading so that I can discover connections.
- As a user, I want the AI to summarize a long note so that I can quickly recall its contents.
- As a user, I want to write notes in Markdown so that I can format them richly.
- As a user, I want to search my notes semantically so that I find relevant notes even when I don't remember exact words.

### Example Interactions

```
User creates a note about "building a FastAPI app with authentication"
AI automatically:
  - Generates tags: ["fastapi", "python", "backend", "authentication", "api"]
  - Finds similar notes: "JWT tokens in Python", "SQLAlchemy user models"
  - Generates a 2-sentence summary for the note preview
```

### AI Behavior
- On note save: triggers async embedding generation via Celery
- Embedding stored in Qdrant `notes_vectors` collection
- Similar notes found via cosine similarity (top 5 results)
- Auto-tagging via LLM prompt: "Extract 3-7 relevant tags from this note"
- Summarization via LLM prompt: "Summarize this note in 2-3 sentences"
- Tags and summaries are generated asynchronously (non-blocking)

### Note Data Model
| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique identifier |
| title | string | Note title |
| content | text | Markdown content |
| tags | string[] | Auto + manual tags |
| summary | text | AI-generated summary |
| created_at | datetime | Creation timestamp |
| updated_at | datetime | Last update timestamp |
| is_pinned | boolean | Pinned to top |
| is_archived | boolean | Archived state |

### Acceptance Criteria
- [ ] User can create, edit, delete notes
- [ ] Notes support Markdown formatting with preview
- [ ] AI generates tags within 30 seconds of saving
- [ ] AI generates summary within 30 seconds of saving
- [ ] Similar notes panel shows up to 5 related notes
- [ ] Notes are searchable via semantic search
- [ ] Notes can be pinned, archived, and filtered by tags
- [ ] Note editor supports keyboard shortcuts

---

## 3. AI Search

### Description
AI Search provides semantic, vector-based search across all content in AI LifeOS — notes, tasks, documents, knowledge items, and AI memory. Unlike keyword search, it understands meaning and intent, returning relevant results even when exact words don't match.

### User Stories

- As a user, I want to search "ideas about productivity systems" and find relevant notes even if I never used those exact words.
- As a user, I want search results to show me content from notes, documents, and the knowledge base in one place.
- As a user, I want to filter search results by content type so that I can narrow results.

### Example Interactions

```
Query: "how to structure a React project"
Results:
  📝 Note: "Frontend Architecture Decisions" (similarity: 0.92)
  📄 Document: "react-best-practices.pdf" (similarity: 0.87)
  📚 Knowledge: "Article: React folder structure guide" (similarity: 0.84)
  📝 Note: "Next.js project setup notes" (similarity: 0.81)
```

### AI Behavior
- Query is embedded using the same sentence transformer model
- Qdrant queried across all collections simultaneously (notes, documents, knowledge, memory)
- Results ranked by cosine similarity score
- Results filtered by user ownership
- Optional content-type filter applied post-retrieval
- Minimum similarity threshold: 0.65

### Acceptance Criteria
- [ ] Search bar accessible from all pages via keyboard shortcut (Cmd/Ctrl+K)
- [ ] Results appear within 2 seconds
- [ ] Results span notes, documents, and knowledge base
- [ ] Results show content type icon, title, snippet, and similarity score
- [ ] Filter by content type (notes, documents, knowledge, tasks)
- [ ] Clicking a result navigates to the source item
- [ ] Empty state guides user when no results found

---

## 4. Tasks & Reminders

### Description
A full-featured task management system with AI-powered natural language creation, priority management, due dates, recurring tasks, and smart reminders. Tasks integrate with the Calendar and AI Assistant.

### User Stories

- As a user, I want to create a task with "buy groceries tomorrow" and have the due date automatically set.
- As a user, I want to see my tasks organized by priority so that I focus on what matters.
- As a user, I want to receive notifications when tasks are due so that I don't miss deadlines.
- As a user, I want to create recurring tasks like "weekly review every Sunday" so that routines are automated.

### Example Interactions

```
Natural language input: "Submit the tax return by April 15th, very important"
Parsed result:
  Title: "Submit the tax return"
  Due date: April 15
  Priority: High
  
Input: "Call dentist for appointment, remind me 3 days before"
Parsed result:
  Title: "Call dentist for appointment"
  Reminder: 3 days before due date
```

### Task Data Model
| Field | Type | Values |
|-------|------|--------|
| id | UUID | - |
| title | string | Task description |
| description | text | Optional details |
| priority | enum | low, medium, high, urgent |
| status | enum | todo, in_progress, done, cancelled |
| due_date | datetime | Optional deadline |
| reminder_at | datetime | Notification trigger time |
| is_recurring | boolean | Recurring flag |
| recurrence_rule | string | RRULE format |
| tags | string[] | Categorization |
| created_at | datetime | - |

### AI Behavior
- NLP parsing extracts title, due date, priority, and tags from free text
- Uses LLM with structured output (JSON mode) for reliable parsing
- Celery task triggers push notifications at reminder time
- AI Assistant can list, create, update, and complete tasks via chat

### Acceptance Criteria
- [ ] User can create tasks via form or natural language
- [ ] Natural language parsing correctly extracts date, priority >85% of the time
- [ ] Tasks displayed in list and kanban views
- [ ] Filter and sort by priority, due date, status, and tags
- [ ] Push notifications delivered at reminder time
- [ ] Recurring tasks auto-regenerate after completion
- [ ] AI Assistant can manage tasks via chat commands
- [ ] Bulk operations: mark done, delete, change priority

---

## 5. Calendar

### Description
An integrated calendar with natural language event creation, conflict detection, and AI-powered scheduling suggestions. Calendar events sync with tasks and provide context for the AI Assistant and Daily Brief.

### User Stories

- As a user, I want to say "schedule a team meeting next Tuesday at 10am for 1 hour" and have the event created automatically.
- As a user, I want to see my month/week/day view so that I can plan my time.
- As a user, I want the AI to warn me about scheduling conflicts so that I don't double-book.
- As a user, I want tasks with due dates to appear on my calendar so that I have a unified view.

### Example Interactions

```
Input: "Team standup every weekday at 9am starting Monday"
Event created:
  Title: Team standup
  Start: Monday 9:00 AM
  Duration: 30 min (default)
  Recurrence: Monday–Friday
  
Input: "Block 2 hours tomorrow for deep work on the API design"
Event created:
  Title: Deep work — API design
  Start: Tomorrow 9:00 AM (first available 2-hour block)
  Duration: 2 hours
```

### AI Behavior
- NLP event parsing using LLM with structured JSON output
- Conflict detection via database query for overlapping events
- Smart scheduling: suggests open time slots based on existing events
- Recurring event support via RRULE standard
- Events with task associations bidirectionally linked

### Acceptance Criteria
- [ ] Month, week, and day calendar views
- [ ] Create events via form or natural language
- [ ] Recurring events with flexible recurrence rules
- [ ] Conflict detection and warning on overlap
- [ ] Tasks with due dates visible on calendar
- [ ] AI can create events from chat interface
- [ ] Color coding by event category

---

## 6. Document Chat

### Description
Upload PDF, Word, Excel, and image files and have a full conversation with them. Documents are processed, chunked, embedded, and stored in the vector database, enabling semantic retrieval and LLM-powered question answering.

### User Stories

- As a user, I want to upload a 50-page PDF and ask "what are the key conclusions?" so that I don't have to read the whole document.
- As a user, I want to upload an invoice and ask "what is the total amount and due date?" so that I can extract information quickly.
- As a user, I want to ask follow-up questions about a document so that I can explore it conversationally.
- As a user, I want all my uploaded documents to be searchable so that I can find information across files.

### Example Interactions

```
[User uploads "Q3-Financial-Report.pdf"]
AI: "I've processed Q3-Financial-Report.pdf (47 pages). What would you like to know?"

User: "What was the revenue growth compared to Q2?"
AI: "According to the report (page 12), Q3 revenue was $4.2M, representing a 
     23% increase from Q2's $3.4M. The growth was primarily driven by the 
     enterprise segment which grew 41% quarter-over-quarter."

User: "What are the risk factors mentioned?"
AI: "The report identifies three main risk factors (pages 31-33):
     1. Supply chain disruptions affecting hardware procurement
     2. Increased competition in the SMB market segment
     3. Regulatory uncertainty in the EU market"
```

### Processing Pipeline

```
Upload → Validate file type → Extract text (PyMuPDF / python-docx / openpyxl)
       → Chunk text (512 tokens, 50 token overlap)
       → Generate embeddings (sentence-transformers)
       → Store chunks in Qdrant (documents_vectors collection)
       → Store file metadata in PostgreSQL
       → Store file binary in MinIO
```

### Supported Formats
| Format | Library | Notes |
|--------|---------|-------|
| PDF | PyMuPDF | Text + OCR for scanned PDFs |
| Word (.docx) | python-docx | Text extraction |
| Excel (.xlsx) | openpyxl | Sheet data extraction |
| Images | Tesseract OCR | Text extraction from images |
| Plain text | built-in | Direct processing |

### Acceptance Criteria
- [ ] Upload PDF, DOCX, XLSX, and image files up to 50MB
- [ ] Document processed and indexed within 60 seconds
- [ ] Chat interface opens post-upload with document context
- [ ] AI answers cite page/section numbers when available
- [ ] Multiple documents can be open simultaneously
- [ ] Documents listed in a library view with search
- [ ] Document embeddings included in global AI Search

---

## 7. AI Memory

### Description
AI Memory is a persistent, evolving knowledge graph about the user. The system automatically extracts and stores facts, preferences, goals, habits, and project context from conversations and interactions. This memory is injected into every AI context window, making the AI deeply personalized over time.

### User Stories

- As a user, I want the AI to remember that I'm a Python developer so that I don't have to explain my tech stack every conversation.
- As a user, I want the AI to remember my goal of "lose 10kg by December" so that it can factor that into health-related suggestions.
- As a user, I want to view and edit what the AI knows about me so that I have control over my memory.
- As a user, I want the AI to forget specific facts if I request it so that I control my data.

### Memory Categories

| Category | Examples |
|----------|---------|
| Personal Info | Name, age, location, occupation |
| Preferences | Preferred work hours, communication style, food preferences |
| Goals | Short-term and long-term goals |
| Projects | Active projects, deadlines, tech stack |
| Habits | Morning routine, exercise habits, reading goals |
| Skills | Programming languages, tools, expertise areas |
| Relationships | Colleagues, family members, frequent contacts |
| Context | Current focus areas, recent decisions |

### AI Behavior
- After each conversation, Celery job extracts memorable facts via LLM
- Facts stored in PostgreSQL with category, confidence score, and source
- Top relevant memories retrieved via vector similarity for each new conversation
- Memory injected into system prompt: "What the AI knows about you: ..."
- User can mark memories as incorrect or delete them
- Memory automatically expires if not reinforced (configurable TTL)

### Memory Extraction Prompt
```
Analyze this conversation and extract any facts about the user that should be 
remembered for future conversations. Return as JSON array with fields:
- category: one of [personal, preference, goal, project, habit, skill, relationship]
- fact: the specific fact to remember
- confidence: 0.0-1.0
Only extract clear, explicit facts. Do not infer or assume.
```

### Acceptance Criteria
- [ ] Memory extracted from conversations within 60 seconds
- [ ] Memory viewer page shows all stored memories by category
- [ ] User can edit or delete any memory entry
- [ ] Relevant memories injected into AI context for each query
- [ ] Memory search via semantic similarity
- [ ] Import/export memory as JSON
- [ ] Memory confidence scores visible to user

---

## 8. Daily Brief

### Description
Every morning, AI LifeOS generates a personalized daily briefing summarizing the user's day ahead — upcoming meetings, due tasks, important reminders, recent news/weather, and a motivational insight based on user goals.

### User Stories

- As a user, I want to receive a morning summary at 7am so that I can start my day with full context.
- As a user, I want the daily brief to include my top tasks and meetings so that I can plan my morning quickly.
- As a user, I want the AI to include relevant insights based on my goals so that I stay aligned with my priorities.

### Example Daily Brief

```
☀️ Good morning, Alex! Here's your brief for Tuesday, October 15.

📅 TODAY'S SCHEDULE
• 9:00 AM — Team standup (30 min)
• 2:00 PM — Product review with Sarah and Mike (1 hr)
• 4:30 PM — 1:1 with manager (45 min)

✅ TOP PRIORITIES
• 🔴 URGENT: Submit tax return documents (due today)
• 🟡 HIGH: Finish API design document (due tomorrow)
• 🟢 MEDIUM: Review pull requests from the team

💰 FINANCIAL PULSE
• Monthly spend so far: $1,240 / $2,000 budget (62%)
• 3 days until rent payment ($1,200)

🎯 GOAL CHECK-IN
• Fitness goal: You've exercised 3/5 times this week — great progress!
• Learning goal: You haven't reviewed your Python notes in 5 days

💡 INSIGHT
Based on your workload, today looks intense. Consider blocking time 
after lunch for focused work before the afternoon meetings.
```

### Generation Pipeline

```
Celery scheduled task (configurable time, default 7:00 AM)
  → Fetch today's calendar events
  → Fetch overdue + today's tasks
  → Fetch recent expenses + budget status
  → Fetch user goals from AI memory
  → Compose brief via LLM with structured data context
  → Store in daily_briefs table
  → Send push notification to user
```

### Acceptance Criteria
- [ ] Daily brief generated daily at configured time (default 7:00 AM)
- [ ] Brief includes calendar, tasks, expenses, and goal summary
- [ ] Push notification sent on brief generation
- [ ] Brief viewable from dashboard and dedicated page
- [ ] User can configure daily brief time and sections
- [ ] Brief stored with history (viewable past briefs)
- [ ] Brief generation can be triggered manually

---

## 9. Expense Tracker

### Description
A personal finance tracker with AI-powered auto-categorization, budget management, and monthly spending reports. The AI can answer natural language questions about spending patterns and provide financial insights.

### User Stories

- As a user, I want to log an expense by saying "spent $45 at Whole Foods" so that tracking is frictionless.
- As a user, I want expenses automatically categorized so that I don't have to manually tag every transaction.
- As a user, I want to see a monthly spending report so that I understand my patterns.
- As a user, I want to set budgets per category so that I get warnings when overspending.

### Example Interactions

```
Input: "Coffee at Starbucks $6.50"
Auto-categorized as: Food & Drink > Coffee
Merchant: Starbucks
Amount: $6.50

Input: "Monthly Spotify subscription renewed"  
Auto-categorized as: Entertainment > Subscriptions
Merchant: Spotify
Amount: [extracted from recent transaction or prompted]

AI Query: "How much have I spent on subscriptions this month?"
AI: "This month you've spent $87 on subscriptions:
     • Spotify: $10.99
     • Netflix: $15.99
     • GitHub Pro: $4
     • Notion: $8
     • Other: $48.02"
```

### Categories

```
Housing: Rent, Mortgage, Utilities, Internet
Food: Groceries, Restaurants, Coffee, Takeaway
Transportation: Gas, Public Transit, Uber/Lyft, Parking
Healthcare: Doctor, Pharmacy, Insurance
Entertainment: Streaming, Games, Movies, Events
Shopping: Clothing, Electronics, Home goods
Education: Courses, Books, Conferences
Personal: Gym, Haircut, Personal care
Subscriptions: Software, Media, Services
Other: Miscellaneous
```

### Acceptance Criteria
- [ ] Add expense via form or natural language
- [ ] AI auto-categorizes expenses with >85% accuracy
- [ ] Monthly spending dashboard with charts
- [ ] Budget setting per category with alerts
- [ ] Export expenses as CSV
- [ ] AI can answer natural language finance queries
- [ ] Recurring expense detection and alerts

---

## 10. Voice Assistant

### Description
A microphone-based voice interface that captures spoken commands, transcribes them using Whisper (locally), and processes them through the AI Assistant. Supports all the same commands as text input with hands-free operation.

### User Stories

- As a user, I want to press a hotkey and speak a command so that I can add tasks without typing.
- As a user, I want my voice transcribed locally so that my audio data stays private.
- As a user, I want the AI to read its response aloud so that I can use it hands-free.

### Example Interactions

```
[User presses Ctrl+Space]
[Recording indicator appears]
User: "Add task: pick up dry cleaning on Saturday"
[Transcription appears]
AI processes and confirms: "Added task: Pick up dry cleaning, due Saturday."
[Optional TTS reads the confirmation]
```

### Technical Flow

```
Microphone input (Web Audio API)
  → Capture audio as WebM/WAV
  → Send to backend
  → Whisper.cpp (local) transcription
  → Text forwarded to AI Assistant
  → AI response returned
  → Optional: Text-to-speech (pyttsx3 or browser TTS)
```

### Acceptance Criteria
- [ ] Voice recording triggered by keyboard shortcut
- [ ] Transcription visible to user before processing
- [ ] Same AI capabilities as text interface
- [ ] Works in Chrome and Firefox
- [ ] Microphone permissions handled gracefully
- [ ] Processing indicator visible during transcription
- [ ] Fallback to text input if microphone unavailable

---

## 11. Knowledge Base

### Description
A personal knowledge aggregator that ingests content from YouTube videos (transcripts), web articles, GitHub repositories, blog posts, and research papers. All content is processed, embedded, and made searchable and chatworthy.

### User Stories

- As a user, I want to save a YouTube video and ask questions about its content so that I don't have to watch the full video.
- As a user, I want to save articles and have them searchable so that I can retrieve information I've read.
- As a user, I want to organize knowledge by topic so that I can build a structured resource library.

### Example Interactions

```
User saves: https://www.youtube.com/watch?v=abc123 "React 19 Features"
System:
  → Fetches YouTube transcript
  → Chunks and embeds transcript
  → Extracts: title, author, duration, key topics
  
User: "What new hooks were introduced in React 19?"
AI: "Based on the YouTube video 'React 19 Features' you saved:
     React 19 introduces the following new hooks:
     1. useFormStatus — tracks form submission state
     2. useFormState — manages form state with server actions
     3. useOptimistic — enables optimistic UI updates
     These are demonstrated around the 12-minute mark in the video."
```

### Supported Source Types

| Source Type | Method | Extracted Data |
|-------------|--------|----------------|
| YouTube | youtube-transcript API | Transcript, title, duration, chapters |
| Web Article | readability.js + BeautifulSoup | Title, content, author, date |
| GitHub Repo | GitHub API | README, description, languages, stars |
| PDF | PyMuPDF | Full text content |
| Blog Post | Web scraper | Title, content, author |
| Research Paper | arXiv API / PDF | Abstract, sections, references |

### Acceptance Criteria
- [ ] Save content by pasting URL
- [ ] YouTube transcripts fetched and indexed within 30 seconds
- [ ] Web articles scraped and indexed within 30 seconds
- [ ] Knowledge items tagged and categorized automatically
- [ ] Full-text and semantic search across knowledge base
- [ ] Chat with any saved content
- [ ] Collections/folders for organizing knowledge items
- [ ] Content viewable in reader mode within the app
