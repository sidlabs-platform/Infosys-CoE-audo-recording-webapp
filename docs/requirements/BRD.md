# Business Requirements Document (BRD)

| Field       | Value                                          |
|-------------|-------------------------------------------------|
| **Title**   | Audio Recording Web Application                 |
| **Version** | 1.0                                             |
| **Date**    | 2026-04-09                                      |
| **Author**  | SDLC Requirement Agent (Copilot)                |
| **Status**  | Draft                                           |

---

## 1. Executive Summary

### 1.1 Project Overview

The Audio Recording Web Application is a browser-based platform that enables users to record audio from any application running on their system, play back recordings, manage an audio library, and optionally transcribe recordings using AI. Built with Python 3.11+ and FastAPI on the backend and vanilla HTML/CSS/JavaScript on the frontend, the MVP delivers a fully functional audio recorder with real-time waveform visualization, recording management (rename, tag, download, delete), and AI-powered transcription via the GitHub Models API (Whisper for speech-to-text, GPT-4o for summarization). The application uses SQLite for zero-configuration persistence and stores audio files on the local filesystem, making it suitable for individual and small-team use.

### 1.2 Business Objectives

- **BO-1**: Provide a simple, browser-based audio recording tool that captures system audio without requiring third-party desktop software.
- **BO-2**: Deliver an intuitive recording management library where users can organize, search, tag, rename, download, and delete recordings.
- **BO-3**: Enable optional AI-powered transcription and summarization of recordings through GitHub Models API integration.
- **BO-4**: Support role-based access control with User and Admin roles for multi-user deployment.
- **BO-5**: Maintain a minimal, zero-configuration deployment footprint using SQLite and local filesystem storage.

### 1.3 Success Metrics / KPIs

| Metric ID | Metric                                       | Target                    | Measurement Method                                                       |
|-----------|----------------------------------------------|---------------------------|--------------------------------------------------------------------------|
| KPI-001   | Time to first recording                      | ≤ 1 click from dashboard  | UI interaction audit — user reaches record state in a single click       |
| KPI-002   | Non-AI API response time                     | < 2 seconds (p95)         | Server-side request latency logging                                      |
| KPI-003   | Recording data integrity                     | Zero data loss on refresh | Automated test: recording persists after simulated browser close         |
| KPI-004   | Audio upload success rate                    | ≥ 99 %                    | Upload success/failure ratio from server logs                            |
| KPI-005   | AI transcription completion rate             | ≥ 95 % (when API is up)   | Transcription request status tracking                                    |
| KPI-006   | Test coverage                                | ≥ 80 % line coverage      | pytest-cov report                                                        |

---

## 2. Background & Context

Users frequently need to record audio from applications running on their system — meetings, webinars, podcasts, tutorials — but existing tools often require installing heavyweight desktop software, navigating complex audio routing, or paying for subscription services.

This project addresses that gap by providing a lightweight, browser-based audio recording application. Using the Web Audio API and MediaRecorder API natively available in modern browsers, the application captures audio directly, stores it server-side, and offers a full management interface. For users who need text versions of their recordings, optional AI transcription is available via the GitHub Models API.

The application is designed as an MVP with a clear path to future enhancements. It uses a simple tech stack (Python / FastAPI backend, vanilla JS frontend, SQLite database, local file storage) to minimize operational complexity while delivering a complete, usable product.

---

## 3. Stakeholders

| Name               | Role                     | Interest                                                    | Influence |
|--------------------|--------------------------|-------------------------------------------------------------|-----------|
| End Users          | Audio Recorder Users     | Easy recording, playback, and management of audio files     | High      |
| System Admin       | Application Administrator| User management, storage oversight, system configuration    | High      |
| Product Owner      | Project Sponsor          | Feature completeness, user satisfaction, MVP delivery       | High      |
| Development Team   | Developers / Engineers   | Clean architecture, testability, maintainability            | Medium    |
| Security Auditor   | Security Reviewer        | Data protection, input validation, access control           | Medium    |

---

## 4. Scope

### 4.1 In-Scope

