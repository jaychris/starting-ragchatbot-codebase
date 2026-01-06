# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a RAG (Retrieval-Augmented Generation) chatbot for course materials. Users query course content through a web interface, and the system uses semantic search + Claude AI to generate answers.

## Commands

```bash
# Install dependencies
uv sync

# Run the server (from project root)
cd backend && uv run uvicorn app:app --reload --port 8000

# Or use the convenience script
./run.sh
```

The app runs at http://localhost:8000 with API docs at http://localhost:8000/docs.

## Architecture

### Query Flow

1. **Frontend** (`frontend/script.js`) → POST `/api/query` with `{query, session_id}`
2. **FastAPI** (`backend/app.py:56`) → Routes to `RAGSystem.query()`
3. **RAGSystem** (`backend/rag_system.py:102`) → Orchestrates the pipeline:
   - Gets conversation history from `SessionManager`
   - Calls `AIGenerator.generate_response()` with tools
4. **AIGenerator** (`backend/ai_generator.py:43`) → Two-call pattern with Claude:
   - First call: Claude decides whether to use `search_course_content` tool
   - If tool used: executes search, then second call to synthesize answer
5. **CourseSearchTool** (`backend/search_tools.py:52`) → Executes vector search
6. **VectorStore** (`backend/vector_store.py:61`) → ChromaDB query with filters

### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| `RAGSystem` | `rag_system.py` | Main orchestrator, initializes all components |
| `AIGenerator` | `ai_generator.py` | Claude API calls, tool execution loop |
| `VectorStore` | `vector_store.py` | ChromaDB wrapper, two collections: `course_catalog` + `course_content` |
| `DocumentProcessor` | `document_processor.py` | Parses course files, sentence-based chunking |
| `CourseSearchTool` | `search_tools.py` | Tool definition for Claude, formats search results |
| `SessionManager` | `session_manager.py` | Per-session conversation history |

### Document Format

Course documents in `docs/` follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [name]

Lesson 0: Introduction
Lesson Link: [url]
[content...]

Lesson 1: Topic
[content...]
```

### Configuration

All settings in `backend/config.py`:
- `CHUNK_SIZE`: 800 chars
- `CHUNK_OVERLAP`: 100 chars
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation exchanges
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514

### Data Models

Defined in `backend/models.py`:
- `Course` → title (unique ID), lessons list, instructor, link
- `Lesson` → number, title, link
- `CourseChunk` → content with course/lesson metadata for vector storage

## Environment

Requires `.env` file with:
```
ANTHROPIC_API_KEY=your_key_here
```
