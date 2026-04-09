# Copilot Instructions — Audio Recording Web Application

## Project Overview

A web-based audio recording application where **Users** can record, playback, manage, and
download audio recordings directly from the browser, and **Admins** manage users and oversee
stored recordings.

The MVP delivers a fully functional browser-based audio recorder with waveform visualization,
recording management, and optional AI-powered transcription via GitHub Models.
The SDLC is driven by custom Copilot agents that chain together to produce
requirements → design → backlog → **orchestrated wave execution** → UI → tests → security review.

The pipeline is: `@1-requirement-agent` → `@2-plan-and-design-agent` → `@3-epic-and-tasks-agent` → **`sdlc-build-planner` workflow (automated)** → `@4-develop-agent` / `@5-ui-develop-agent` (wave by wave) → `@6-automation-test-agent` → `@7-security-agent`.

### Requirements Source

The authoritative product requirements live in `audio-recording-app-requirements.md` at the
repo root. All agents and contributors **must** read this document for scope, user journeys,
functional requirements, data entities, and acceptance criteria.

## Users & Roles

| Role      | Description                                                                 |
|-----------|-----------------------------------------------------------------------------|
| **User**  | Records audio, plays back recordings, manages (rename/delete) own recordings, downloads files |
| **Admin** | Manages users, views all recordings, configures system settings, monitors storage |

Role-based access control is required. Admins can perform all user actions plus
user management, system configuration, and storage monitoring.
Users can only access their own recordings.

## Domain Model (Core Entities)

- **User** — id, name, email, role, createdAt
- **Recording** — id, userId, title, filename, fileSize, duration, mimeType, status (recording/saved/deleted), createdAt, updatedAt
- **RecordingTag** — id, recordingId, tag
- **TranscriptionRequest** — id, recordingId, requesterId, model, status (pending/processing/completed/failed), createdAt
- **Transcription** — id, recordingId, transcriptionRequestId, text, language, confidence, createdAt

## Recording Workflow

`User opens recorder → starts recording → real-time waveform visualization → stops recording → preview playback → save with title/tags → manage in library`

Each recording is stored as a WebM/WAV audio file. Users can record, pause/resume, stop, preview,
save, rename, tag, download, and delete recordings. AI transcription is optional and starts as
**pending** until processed.

## Tech Stack & Conventions

- **Language**: Python 3.11+
- **Backend Framework**: FastAPI
- **Data Models**: Pydantic v2
- **Database**: SQLite (zero-config, file-based for MVP)
- **Frontend**: Vanilla HTML/CSS/JS with Jinja2 templates (no frameworks)
- **Audio API**: Web Audio API + MediaRecorder API (browser-native)
- **Testing**: pytest + httpx AsyncClient + respx/unittest.mock for external API mocks
- **AI Backend**: GitHub Models API (GPT-4o / Whisper) for transcription
- **Project Structure**:
  - `src/` — application source code (backend + frontend static/templates)
  - `tests/` — test suite
  - `docs/` — SDLC artifacts (requirements/, design/, testing/)
  - `templates/` — SDLC document templates
  - `backlog/` — work items (epics/, stories/, tasks/)

## API / Service Boundaries

The backend should be organized into these service modules:

| Service               | Responsibility                                                  |
|-----------------------|-----------------------------------------------------------------|
| Auth                  | Sign-in, role-based access control                              |
| Recording Management  | Upload/download/delete recordings, metadata CRUD, file storage  |
| Audio Processing      | Waveform data generation, format conversion, duration detection |
| AI Transcription      | GitHub Models integration, speech-to-text, transcription status |
| User Management       | CRUD users, admin controls, storage quota tracking              |

REST endpoints live under `/api/v1/`. Frontend page routes return Jinja2 templates at root paths.

## Audio Storage & Handling

1. Audio is captured in the browser via MediaRecorder API (WebM/Opus or WAV).
2. Recorded blobs are uploaded to the backend via multipart form data.
3. Files are stored on the local filesystem under `uploads/` with unique filenames.
4. File metadata (size, duration, MIME type) is extracted server-side and stored in the database.
5. Streaming playback is supported via HTTP range requests.
6. Admins can configure max recording duration and per-user storage quotas.

## Coding Standards

- Use type hints on all function signatures and variables where non-obvious.
- Define request/response schemas as Pydantic models (never raw dicts).
- Use `async def` for I/O-bound endpoints; sync is fine for pure computation.
- Raise `fastapi.HTTPException` with appropriate status codes for error handling.
- Load configuration from environment variables via `pydantic-settings` — never hardcode secrets.
- Add docstrings to all public functions and classes.
- Keep functions small and single-purpose.
- Validate and sanitize all file uploads (MIME type, size limits) to prevent malicious uploads.
- Sanitize all user-supplied text (titles, tags) to prevent XSS.

## Agent Workflow Rules

- SDLC document templates live in `templates/` — always start from a template.
- Generated artifacts go under `docs/`:
  - `docs/requirements/` — BRD, functional specs
  - `docs/design/` — architecture, API design, data models
  - `docs/testing/` — test plans, test cases, security review
- Backlog items go under `backlog/`:
  - `backlog/epics/` — high-level features
  - `backlog/stories/` — user stories
  - `backlog/tasks/` — implementation tasks
- Every artifact **must** trace back to a BRD requirement ID (`BRD-xxx`).
- Update `docs/change-log.md` when making key decisions or significant changes.
- Agents must read `audio-recording-app-requirements.md` to understand full scope.

## GitHub Models Integration

- **Auth**: Use environment variable `GITHUB_MODELS_API_KEY`.
- **Endpoint**: Configured via environment variable `GITHUB_MODELS_ENDPOINT`.
- **Preferred model**: Whisper for transcription, GPT-4o for summarization tasks.
- Always handle rate limits with exponential backoff.
- Wrap API calls in try/except and return meaningful error responses.
- Never log or expose API keys in output or error messages.
- Store transcription results with request metadata for audit.
- Design the transcription service as a pluggable module (MCP-ready for future).

## Non-Functional Requirements

- **Usability**: User starts recording within 1 click from dashboard. Responsive for desktop and tablet. Real-time waveform feedback during recording.
- **Performance**: Non-AI API calls < 2s. Audio upload/download streamed efficiently. AI transcription may be async if long-running.
- **Security**: RBAC enforced on all endpoints. Secrets in env vars only. File upload validation (type, size). XSS prevention on user input. CORS scoped appropriately.
- **Reliability**: Recordings never lost on page refresh (auto-save on stop). AI failures return retryable error states. Recordings playable even if AI services are down.
- **Observability**: Log auth events, recording uploads/deletes, transcription calls. Capture transcription errors and latency.

## Do / Avoid

### Do

- Use templates from `templates/` for all SDLC documents.
- Read `audio-recording-app-requirements.md` for authoritative scope and acceptance criteria.
- Maintain requirement ID traceability (`BRD-xxx`) across all artifacts.
- Write tests for every new endpoint and service function.
- Keep dependencies minimal and justified.
- Validate file uploads (MIME type whitelist, size limits) before storing.

### Avoid

- Hardcoding API keys, secrets, or environment-specific values.
- Adding complexity beyond MVP scope — keep it simple.
- Introducing frameworks or libraries not needed for the MVP.
- Committing generated credentials or `.env` files.
- Skipping error handling on external API calls.
- Storing audio files without proper validation and sanitization.