- Browser-based audio recording using Web Audio API and MediaRecorder API
- Real-time waveform visualization during recording
- Recording lifecycle: record, pause/resume, stop, preview, save
- Recording metadata management: rename titles, add/edit/remove tags
- Recording library with list view, search by title, filter by tag
- Audio file download (WebM / WAV)
- Streaming playback via HTTP range requests
- File upload with server-side metadata extraction (size, duration, MIME type)
- Audio file storage on local filesystem under `uploads/`
- SQLite database for all persistent data
- User authentication (sign-in with credentials, session/token)
- Role-based access control (User and Admin roles)
- Admin user management (create, update, deactivate, delete users)
- Admin recording oversight (view all recordings across users)
- Admin system settings (max recording duration, per-user storage quotas)
- Admin storage monitoring (per-user and system-wide usage)
- AI transcription via Whisper model (GitHub Models API)
- AI summarization via GPT-4o model (GitHub Models API)
- Async transcription processing with status tracking
- Auto-save on recording stop to prevent data loss
- XSS prevention on user-supplied text
- File upload validation (MIME type whitelist, size limits)
- REST API under `/api/v1/` prefix
- Jinja2-rendered frontend pages

### 4.2 Out-of-Scope

- Mobile-native applications (iOS / Android)
- Video recording or screen capture
- Cloud storage backends (AWS S3, Azure Blob, GCS)
- User self-registration / public sign-up (admin creates users for MVP)
- OAuth / SSO integration with external identity providers
- Advanced audio editing (trimming, splicing, effects)
- Batch transcription of multiple recordings in a single request
- Real-time live transcription during recording
- Multi-language frontend localization
- Payment processing or subscription billing
- Horizontal scaling / multi-instance deployment

### 4.3 Assumptions

- Users access the application via modern browsers (Chrome, Firefox, Edge) that support Web Audio API and MediaRecorder API.
- The server runs on a machine with sufficient local disk space for audio file storage.
- A valid GitHub Models API key is available and configured via environment variable for AI transcription features.
- SQLite is sufficient for MVP data volumes (single server, moderate concurrent users).
- The application is deployed as a single-instance server (no horizontal scaling required for MVP).
- Users grant browser microphone / audio-capture permissions when prompted.

### 4.4 Constraints

- MVP must use SQLite as the database (zero-configuration requirement).
- All AI inference must use the GitHub Models API — no other AI/ML providers.
- Frontend must use vanilla HTML/CSS/JS with Jinja2 templates — no JavaScript frameworks (React, Vue, Angular).
- Backend must be Python 3.11+ with FastAPI.
- Audio files stored on local filesystem under `uploads/` directory.
- All secrets and configuration loaded from environment variables via `pydantic-settings`.
- Minimal dependency footprint — every dependency must be justified.

### 4.5 Dependencies

- **GitHub Models API** — Required for AI transcription (Whisper) and summarization (GPT-4o). Rate limits and availability are external dependencies.
- **Browser APIs** — Web Audio API and MediaRecorder API must be supported by the user's browser.
- **Python 3.11+** — Runtime environment for the backend.
- **Local Filesystem** — Audio file storage depends on available disk space and filesystem permissions.
- **SQLite** — Embedded database engine shipped with Python standard library.

---

## 5. Use Cases

| Use Case ID | Name                              | Description                                                                                                                | Priority    | Actors          |
|-------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------|-------------|-----------------|
| UC-001      | Record Audio                      | User opens the recorder, starts recording with real-time waveform visualization, pauses/resumes as needed, and stops.      | Must-Have   | User            |
| UC-002      | Preview and Save Recording        | After stopping, user previews the recorded audio via playback, then saves it with a title and optional tags.               | Must-Have   | User            |
| UC-003      | Browse Recording Library          | User views a list of their saved recordings with metadata (title, duration, date, tags), with search and filter.           | Must-Have   | User            |
| UC-004      | Playback Recording                | User selects a recording from the library and plays it back with streaming audio support.                                  | Must-Have   | User            |
| UC-005      | Download Recording                | User downloads a recording as a WebM or WAV file to their local machine.                                                  | Must-Have   | User            |
| UC-006      | Rename Recording                  | User renames an existing recording's title.                                                                                | Must-Have   | User            |
| UC-007      | Tag Recording                     | User adds, edits, or removes tags on a recording for organization.                                                         | Must-Have   | User            |
| UC-008      | Delete Recording                  | User deletes a recording (file and metadata removed). Confirmation prompt shown.                                           | Must-Have   | User            |
| UC-009      | Request AI Transcription          | User requests AI transcription of a recording. System sends audio to Whisper via GitHub Models API asynchronously.         | Should-Have | User            |
| UC-010      | View Transcription                | User views the completed transcription text, language, and confidence score for a recording.                               | Should-Have | User            |
| UC-011      | Request AI Summarization          | User requests an AI-generated summary of a transcription using GPT-4o via GitHub Models API.                              | Could-Have  | User            |
| UC-012      | Sign In                           | User authenticates with credentials to access the application.                                                             | Must-Have   | User, Admin     |
| UC-013      | Manage Users (Admin)              | Admin creates, updates, deactivates, or deletes user accounts and assigns roles.                                           | Must-Have   | Admin           |
| UC-014      | View All Recordings (Admin)       | Admin browses and searches all recordings across all users for oversight.                                                  | Must-Have   | Admin           |
| UC-015      | Configure System Settings (Admin) | Admin sets max recording duration, per-user storage quotas, and other system parameters.                                   | Should-Have | Admin           |
| UC-016      | Monitor Storage (Admin)           | Admin views storage usage statistics per user and system-wide.                                                             | Should-Have | Admin           |

