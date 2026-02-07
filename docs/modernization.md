# Dribdat v3 Roadmap

_Contents:_

- [Modernization Proposal](#modernization-propsal)
- [Sync Engine Redesign](#sync-engine-redesign)
- [Migration Strategy](#migration-strategy)

## Modernization Proposal

This document outlines a major rework of the Dribdat tech stack to modernize the application, improve developer and user experience, and ensure the codebase is optimized for agentic development.

Generated with Google Jules based on a complete analysis of the source code and documentation, as well as an interview with @loleg

## 1. Vision & Goals

- **Streamline Modern Hackathons:** Refocus on recommending bootstraps, collecting challenges/results, and supporting "vibe coding."
- **Agent-Friendly Codebase:** Architecture designed to be easily understood and modified by AI coding assistants.
- **Lightweight & Sustainable:** Minimize resource footprint, maximize portability, and keep hosting costs low.
- **Data Portability:** Use open standards like Frictionless Data Packages for all migrations and exports.

## 2. Modernized Tech Stack

### Backend: FastAPI

- **Language:** Python 3.12+
- **Framework:** FastAPI
- **Why:** High performance, native support for asynchronous programming, and automatic OpenAPI generation.
- **Agent Advantage:** Strict typing with Pydantic makes the API self-documenting for AI agents.

### Data Layer: SQLModel

- **ORM:** SQLModel (SQLAlchemy + Pydantic)
- **Database:** PostgreSQL (Production), SQLite (Development/Edge)
- **Why:** Reduces boilerplate by using the same models for database schemas and API responses.
- **Agent Advantage:** Single source of truth for data structures prevents "hallucinations" about model fields.

### Frontend: Vue.js 3 (Integrated Backboard)

- **Framework:** Vue 3 + Vite
- **UI Component Library:** Tailwind CSS (replaces Bootstrap for better customizability and smaller bundles)
- **Why:** Real-time interactivity for "Dribs" (activity logs) and a modern presentation mode.
- **Integration:** Merge the existing "Backboard" frontend into the core project.

### Storage & Media

- **Service:** S3-compatible storage (via `aioboto3`)
- **Use Case:** Project images, logos, and presentation uploads.

---

## 3. Agentic Development Process

To make the codebase "conducive to the use of agentic development," we recommend the following process and structure:

### Modular "Domain" Directory Structure

Instead of a monolithic `models.py` or `views.py`, the app should be organized by domain:

```text
dribdat_vnext/
├── domains/
│   ├── event/        # Models, Services, API routes for Events
│   │   ├── models.py
│   │   ├── service.py
│   │   ├── router.py
│   │   └── AGENTS.md # Specific instructions for AI agents working here
│   ├── project/      # Project management & Syncing
│   ├── user/         # Auth & Profiles
│   └── activity/     # Dribs & Logs
├── core/             # Base configurations, DB setup, generic utils
└── main.py           # Application entry point
```

### Strict Coding Standards

1. **100% Type Coverage:** Every function must have type hints for arguments and return values.
2. **Pydantic for Data Validation:** All external data must be validated through Pydantic schemas.
3. **Explicit Side Effects:** Business logic should reside in "Service" layers, keeping Routers (API endpoints) thin.
4. **AGENTS.md Files:** Each module contains an `AGENTS.md` file explaining its purpose, key patterns, and "gotchas."

---

## 4. Full Set of Requirements for Dribdat vNext

### R1: Core Event Management

- Ability to create, manage, and archive hackathon events.
- Support for "Phases" (Draft, Active, Finished).
- Customizable Event CSS and branding.
- Countdown timers and real-time announcements.

### R2: Project & Team Formation

- 7-Stage Project Lifecycle (Challenge -> ... -> Result).
- "Join" and "Star" functionality for team formation.
- **Bootstrap Recommendation Engine:** Organizers can tag projects as "Starters" or "Bootstraps" to guide participants.
- Support for "Vibe Coding" through simple, low-friction project updates.

### R3: Intelligent Data Sync (The "Sync Engine")

- Asynchronous synchronization of project metadata from:
  - GitHub (Repositories & Gists)
  - GitLab
  - Codeberg
  - HuggingFace
  - Etherpad
- Support for custom "Data Package" descriptors for project metadata.

### R4: Real-time Activity Log ("Dribs")

- A global and project-specific "firehose" of activity.
- Support for WebSockets or Server-Sent Events (SSE) to provide instant updates.
- Commenting and upvoting ("Boosting") features.

### R5: Presentation Mode

- Integrated slide mode using Markdown (Marpit compatible).
- Auto-embedding of external presentation links (Google Slides, Speaker Deck).
- Integrated timer and navigation for project showcases.

### R6: Data Portability & Migration

- Support for **Frictionless Data Package** (CSV/JSON) for exporting all event data.
- Migration scripts to import data from legacy Dribdat (Flask) instances.
- "Simple Resume" support via JSON-LD in user profiles.

---

## 5. Migration Strategy

1. **Export:** Use the existing Dribdat API to export data into a standard Data Package format.
2. **Schema Mapping:** Map legacy SQLAlchemy models to new SQLModel classes.
3. **Import:** Create a CLI tool in vNext that reads the Data Package and populates the new PostgreSQL database.
4. **Proxy/Co-existence:** (Optional) Run the legacy app in read-only mode for historical events while using vNext for new events.

# Sync Engine Redesign

The Project Sync Engine is responsible for aggregating data from various external sources (GitHub, GitLab, etc.) into the Dribdat project dashboard.

## 1. Current Architecture

Currently, synchronization is synchronous and happens during web requests or via simple CLI commands. It uses multiple libraries (`requests`, `pyquery`, etc.) and lacks a standardized plugin system.

## 2. Proposed vNext Architecture

### Asynchronous & Task-Based

Syncing should be non-blocking and managed by a task queue:

- **FastAPI BackgroundTasks** for simple, immediate syncs.
- **Taskiq** or **Celery** for scheduled or high-volume syncs.

### Provider-Based Plugin System

Each external data source should be a "Provider" class implementing a standard interface:

```python
class BaseProvider(ABC):
    @abstractmethod
    async def fetch_metadata(self, url: str) -> ProjectMetadata:
        """Fetch basic info: name, summary, image, etc."""
        pass

    @abstractmethod
    async def fetch_content(self, url: str) -> str:
        """Fetch long-form content (README.md)."""
        pass

    @abstractmethod
    async def fetch_activity(self, url: str, since: datetime) -> List[Activity]:
        """Fetch recent commits/updates."""
        pass
```

### Supported Providers (Priority)

1. **GitHub Provider:** Uses GitHub API (with optional token support to avoid rate limiting).
2. **GitLab Provider:** Supports both gitlab.com and self-hosted instances.
3. **HuggingFace Provider:** Syncs model/dataset cards.
4. **Codeberg/Gitea Provider:** Generic Forgejo/Gitea support.
5. **Generic Web/OpenGraph Provider:** Fallback for any URL (extracting Title, Description, and OG images).

## 3. Data Flow

1. **Trigger:** A user clicks "Sync", or a scheduled job runs.
2. **Dispatch:** The Sync Engine identifies the provider based on the `autotext_url`.
3. **Fetch:** The provider fetches data asynchronously.
4. **Transform:** Raw data is transformed into Pydantic models (e.g., `SyncData`).
5. **Reconcile:**
    - If a field is empty in Dribdat but present in the remote source, it is filled.
    - If a field is marked as "Always Sync," it is overwritten.
    - Commits are added as `Activity` objects if they don't already exist.
6. **Notify:** The user is notified via WebSocket/SSE that the sync is complete.

## 4. Agentic Advantage

- **Separation of Concerns:** Agents can easily implement new providers by following the `BaseProvider` interface without touching the core engine logic.
- **Mocking:** The standard interface makes it trivial to write unit tests with mocked network responses.
- **Type Safety:** Using Pydantic for the `ProjectMetadata` ensures that the transformation logic is robust and easy for agents to validate.

# Migration Strategy

Moving from the legacy Flask stack to the modern FastAPI stack requires a robust data migration path. We utilize the **Frictionless Data Package** format as our intermediate "bridge."

## 1. Phase 1: Export from Legacy Dribdat

The legacy application already includes some support for data exports. We recommend enhancing the existing `dribdat/apipackage.py` to produce a full Data Package:

- **Resource: Users:** Export all active users (excluding passwords, which will need to be reset or migrated via a secure hash if compatible).
- **Resource: Events:** All hackathon event metadata.
- **Resource: Projects:** All project data including team memberships.
- **Resource: Activities:** The full "Dribs" history for each project.

## 2. Phase 2: Data Package Validation

Before importing into vNext, the exported package can be validated using the `frictionless` CLI:

```bash
frictionless validate datapackage.json
```

This ensures that the data conforms to the expected schema and that there are no broken relationships (e.g., a project pointing to a non-existent event).

## 3. Phase 3: Import into Dribdat vNext

The vNext application will include a migration CLI:

```bash
dribdat-cli migrate --source datapackage.json
```

### Key Import Steps

1. **Dependency Sorting:** Import Events first, then Categories, then Users, then Projects, then Activities.
2. **Password Handling:** Since FastAPI (via Passlib/Bcrypt) might use different salt/rounds than Flask-Bcrypt, we recommend:
    - Carrying over the hashes and attempting compatibility.
    - OR flagging users to "Reset Password on First Login."
3. **ID Mapping:** Maintain a mapping of old IDs to new IDs if the primary keys change during the import process.

## 4. Phase 4: S3 Media Migration

If S3 was used in the legacy app, the `image_url` fields in the database will likely remain valid if the same S3 bucket is used. If moving buckets:

1. Use a script to iterate through all `image_url` and `logo_url` fields.
2. Download from the old bucket and upload to the new bucket.
3. Update the database record in vNext.

## 5. Why this strategy?

- **Agent Friendly:** Migration logic is decoupled from both the old and new database schemas.
- **Standardized:** Uses the [Data Package](https://specs.frictionlessdata.io/data-package/) standard.
- **Resilient:** You can edit the intermediate JSON/CSV files manually if data cleanup is required during the transition.
