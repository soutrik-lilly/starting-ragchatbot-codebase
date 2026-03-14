# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**Course Materials RAG System** - A full-stack web application that implements Retrieval-Augmented Generation (RAG) to answer questions about course materials. It combines semantic search with Claude AI to provide intelligent, context-aware responses.

---

## Architecture

### High-Level Components

1. **Backend (FastAPI)** - REST API server that orchestrates the RAG system
   - Located: `backend/`
   - Main entry: `backend/app.py`
   - Runs on port 8000 by default

2. **Frontend (Vanilla JS)** - Single-page web interface
   - Located: `frontend/`
   - Files: `index.html`, `script.js`, `style.css`
   - Served by FastAPI from the backend

3. **RAG System** - Core retrieval and generation logic
   - **RAGSystem** (`rag_system.py`) - Main orchestrator that coordinates all components
   - **DocumentProcessor** (`document_processor.py`) - Parses course files and chunks text
   - **VectorStore** (`vector_store.py`) - ChromaDB wrapper for semantic search
   - **AIGenerator** (`ai_generator.py`) - Claude API integration with tool use
   - **SessionManager** (`session_manager.py`) - Manages conversation history per user session
   - **ToolManager** (`search_tools.py`) - Defines and manages AI tools

### Data Flow

1. User submits a query via frontend
2. Frontend sends to `/api/query` endpoint
3. Backend passes to RAGSystem.query()
4. AIGenerator uses Claude with tools enabled
5. Claude invokes CourseSearchTool to search vector store via VectorStore
6. Results returned to frontend with sources
7. Conversation history tracked per session_id

### Key Data Models

- **Course** - Course metadata (title, instructor, lessons)
- **Lesson** - Lesson information with number and link
- **CourseChunk** - Text chunk for vector storage (content, course_title, lesson_number)
- **SearchResults** - Wrapper for ChromaDB query results

---

## Common Commands

### Setup

```bash
# Install dependencies with uv (always in this order)
uv lock
uv sync

# Set up environment variables
cp .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY
```

### Running

```bash
# Quick start (runs backend server on port 8000)
./run.sh

# Manual startup from backend directory
cd backend && uv run uvicorn app:app --reload --port 8000

# Access the application
# Web UI: http://localhost:8000
# API docs: http://localhost:8000/docs
```

### Loading Course Materials

The backend automatically loads documents from `docs/` folder at startup. Supported formats: `.pdf`, `.docx`, `.txt`

To programmatically add courses:

```python
from rag_system import RAGSystem
from config import config

rag = RAGSystem(config)
courses, chunks = rag.add_course_folder("docs", clear_existing=False)
```

---

## Configuration

All settings are in `backend/config.py`:

- **ANTHROPIC_MODEL** - Claude model to use (default: `claude-sonnet-4-6`)
- **EMBEDDING_MODEL** - Sentence transformer for embeddings (currently `all-MiniLM-L6-v2`)
- **CHUNK_SIZE** - Text chunk size for semantic search (800 chars)
- **CHUNK_OVERLAP** - Overlap between chunks (100 chars)
- **MAX_RESULTS** - Max search results per query (5)
- **MAX_HISTORY** - Conversation messages to remember (2)
- **CHROMA_PATH** - Vector database location (`./chroma_db`)

---

## Tech Stack

- **Backend**: FastAPI 0.116.1, Uvicorn 0.35.0, Python 3.13+
- **Vector DB**: ChromaDB 1.0.15 with Sentence Transformers 5.0.0
- **AI**: Anthropic Claude SDK 0.58.2
- **Package Manager**: uv (Python)
- **Frontend**: Vanilla JavaScript, HTML, CSS

---

## API Endpoints

- `POST /api/query` - Submit a query with optional session_id, returns answer with sources
- `GET /api/courses` - Get course statistics and list
- `GET /` - Serves frontend (static files)
- `GET /docs` - Interactive API documentation (Swagger)

---

## Important Notes

- **Python Version**: Requires Python 3.13+ (specified in `.python-version`)
- **CORS**: Enabled for all origins in development
- **API Key**: Must set `ANTHROPIC_API_KEY` in `.env` before running
- **ChromaDB**: Vector database persists in `chroma_db/` directory
- **Tool Use**: Claude uses the CourseSearchTool to semantically search course content - searches are automatic and transparent to the user
- **Sessions**: Each user session maintains conversation history for context-aware responses

---

## Agentic Coding Rules

These rules govern how Claude Code must behave in this project. Follow them precisely.

### Workflow — Package Management