---

## 6. Functional Requirements

### 6.1 Authentication & Authorization

| Req ID      | Description                                                                                                          | Priority    | Acceptance Criteria                                                                                                                 |
|-------------|----------------------------------------------------------------------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------|
| BRD-FR-001  | The system shall provide a sign-in endpoint that authenticates users by credentials and returns a session/token.      | Must-Have   | Given valid credentials, the user receives a token and is redirected to the dashboard. Given invalid credentials, a 401 is returned. |
| BRD-FR-002  | The system shall enforce role-based access control (RBAC) on all API endpoints, distinguishing User and Admin roles.  | Must-Have   | User-role requests to admin-only endpoints return 403. Admin-role requests succeed on all endpoints.                                 |
| BRD-FR-003  | Users shall only be able to access their own recordings via the API.                                                  | Must-Have   | A User requesting another user's recording receives 403. Requesting their own recording receives 200.                               |
| BRD-FR-004  | Admin users shall be able to perform all actions that regular Users can, plus admin-specific operations.               | Must-Have   | Admin can access any user's recordings, manage users, and configure system settings.                                                |

### 6.2 Audio Recording

| Req ID      | Description                                                                                                          | Priority    | Acceptance Criteria                                                                                                                 |
|-------------|----------------------------------------------------------------------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------|
| BRD-FR-005  | The frontend shall capture audio via the browser MediaRecorder API (WebM/Opus or WAV).                                | Must-Have   | Clicking "Record" starts audio capture. The browser requests microphone/audio permissions if not already granted.                    |
| BRD-FR-006  | The recorder shall support pause and resume during an active recording session.                                       | Must-Have   | Clicking "Pause" pauses capture; clicking "Resume" continues. The final file includes all non-paused segments.                      |
| BRD-FR-007  | The frontend shall display real-time waveform visualization during recording using the Web Audio API.                 | Must-Have   | A waveform animation is visible and updates at ≥ 30 fps while recording is active.                                                 |
| BRD-FR-008  | After stopping a recording, the user shall be able to preview playback before saving.                                 | Must-Have   | A playback control appears after "Stop" is pressed. The user can listen to the recording before deciding to save or discard.        |
| BRD-FR-009  | The recording shall be auto-saved on stop to prevent data loss if the user navigates away or the page refreshes.      | Must-Have   | If the browser is closed or refreshed after stop, the recording data is preserved and available on next visit.                       |
| BRD-FR-010  | The system shall enforce a maximum recording duration limit (admin-configurable).                                     | Should-Have | Recording automatically stops when the max duration is reached. User is notified.                                                   |

### 6.3 Recording Management

| Req ID      | Description                                                                                                          | Priority    | Acceptance Criteria                                                                                                                 |
|-------------|----------------------------------------------------------------------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------|
| BRD-FR-011  | The user shall be able to save a recording with a title and optional tags via a save form.                            | Must-Have   | After preview, user enters a title (required) and optional tags, submits, and the recording appears in their library.                |
| BRD-FR-012  | The system shall upload recorded audio blobs to the backend as multipart form data.                                   | Must-Have   | Audio blob is sent via POST multipart/form-data. Server returns 201 Created with recording metadata.                                |
| BRD-FR-013  | The backend shall store uploaded audio files on the local filesystem under `uploads/` with unique filenames.           | Must-Have   | Each file is saved with a UUID-based filename. No collisions. File path stored in database.                                         |
| BRD-FR-014  | The backend shall extract and store file metadata (size, duration, MIME type) for each uploaded recording.             | Must-Have   | After upload, the database record contains accurate fileSize (bytes), duration (seconds), and mimeType.                             |
| BRD-FR-015  | The user shall be able to rename a recording's title.                                                                 | Must-Have   | User submits a new title via the API. Title is updated in the database and reflected in the library.                                 |
| BRD-FR-016  | The user shall be able to add, edit, and remove tags on a recording.                                                  | Must-Have   | Tags can be added, modified, or deleted via the API. Tags stored in RecordingTag entity.                                            |
| BRD-FR-017  | The user shall be able to delete a recording, removing both the file and all associated metadata.                     | Must-Have   | After deletion, the audio file is removed from `uploads/`, Recording and related RecordingTag/Transcription records are deleted.     |
| BRD-FR-018  | The user shall be able to download a recording as a file (WebM or WAV).                                               | Must-Have   | Download endpoint returns the audio file with correct Content-Type and Content-Disposition headers.                                  |
| BRD-FR-019  | The system shall support streaming audio playback via HTTP range requests.                                            | Must-Have   | Playback requests with Range headers receive 206 Partial Content. Seeking works without re-downloading the entire file.             |
| BRD-FR-020  | The user shall be able to browse their recording library with title, duration, date, size, and tags.                  | Must-Have   | Library page displays all user recordings with metadata columns. Empty state shown when no recordings exist.                        |
| BRD-FR-021  | The recording library shall support search by title and filter by tags.                                               | Should-Have | Searching by partial title returns matching recordings. Filtering by tag returns only tagged recordings. Combined search+filter works.|

