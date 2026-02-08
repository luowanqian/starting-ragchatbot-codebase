# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Retrieval-Augmented Generation (RAG) system for querying course materials. It's a full-stack Python web application with:
- **Backend**: FastAPI server with ChromaDB vector storage, Anthropic Claude AI, and semantic search
- **Frontend**: Static HTML/CSS/JavaScript interface
- **Data**: Course documents in `docs/` directory as structured text files

## Architecture

### Core Components

1. **RAGSystem** (`backend/rag_system.py`): Main orchestrator that coordinates:
   - Document processing via `DocumentProcessor`
   - Vector storage via `VectorStore`
   - AI generation via `AIGenerator`
   - Session management via `SessionManager`
   - Tool-based search via `ToolManager`

2. **Vector Storage** (`backend/vector_store.py`): Uses ChromaDB with:
   - Two collections: `course_metadata` and `course_content`
   - SentenceTransformer embeddings (`all-MiniLM-L6-v2` by default)
   - Persistent storage in `./chroma_db`

3. **Document Processing** (`backend/document_processor.py`): Parses course text files with structure:
   ```
   Course Title: [title]
   Course Link: [url]
   Course Instructor: [name]

   Lesson [N]: [title]
   Lesson Link: [url]
   [content...]
   ```

4. **AI Generation** (`backend/ai_generator.py`): Interfaces with Anthropic Claude API with:
   - Tool-based search integration (one search per query maximum)
   - Conversation history support
   - Strict system prompt for educational responses

5. **API Layer** (`backend/app.py`): FastAPI application with endpoints:
   - `POST /api/query`: Process user questions
   - `GET /api/courses`: Get course statistics
   - Serves frontend from `../frontend`

### Data Flow
1. Course documents in `docs/` → DocumentProcessor → Course/CourseChunk objects
2. Course metadata → VectorStore.course_metadata collection
3. Course content chunks → VectorStore.course_content collection
4. User query → ToolManager.search_tool → VectorStore semantic search
5. Search results + conversation history → AIGenerator → Response

## Development Commands

### Setup
```bash
# Install uv (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Set up environment (copy .env.example to .env)
cp .env.example .env
# Edit .env to add ANTHROPIC_API_KEY
```

### Running the Application
```bash
# Quick start (uses run.sh)
chmod +x run.sh
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Application URLs
- Web Interface: `http://localhost:8000`
- API Documentation: `http://localhost:8000/docs`

### Adding Course Documents
Place structured text files in `docs/` directory. Files should follow the format shown in `docs/course1_script.txt`. The system automatically loads documents on startup.

## Configuration

Key settings in `backend/config.py`:
- `CHUNK_SIZE`: 800 characters (text chunking for vector storage)
- `CHUNK_OVERLAP`: 100 characters (overlap between chunks)
- `MAX_RESULTS`: 5 (maximum search results)
- `MAX_HISTORY`: 2 (conversation history messages)
- `EMBEDDING_MODEL`: "all-MiniLM-L6-v2"
- `ANTHROPIC_MODEL`: "claude-sonnet-4-20250514"

## File Structure

```
├── backend/                    # FastAPI application
│   ├── app.py                 # Main FastAPI app and endpoints
│   ├── config.py              # Configuration settings
│   ├── rag_system.py          # Main RAG orchestrator
│   ├── vector_store.py        # ChromaDB vector storage
│   ├── document_processor.py  # Course document parsing
│   ├── ai_generator.py        # Anthropic Claude integration
│   ├── session_manager.py     # Conversation session management
│   ├── search_tools.py        # Tool-based search implementation
│   └── models.py              # Pydantic data models
├── frontend/                  # Static web interface
│   ├── index.html            # Main HTML page
│   ├── style.css             # CSS styles
│   └── script.js             # Frontend JavaScript
├── docs/                      # Course documents (text files)
├── pyproject.toml            # Python dependencies (uv)
├── run.sh                    # Startup script
└── .env.example              # Environment template
```

## Key Patterns

### Tool-Based Search
The system uses a tool-based approach where:
- AI decides when to search (course-specific questions only)
- One search maximum per query
- Search results are synthesized into responses without meta-commentary

### Session Management
- Each user gets a session ID for conversation history
- History limited to `MAX_HISTORY` messages
- Sessions stored in memory (not persistent)

### Vector Storage Organization
- `course_metadata`: Course titles, links, instructors for analytics
- `course_content`: Text chunks with course_title and lesson_number metadata

## Notes for Development

1. **Document Format**: Course files must follow the exact structure with "Course Title:", "Course Link:", "Lesson [N]:" headers
2. **API Key**: Requires `ANTHROPIC_API_KEY` in `.env` file
3. **ChromaDB**: Vector database persists in `backend/chroma_db/` directory
4. **CORS**: Configured to allow all origins for development
5. **Cache Headers**: Frontend served with no-cache headers for development