- **DO**: Always run `uv lock` → `uv sync` → `uv run <command>` in that exact order
- **DO**: Use `uv run` to execute scripts, tests, and servers (never `python` directly)
- **DO NOT**: Use `pip install`, `python -m`, or any package manager other than `uv`
- **DO NOT**: Skip `uv lock` even for minor dependency changes

### Workflow — Model Selection

- **DO**: Always use `claude-sonnet-4-6` (sonnet-latest) as the default model for all AI operations
- **DO NOT**: Use outdated model IDs like `claude-sonnet-4-20250514` in new code

### Workflow — Git & Version Control

- **DO**: Create new commits rather than amending existing ones
- **DO**: Stage specific files by name (never `git add -A` or `git add .`)
- **DO**: Investigate before deleting unfamiliar files, branches, or configuration
- **DO NOT**: Force push, reset --hard, or run destructive git commands without explicit user confirmation
- **DO NOT**: Skip hooks (`--no-verify`) or bypass signing
- **DO NOT**: Commit `.env` files, secrets, API keys, or credentials

---

## Coding Standards

### Type Safety & Validation

- **DO**: Use type hints on all function signatures using the `typing` module (List, Dict, Optional, Tuple, Any)
- **DO**: Use `Optional[Type] = None` for all nullable fields
- **DO**: Specify return types on every function, especially tuples (e.g., `-> Tuple[Course, int]`)
- **DO**: Use Pydantic `BaseModel` for all request/response data structures
- **DO NOT**: Leave function arguments or return types untyped
- **DO NOT**: Use bare `dict` or `list` without type parameters in signatures

### Error Handling & Safety

- **DO**: Wrap all external API calls, file I/O, and third-party library calls in `try-except`
- **DO**: Return sensible defaults on failure (e.g., `return None, 0`) rather than crashing
- **DO**: Log descriptive error messages including file name or operation context
- **DO**: Validate that `ANTHROPIC_API_KEY` is non-empty before initializing any Anthropic client
- **DO**: Check `os.path.exists()` before reading from or writing to any file path
- **DO NOT**: Swallow exceptions silently with bare `except: pass`
- **DO NOT**: Let unhandled exceptions propagate through API endpoints — wrap in `HTTPException`
- **DO NOT**: Raise generic exceptions; use the most specific exception type available

### Configuration & Environment

- **DO**: Keep all tuneable parameters in `backend/config.py` using `@dataclass`
- **DO**: Load environment variables with `load_dotenv()` at module level, then `os.getenv()` with defaults
- **DO**: Add inline comments explaining what each config value controls
- **DO NOT**: Hardcode numeric values, model names, paths, or limits anywhere in source code
- **DO NOT**: Access environment variables directly (`os.environ["KEY"]`) without a fallback default
- **DO NOT**: Duplicate config values across multiple files

### Data Models & Structure

- **DO**: Add a one-line docstring to every class describing its purpose
- **DO**: Use inline comments for non-obvious fields
- **DO**: Use nested Pydantic models to reflect real-world hierarchies (e.g., Course → List[Lesson])
- **DO**: Include metadata fields that trace origin (e.g., `course_title`, `lesson_number` on chunks)
- **DO NOT**: Create flat, unstructured dicts where a typed model would be more precise
- **DO NOT**: Mix domain logic into data model classes

### API Design

- **DO**: Define explicit Pydantic request and response models for every endpoint
- **DO**: Use `async def` for all FastAPI route handlers
- **DO**: Raise `HTTPException(status_code=500, detail=str(e))` to surface errors cleanly
- **DO**: Configure `CORSMiddleware` and `TrustedHostMiddleware` for every new app
- **DO NOT**: Return raw dicts from endpoints — always return a typed response model
- **DO NOT**: Expose internal stack traces or sensitive details in HTTP error responses

### Performance & Efficiency

- **DO**: Define system prompts as class-level constants (`SYSTEM_PROMPT = "..."`) to avoid rebuilding per call
- **DO**: Pre-build reusable API parameter dicts in `__init__` (e.g., `self.base_params`)
- **DO**: Use f-strings for readable string formatting
- **DO**: Use ternary expressions for simple conditionals (e.g., `x if condition else y`)
- **DO NOT**: Reconstruct large strings or dicts inside hot paths or loops
- **DO NOT**: Make redundant API calls when results can be reused or cached

### Code Organization & Architecture