### 6.4 User Management (Admin)

| Req ID      | Description                                                                                                          | Priority    | Acceptance Criteria                                                                                                                 |
|-------------|----------------------------------------------------------------------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------|
| BRD-FR-022  | Admins shall be able to create new user accounts with name, email, role, and initial password.                        | Must-Have   | POST to user creation endpoint returns 201. New user can sign in with provided credentials.                                         |
| BRD-FR-023  | Admins shall be able to update user account details (name, email, role).                                              | Must-Have   | PUT/PATCH to user update endpoint modifies the user record. Changes reflected immediately.                                          |
| BRD-FR-024  | Admins shall be able to deactivate or delete user accounts.                                                           | Must-Have   | Deactivated users cannot sign in. Deleted users and their recordings are removed.                                                   |
| BRD-FR-025  | Admins shall be able to view all recordings across all users for oversight.                                           | Must-Have   | Admin recordings list returns recordings from all users with user identification. Pagination supported.                              |
| BRD-FR-026  | Admins shall be able to configure per-user storage quotas.                                                            | Should-Have | Admin sets a quota. Uploads exceeding the quota are rejected with 413 and a descriptive message.                                    |
| BRD-FR-027  | Admins shall be able to configure the maximum recording duration.                                                     | Should-Have | Admin sets max duration in settings. Frontend enforces the limit during recording.                                                  |
| BRD-FR-028  | Admins shall be able to view storage usage statistics per user and system-wide.                                       | Should-Have | Storage monitoring endpoint returns total used, per-user breakdown, and quota utilization percentages.                               |

### 6.5 AI Transcription & Summarization

| Req ID      | Description                                                                                                          | Priority    | Acceptance Criteria                                                                                                                 |
|-------------|----------------------------------------------------------------------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------|
| BRD-FR-029  | The user shall be able to request AI transcription of a saved recording.                                              | Should-Have | User clicks "Transcribe". A TranscriptionRequest is created with status "pending". Request ID returned.                             |
| BRD-FR-030  | The system shall send the audio file to the Whisper model via GitHub Models API for speech-to-text transcription.     | Should-Have | Audio file sent to configured endpoint. Whisper processes it and returns transcription text.                                         |
| BRD-FR-031  | Transcription shall be processed asynchronously with status tracking (pending → processing → completed/failed).       | Should-Have | TranscriptionRequest status transitions are tracked. User can poll for status. Final status is "completed" or "failed".              |
| BRD-FR-032  | The transcription result shall be stored with text, detected language, and confidence score.                           | Should-Have | Transcription entity contains text, language code, and confidence value. All fields populated on completion.                         |
| BRD-FR-033  | The user shall be able to view the completed transcription for a recording.                                           | Should-Have | Transcription text, language, and confidence displayed on the recording detail page.                                                 |
| BRD-FR-034  | The user shall be able to request an AI-generated summary of a transcription using GPT-4o.                            | Could-Have  | User clicks "Summarize" on a completed transcription. Text sent to GPT-4o. Summary returned and displayed.                          |
| BRD-FR-035  | AI transcription failures shall return retryable error states without affecting recording availability.                | Should-Have | If API fails, TranscriptionRequest status set to "failed". Recording remains playable and downloadable. User can retry.             |

### 6.6 File Upload Validation

| Req ID      | Description                                                                                                          | Priority    | Acceptance Criteria                                                                                                                 |
|-------------|----------------------------------------------------------------------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------|
| BRD-FR-036  | The system shall validate uploaded files by MIME type, accepting only audio/webm, audio/wav, and audio/ogg.           | Must-Have   | Uploads with disallowed MIME types are rejected with 415 Unsupported Media Type.                                                    |
| BRD-FR-037  | The system shall enforce a maximum file size limit on uploads.                                                        | Must-Have   | Uploads exceeding the size limit are rejected with 413 Payload Too Large and a descriptive error message.                           |
| BRD-FR-038  | The system shall sanitize all user-supplied text inputs (titles, tags) to prevent XSS.                                | Must-Have   | HTML/script tags in title or tag inputs are escaped or stripped. Stored values contain no executable markup.                         |

---

## 7. Non-Functional Requirements

| Req ID       | Category       | Description                                                                                      | Target                                                       |
|--------------|----------------|--------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| BRD-NFR-001  | Usability      | User shall start recording within 1 click from the dashboard.                                    | 1-click access to recording state                            |
| BRD-NFR-002  | Usability      | The UI shall be responsive for desktop (≥ 1024 px) and tablet (≥ 768 px) viewports.              | All core features usable on screens ≥ 768 px                 |
| BRD-NFR-003  | Usability      | Real-time waveform feedback during recording with no perceptible lag.                             | Waveform updates at ≥ 30 fps                                 |
| BRD-NFR-004  | Performance    | Non-AI API endpoints shall respond in under 2 seconds at p95.                                    | p95 latency < 2 s                                            |
| BRD-NFR-005  | Performance    | Audio upload and download shall use streaming to handle large files efficiently.                  | Files up to 500 MB transferred without timeout or OOM        |
| BRD-NFR-006  | Performance    | AI transcription requests shall be processed asynchronously without blocking UI or other API calls.| User can navigate away during transcription; status polled   |
| BRD-NFR-007  | Security       | RBAC shall be enforced on all API endpoints.                                                     | 100 % of endpoints have RBAC middleware                      |
| BRD-NFR-008  | Security       | All secrets (API keys, database credentials) shall be loaded from environment variables only.     | Zero hardcoded secrets in source code                        |
| BRD-NFR-009  | Security       | File uploads shall be validated for MIME type and size before storage.                            | All uploads pass validation before writing to disk           |
| BRD-NFR-010  | Security       | User-supplied text shall be sanitized to prevent XSS attacks.                                    | No executable markup in stored or rendered user input         |
| BRD-NFR-011  | Security       | CORS shall be scoped appropriately to restrict cross-origin access.                              | CORS whitelist includes only authorized origins               |
| BRD-NFR-012  | Reliability    | Recordings shall never be lost on page refresh — auto-save on stop is mandatory.                 | 100 % of stopped recordings persisted                        |
| BRD-NFR-013  | Reliability    | AI service failures shall not affect core recording functionality.                               | Record, play, manage, download all work when API is down     |
| BRD-NFR-014  | Reliability    | AI transcription failures shall produce retryable error states.                                  | Failed transcriptions have "failed" status and can be retried|
| BRD-NFR-015  | Observability  | The system shall log authentication events (sign-in success/failure).                            | Auth events logged with timestamp, user ID, outcome          |
| BRD-NFR-016  | Observability  | The system shall log recording uploads and deletions.                                            | Upload/delete events logged with recording ID and user ID    |
| BRD-NFR-017  | Observability  | The system shall log AI transcription API calls, errors, and latency.                            | Transcription calls logged with request ID, latency, status  |
| BRD-NFR-018  | Scalability    | The system shall support at least 50 concurrent users for MVP single-instance deployment.        | 50 concurrent users without degradation below SLA            |

---

## 8. GitHub Models Integration Requirements

This section captures requirements specific to the platform's use of the GitHub Models API for AI-driven audio transcription and summarization.

| Req ID       | Description                                                                                     | Priority    | Notes                                                                |
|--------------|-------------------------------------------------------------------------------------------------|-------------|----------------------------------------------------------------------|
| BRD-INT-001  | Authenticate with GitHub Models API using `GITHUB_MODELS_API_KEY` environment variable.          | Must-Have   | Key must never be logged or exposed in error messages.               |
| BRD-INT-002  | Connect to GitHub Models API endpoint via `GITHUB_MODELS_ENDPOINT` environment variable.         | Must-Have   | Endpoint URL is environment-configurable, not hardcoded.             |
| BRD-INT-003  | Send audio files to the Whisper model for speech-to-text transcription.                          | Should-Have | Input: audio file (WebM/WAV). Output: text, language, confidence.    |
| BRD-INT-004  | Send transcription text to GPT-4o model for summarization.                                       | Could-Have  | Input: transcription text. Output: summary text.                     |
| BRD-INT-005  | Handle GitHub Models API rate limits with exponential backoff retry (max 3 retries: 1 s, 2 s, 4 s).| Must-Have | After max retries, mark request as failed.                           |
| BRD-INT-006  | Wrap all API calls in try/except and return structured error responses.                          | Must-Have   | Errors logged (without API key), returned with status + message + retryable flag.|
| BRD-INT-007  | Store transcription results with request metadata (model, requester, timestamps) for audit.      | Should-Have | TranscriptionRequest and Transcription entities capture full audit trail.|
| BRD-INT-008  | Design the transcription service as a pluggable module for future MCP integration.               | Should-Have | Defined interface/protocol allows swap without changing callers.      |