- **DO**: Give each module exactly one responsibility (models, config, vector store, AI generator, etc.)
- **DO**: Inject config into classes via constructor — never create or import `config` inside a class
- **DO**: Initialize all sub-components in `__init__` and register tools explicitly
- **DO**: Use `ToolManager` to centrally register and dispatch tools
- **DO NOT**: Put business logic inside API route handlers — delegate to the RAGSystem layer
- **DO NOT**: Create circular imports between modules

### Batch Operations & Idempotency

- **DO**: Check for existing data before inserting (e.g., track `existing_course_titles` in a set)
- **DO**: Support both `clear_existing=True` (full rebuild) and `False` (incremental append) modes
- **DO**: Print progress logs during bulk operations (e.g., courses loaded, chunks added)
- **DO NOT**: Re-process or re-embed documents that already exist in the vector store
- **DO NOT**: Assume the database is always empty at startup

### State & Session Management

- **DO**: Use `SessionManager` for all conversation history — never store state in global variables
- **DO**: Respect `MAX_HISTORY` to limit memory footprint and prevent context bloat
- **DO**: Auto-create a new session ID when none is provided by the client
- **DO NOT**: Share session state between different users or requests
- **DO NOT**: Store secrets, API keys, or credentials in session data

### Naming Conventions

- **DO**: Use `PascalCase` for classes (e.g., `RAGSystem`, `AIGenerator`, `DocumentProcessor`)
- **DO**: Use `snake_case` for functions, methods, and variables (e.g., `add_course_document`)
- **DO**: Use `UPPERCASE_WITH_UNDERSCORES` for constants (e.g., `SYSTEM_PROMPT`, `CHUNK_SIZE`)
- **DO**: Prefix internal helpers with underscore (e.g., `_build_messages`)
- **DO**: Use full descriptive names — never abbreviate (e.g., `course_title` not `c_ttl`)
- **DO NOT**: Use single-letter variable names outside of loop indices

### Documentation

- **DO**: Write docstrings for every public function with `Args:` and `Returns:` sections
- **DO**: Use type hints as inline documentation — they reduce the need for comments
- **DO**: Comment only non-obvious logic; do not comment self-evident code
- **DO NOT**: Leave TODO/FIXME comments in committed code — create a tracked issue instead
- **DO NOT**: Add docstrings or comments to code you did not write or modify

---

## Security Rules

- **DO**: Validate all user-supplied input at system boundaries (API endpoints, file uploads)
- **DO**: Follow OWASP Top 10 guidelines — prevent SQL injection, XSS, command injection
- **DO**: Store secrets only in `.env` (never in source code or config files checked into git)
- **DO NOT**: Log sensitive data such as API keys, user queries with PII, or tokens
- **DO NOT**: Use `eval()`, `exec()`, or `subprocess` with unsanitized user input
- **DO NOT**: Commit `.env`, credentials, or private keys under any circumstances

---

## Agentic Behaviour Rules

These rules govern how Claude Code must operate when acting autonomously.

### Scope & Restraint

- **DO**: Make only the changes explicitly requested — do not refactor surrounding code opportunistically
- **DO**: Prefer editing existing files over creating new ones
- **DO**: Confirm with the user before taking any irreversible or high-blast-radius action
- **DO NOT**: Add extra features, error handling, or abstractions beyond what was asked
- **DO NOT**: Add docstrings, comments, or type annotations to code you did not change
- **DO NOT**: Create helper utilities for operations that only happen once

### Before Writing Code

- **DO**: Read the relevant file(s) before editing — never modify code you haven't seen
- **DO**: Understand the existing patterns in the file before introducing new ones
- **DO**: Check if similar functionality already exists before creating something new
- **DO NOT**: Guess at function signatures, class names, or module paths — look them up first

### Parallelism & Tool Use

- **DO**: Make multiple independent tool calls in a single message when they have no dependencies
- **DO**: Use the Explore agent for broad codebase research; use Grep/Glob for targeted lookups
- **DO NOT**: Serially chain tool calls that could run in parallel
- **DO NOT**: Use `bash grep` or `bash find` when the dedicated Grep/Glob tools are available

### Irreversible Actions — Always Confirm First

- Deleting files, branches, or database contents
- Force pushing or resetting git history
- Sending external messages (Slack, email, GitHub comments)
- Modifying CI/CD pipelines or shared infrastructure
- Dropping or migrating database tables

### When Blocked

- **DO**: Diagnose root cause before retrying — never loop the same failing command
- **DO**: Propose an alternative approach or ask the user for direction
- **DO NOT**: Use `--no-verify`, `--force`, or destructive shortcuts to bypass blockers
- **DO NOT**: Sleep-poll or retry the same failed operation in a loop