### Integration Considerations

- **Model Selection**: Whisper is used for speech-to-text transcription. GPT-4o is used for text summarization. Model identifiers are configured via environment variables to allow future upgrades without code changes.
- **Rate Limits & Quotas**: GitHub Models API has rate limits. The application implements exponential backoff (max 3 retries) and queues transcription requests to avoid burst usage. Requests are queued with "pending" status and processed when capacity is available.
- **Prompt Management**: Summarization prompts are stored as configurable templates in the application codebase. Prompts can be updated without redeployment via configuration.
- **Fallback Strategy**: When the API is unavailable, transcription requests are marked "failed" with a retryable flag. Core recording features remain fully functional. Users can retry manually. No fallback to alternative AI providers in MVP.

---

## 9. Data Entities

### 9.1 User

| Attribute  | Type     | Constraints                         | Description                    |
|------------|----------|-------------------------------------|--------------------------------|
| id         | UUID     | Primary Key, auto-generated         | Unique user identifier         |
| name       | String   | Required, max 255 chars             | Display name                   |
| email      | String   | Required, unique, valid email       | Login identifier               |
| role       | Enum     | Required; values: "user", "admin"   | Access control role            |
| createdAt  | DateTime | Auto-set on creation                | Account creation timestamp     |

### 9.2 Recording

| Attribute  | Type     | Constraints                                              | Description                        |
|------------|----------|----------------------------------------------------------|------------------------------------|
| id         | UUID     | Primary Key, auto-generated                              | Unique recording identifier        |
| userId     | UUID     | Foreign Key → User.id, required                          | Owner of the recording             |
| title      | String   | Required, max 255 chars, sanitized                       | User-provided title                |
| filename   | String   | Required, unique, system-generated                       | Stored filename on filesystem      |
| fileSize   | Integer  | Required, bytes                                          | Size of the audio file             |
| duration   | Float    | Required, seconds                                        | Duration of the recording          |
| mimeType   | String   | Required; allowed: audio/webm, audio/wav, audio/ogg      | MIME type of the audio file        |
| status     | Enum     | Required; values: "recording", "saved", "deleted"         | Current lifecycle state            |
| createdAt  | DateTime | Auto-set on creation                                     | Recording creation timestamp       |
| updatedAt  | DateTime | Auto-set, updated on modification                        | Last modification timestamp        |

### 9.3 RecordingTag

| Attribute   | Type   | Constraints                          | Description               |
|-------------|--------|--------------------------------------|---------------------------|
| id          | UUID   | Primary Key, auto-generated          | Unique tag identifier     |
| recordingId | UUID   | Foreign Key → Recording.id, required | Associated recording      |
| tag         | String | Required, max 100 chars, sanitized   | Tag text value            |

### 9.4 TranscriptionRequest

| Attribute   | Type     | Constraints                                                       | Description                      |
|-------------|----------|-------------------------------------------------------------------|----------------------------------|
| id          | UUID     | Primary Key, auto-generated                                       | Unique request identifier        |
| recordingId | UUID     | Foreign Key → Recording.id, required                              | Recording to transcribe          |
| requesterId | UUID     | Foreign Key → User.id, required                                   | User who requested transcription |
| model       | String   | Required; e.g. "whisper"                                          | AI model used                    |
| status      | Enum     | Required; values: "pending", "processing", "completed", "failed"  | Processing lifecycle state       |
| createdAt   | DateTime | Auto-set on creation                                              | Request creation timestamp       |

### 9.5 Transcription

| Attribute              | Type     | Constraints                                    | Description                        |
|------------------------|----------|------------------------------------------------|------------------------------------|
| id                     | UUID     | Primary Key, auto-generated                    | Unique transcription identifier    |
| recordingId            | UUID     | Foreign Key → Recording.id, required           | Source recording                   |
| transcriptionRequestId | UUID     | Foreign Key → TranscriptionRequest.id, required| Originating request                |
| text                   | Text     | Required                                       | Full transcription text            |
| language               | String   | Optional; ISO 639-1 code                       | Detected language                  |
| confidence             | Float    | Optional; 0.0 – 1.0                            | Model confidence score             |
| createdAt              | DateTime | Auto-set on creation                           | Transcription creation timestamp   |

### 9.6 Entity Relationships

- **User** 1 → N **Recording** (a user owns many recordings)
- **Recording** 1 → N **RecordingTag** (a recording has many tags)
- **Recording** 1 → N **TranscriptionRequest** (a recording can have multiple transcription attempts)
- **TranscriptionRequest** 1 → 0..1 **Transcription** (a successful request produces one transcription)
- **User** 1 → N **TranscriptionRequest** (a user initiates many transcription requests)

---

## 10. API Endpoints Summary

All REST endpoints are under the `/api/v1/` prefix. Frontend page routes serve Jinja2 templates at root paths.

### 10.1 Authentication

| Method | Endpoint              | Description                 | Auth Required |
|--------|-----------------------|-----------------------------|---------------|
| POST   | `/api/v1/auth/login`  | Sign in with credentials    | No            |
| POST   | `/api/v1/auth/logout` | Sign out / invalidate token | Yes           |

### 10.2 Recordings

| Method | Endpoint                           | Description                         | Auth Required | Roles       |
|--------|------------------------------------|-------------------------------------|---------------|-------------|
| POST   | `/api/v1/recordings`               | Upload and save a new recording     | Yes           | User, Admin |
| GET    | `/api/v1/recordings`               | List current user's recordings      | Yes           | User, Admin |
| GET    | `/api/v1/recordings/{id}`          | Get recording metadata              | Yes           | User, Admin |
| PUT    | `/api/v1/recordings/{id}`          | Update recording title              | Yes           | User, Admin |
| DELETE | `/api/v1/recordings/{id}`          | Delete recording (file + metadata)  | Yes           | User, Admin |
| GET    | `/api/v1/recordings/{id}/download` | Download recording file             | Yes           | User, Admin |
| GET    | `/api/v1/recordings/{id}/stream`   | Stream recording with range support | Yes           | User, Admin |

### 10.3 Recording Tags

| Method | Endpoint                                | Description              | Auth Required | Roles       |
|--------|-----------------------------------------|--------------------------|---------------|-------------|
| POST   | `/api/v1/recordings/{id}/tags`          | Add a tag to a recording | Yes           | User, Admin |
| DELETE | `/api/v1/recordings/{id}/tags/{tagId}`  | Remove a tag             | Yes           | User, Admin |

### 10.4 Transcription

| Method | Endpoint                                          | Description                            | Auth Required | Roles       |
|--------|---------------------------------------------------|----------------------------------------|---------------|-------------|
| POST   | `/api/v1/recordings/{id}/transcribe`              | Request transcription                  | Yes           | User, Admin |
| GET    | `/api/v1/recordings/{id}/transcription`           | Get transcription result               | Yes           | User, Admin |
| GET    | `/api/v1/recordings/{id}/transcription/status`    | Get transcription request status       | Yes           | User, Admin |
| POST   | `/api/v1/recordings/{id}/transcription/summarize` | Request summarization of transcription | Yes           | User, Admin |

### 10.5 User Management (Admin)

| Method | Endpoint                    | Description                      | Auth Required | Roles |
|--------|-----------------------------|----------------------------------|---------------|-------|
| GET    | `/api/v1/admin/users`       | List all users                   | Yes           | Admin |
| POST   | `/api/v1/admin/users`       | Create a new user                | Yes           | Admin |
| GET    | `/api/v1/admin/users/{id}`  | Get user details                 | Yes           | Admin |
| PUT    | `/api/v1/admin/users/{id}`  | Update user details              | Yes           | Admin |
| DELETE | `/api/v1/admin/users/{id}`  | Delete / deactivate user         | Yes           | Admin |
| GET    | `/api/v1/admin/recordings`  | List all recordings (all users)  | Yes           | Admin |
| GET    | `/api/v1/admin/storage`     | Get storage usage statistics     | Yes           | Admin |

### 10.6 System Settings (Admin)

| Method | Endpoint                    | Description                  | Auth Required | Roles |
|--------|-----------------------------|------------------------------|---------------|-------|
| GET    | `/api/v1/admin/settings`    | Get current system settings  | Yes           | Admin |
| PUT    | `/api/v1/admin/settings`    | Update system settings       | Yes           | Admin |

---

## 11. Risks & Mitigations

| Risk ID | Description                                                                      | Likelihood | Impact | Mitigation Strategy                                                                                       |
|---------|----------------------------------------------------------------------------------|------------|--------|-----------------------------------------------------------------------------------------------------------|
| R-001   | GitHub Models API unavailability or rate limiting disrupts transcription          | Medium     | High   | Exponential backoff retry. Decouple transcription from core features. Mark failed requests as retryable.  |
| R-002   | Large audio files cause upload timeouts or server memory issues                  | Medium     | High   | Streaming upload/download. File size limits. Stream to disk without full memory buffering.                 |
| R-003   | Browser compatibility issues with Web Audio API or MediaRecorder API             | Low        | High   | Target modern browsers (Chrome, Firefox, Edge). Detect API support on load; show clear error messages.    |
| R-004   | Local filesystem storage reaches capacity                                        | Medium     | High   | Per-user storage quotas. Admin monitoring dashboard. Log warnings at thresholds.                          |
| R-005   | SQLite concurrency limitations under moderate load                               | Low        | Medium | Use WAL mode. Document scaling path to PostgreSQL.                                                        |
| R-006   | Malicious file uploads (non-audio files with spoofed MIME types)                 | Medium     | High   | Server-side MIME validation. Check file magic bytes. Size limits. Isolated storage directory.              |
| R-007   | XSS attacks via user-supplied recording titles or tags                           | Medium     | High   | Sanitize on write. Escape on render. Use Jinja2 auto-escaping.                                           |
| R-008   | API key exposure in logs or error messages                                       | Low        | High   | Never log API keys. Redact sensitive values. Environment variables only.                                  |
| R-009   | Data loss from accidental recording deletion                                     | Low        | Medium | Deletion confirmation. Consider soft-delete with "deleted" status before permanent removal.               |
| R-010   | Transcription accuracy issues with Whisper on poor-quality audio                 | Medium     | Low    | Display confidence scores. Document limitations. Allow re-transcription.                                  |

---

## 12. Appendix

### 12.1 Glossary

| Term                    | Definition                                                                                        |
|-------------------------|---------------------------------------------------------------------------------------------------|
| GitHub Models API       | GitHub's hosted AI model inference service providing access to models like Whisper and GPT-4o.    |
| Whisper                 | OpenAI's speech-to-text model, accessed via GitHub Models API for audio transcription.            |
| GPT-4o                  | OpenAI's large language model, accessed via GitHub Models API for text summarization.             |
| FastAPI                 | A modern, high-performance Python web framework for building APIs with automatic OpenAPI docs.    |
| Pydantic v2             | Python data validation library for request/response schemas and settings management.              |
| SQLite                  | A lightweight, file-based relational database engine in the Python standard library.              |
| Web Audio API           | Browser API for processing and analyzing audio content, used for waveform visualization.          |
| MediaRecorder API       | Browser API for recording media streams, producing WebM/WAV blobs.                                |
| Jinja2                  | Python templating engine for server-side HTML rendering.                                          |
| RBAC                    | Role-Based Access Control — authorization model based on user roles (User, Admin).                |
| WebM                    | An open media container format, commonly used with the Opus audio codec.                          |
| WAV                     | Waveform Audio File Format — an uncompressed audio format.                                        |
| XSS                     | Cross-Site Scripting — a vulnerability where malicious scripts are injected into web pages.       |
| CORS                    | Cross-Origin Resource Sharing — HTTP header mechanism for cross-origin request control.           |
| MCP                     | Model Context Protocol — a future integration pattern for pluggable AI model services.            |
| MVP                     | Minimum Viable Product — the initial release with core features for early adoption.               |

### 12.2 References

- GitHub Models API Documentation: https://docs.github.com/en/github-models
- FastAPI Documentation: https://fastapi.tiangolo.com/
- MDN Web Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- MDN MediaRecorder API: https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder
- Pydantic v2 Documentation: https://docs.pydantic.dev/latest/
- SQLite Documentation: https://www.sqlite.org/docs.html
- OWASP XSS Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Scripting_Prevention_Cheat_Sheet.html

### 12.3 Requirement ID Cross-Reference

| ID Range                    | Category                    | Count |
|-----------------------------|-----------------------------|-------|
| BRD-FR-001 to BRD-FR-038   | Functional Requirements     | 38    |
| BRD-NFR-001 to BRD-NFR-018 | Non-Functional Requirements | 18    |
| BRD-INT-001 to BRD-INT-008 | Integration Requirements    | 8     |
| **Total**                   |                             | **64**|
