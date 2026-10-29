# DroidForge — Product Specification

**Version:** 0.3  
**Status:** Draft — architecture selected  
**Project type:** Local-first, personality-driven AI companion and technician droid

## 1. Vision

DroidForge is a personal droid designed to act as both a companion and a technician. It combines a persistent identity, conversational intelligence, memory, and carefully controlled tools to help its user troubleshoot systems, manage information, and identify useful opportunities.

The initial deployment targets a Linux desktop running Bazzite. The droid will use a local language model through LM Studio, with a React-based interface and a Go backend.

Future versions may integrate voice, a Raspberry Pi, GPS, and physical peripherals.

## 2. Core Principles

1. **Character matters:** The droid should have a recognizable identity, personality, and mannerisms.
2. **Local-first:** Core functionality should not require cloud-hosted AI services.
3. **Human authority:** The droid suggests actions and asks before taking consequential actions.
4. **Explicit permissions:** The model cannot bypass the backend's security rules.
5. **Persistent identity:** Configuration and approved memories survive restarts.
6. **Modularity:** Model providers, tools, interfaces, and hardware integrations should be replaceable.
7. **Incremental development:** Features are implemented and tested against explicit acceptance criteria.
8. **Recoverability:** Installation, configuration changes, and updates should be reversible.

## 3. Technology Stack

### 3.1 Backend

- Language: Go.
- HTTP server: Go standard library initially.
- API: REST endpoints using JSON.
- Persistence: SQLite.
- Model integration: LM Studio's OpenAI-compatible API.
- Testing: Go's standard testing framework.
- Configuration: Local configuration files with documented defaults.

The backend owns conversation orchestration, personality configuration, memory, tool registration, permission enforcement, diagnostics, and audit logging.

A framework may be introduced later if concrete requirements justify it.

### 3.2 Frontend

- React.
- TypeScript.
- A modern JavaScript package manager and build tool.
- HTTP/JSON communication with the Go backend.

The frontend provides conversation, personality and memory controls, diagnostic results, and action approval workflows.

The initial interface runs locally in a browser. Desktop packaging may be evaluated after the core application is functional.

### 3.3 AI Provider

LM Studio is the initial model provider.

The backend must communicate with the model through a dedicated provider interface so that model-provider-specific details do not spread throughout the application.

Model output is untrusted input. The backend validates proposed tool calls and enforces permissions independently of model instructions.

### 3.4 Development Environment

- Primary OS: Bazzite Linux.
- Repository: GitHub.
- Development approach: Specification-driven development.
- Installation: User-level where practical, without requiring modifications to the immutable operating system image.

### 3.5 Future Hardware

A Raspberry Pi may eventually run a separate Go service for voice, sensors, GPS, or other peripherals.

Hardware integrations are outside the initial release. Communication between devices must use an explicitly defined, authenticated interface.

## 4. Architecture

The initial application is a modular monolith: one Go backend with internal package boundaries, accompanied by a separate React frontend.

Conceptual data flow:

1. The user interacts with the React interface.
2. The frontend sends a request to the Go backend.
3. The backend manages conversation state and contacts the configured language model.
4. If the model proposes a tool call, the backend validates it against registered tools and permission rules.
5. If approval is required, the backend presents a proposal to the user and waits.
6. The backend executes only an authorized action, records the result, and returns the response to the frontend.

The model must never have unrestricted access to the host shell, filesystem, or arbitrary executable commands.

The frontend is not a security boundary. All authorization checks must occur in the backend.

## 5. Functional Requirements

### FR-001: Conversational Interface

The user can exchange text messages with the droid through a local web interface.

**Acceptance criteria:**
- Messages can be submitted and displayed.
- Model responses are displayed with useful error handling.
- The interface distinguishes pending, completed, and failed requests.
- Model unavailability does not crash the application.

### FR-002: Model Provider

The backend connects to LM Studio through its compatible API.

**Acceptance criteria:**
- The model endpoint and model configuration can be changed without rewriting conversation logic.
- Connection failures produce understandable errors.
- Provider-specific logic is isolated behind an interface.

### FR-003: Droid Identity

The droid has a configurable name, personality, communication style, and mannerisms.

**Acceptance criteria:**
- Identity settings persist across restarts.
- The droid maintains a recognizable voice across ordinary interactions.
- Personality instructions cannot override backend security policies.

### FR-004: Companion Behavior

The droid can converse naturally, answer questions, and assist with general tasks within its available capabilities.

**Acceptance criteria:**
- Conversation context is maintained within the configured limits.
- Limitations and uncertainty are communicated honestly.
- The droid does not fabricate memories, tool results, or completed actions.

### FR-005: Technician Capabilities

The droid can assist with troubleshooting the user's Linux desktop.

**Acceptance criteria:**
- The initial release provides at least three read-only diagnostic tools.
- Initial candidates include CPU and memory usage, storage status, GPU information, and network status.
- Results are returned in a structured form and explained conversationally.
- Diagnostic failures are reported without crashing the backend.

### FR-006: Proactive Suggestions

The droid may suggest useful actions or point out opportunities but must ask before acting.

**Acceptance criteria:**
- A suggestion explains why it is relevant.
- The proposed action and expected effect are clear.
- The user can accept, decline, or defer the suggestion.
- Declined suggestions are not repeatedly presented unless circumstances materially change.
- Presenting a suggestion never executes the proposed action.

### FR-007: Tool Permissions and Approval

The backend maintains an explicit registry of available tools and their permission requirements.

**Acceptance criteria:**
- Unregistered tools and invalid arguments are rejected.
- Read-only tools can run within their documented permissions.
- Consequential, destructive, or state-changing actions require explicit approval.
- Approval is specific to the proposed action and its arguments.
- The backend revalidates authorization when executing an approved action.
- The model cannot grant itself permissions or bypass approval checks.
- Tool execution is logged.

### FR-008: Persistent Memory

The droid stores selected facts and preferences in SQLite.

**Acceptance criteria:**
- Memory persists across restarts.
- The user can inspect, correct, and delete stored memories.
- Memory is not silently inferred to be certain when it is ambiguous.
- Sensitive information is not retained by default.
- Conversation history and long-term memory are treated as distinct concepts.

### FR-009: Audit Logging

The backend records relevant tool requests, approval decisions, execution outcomes, and errors.

**Acceptance criteria:**
- Logs distinguish proposed, approved, rejected, and executed actions.
- Logs do not unnecessarily expose secrets or sensitive content.
- The user can diagnose failures using local logs.

### FR-010: Modular Design

The codebase separates the API, conversation logic, model provider, personality, memory, permissions, and tools.

**Acceptance criteria:**
- Each major subsystem has a defined responsibility.
- Interfaces allow components to be replaced without widespread changes.
- Tests can exercise important logic without requiring a running language model.

### FR-011: Local-First Operation

Core functionality runs locally and does not require a cloud AI service.

**Acceptance criteria:**
- The application uses the configured local model endpoint.
- The application identifies unavailable dependencies clearly.
- No external AI provider is required for ordinary operation.
- Network access outside configured local services is documented and minimized.

### FR-012: Frontend/Backend Separation

The React frontend communicates with the Go backend through a documented API.

**Acceptance criteria:**
- API request and response formats are documented.
- Backend authorization does not depend on frontend behavior.
- API errors have a consistent structure.
- The frontend cannot execute operating-system commands directly.

### FR-013: Installation and Recovery

The application can be installed and removed without making unnecessary system-wide changes.

**Acceptance criteria:**
- Development and runtime instructions are documented for Bazzite.
- User-level configuration and application data have documented locations.
- The user can back up important configuration and memory data.
- Removal instructions explain which data is retained and how to remove it.
- The application does not require modifying the Bazzite system image for ordinary use.

## 6. Future Requirements

The following are explicitly outside the first release:

- Voice input and speech output.
- Raspberry Pi integration.
- GPS and location-aware behavior.
- Physical sensors and peripherals.
- Event-driven suggestions based on monitored system state.
- Desktop packaging.
- Advanced automation and additional state-changing tools.

These features require their own specifications and acceptance criteria before implementation.

## 7. Non-Functional Requirements

### Security
- Apply least-privilege principles.
- Keep secrets out of source control and logs.
- Validate untrusted model output and API input.
- Restrict the backend API to local access by default.
- Require authentication and authorization before exposing control interfaces to other devices.

### Reliability
- Handle model, database, and tool failures gracefully.
- Avoid losing persistent memory during ordinary shutdowns.
- Provide clear startup and configuration errors.

### Maintainability
- Prefer readable, idiomatic Go and TypeScript.
- Document architectural decisions.
- Keep dependencies limited until a concrete need exists.

### Testability
- Automate tests for permission checks, approval workflows, API behavior, and memory operations.
- Define acceptance tests for each milestone.
- Keep critical security logic testable without model-generated output.

### Resource Awareness
- Avoid unnecessary background work.
- Account for the fact that local model inference may consume substantial GPU and system resources.
- Do not assume the backend itself needs GPU access.

## 8. Initial Release Scope

The first usable release must provide:

1. A Go backend.
2. A React + TypeScript web interface.
3. Local communication with LM Studio.
4. A configurable droid identity.
5. Persistent SQLite memory with user controls.
6. Read-only system diagnostic tools.
7. A suggestion and approval workflow.
8. Explicit backend permission checks and audit logging.
9. Automated tests for critical behavior.
10. Documented, reversible installation on Bazzite.

Voice, GPS, Raspberry Pi integration, desktop packaging, cloud AI dependencies, autonomous state-changing actions, and unrestricted shell access are excluded.

## 9. Development Milestones

**Milestone 1 — Droid Shell**
- Initialize the repository.
- Establish Go and React development workflows.
- Implement the backend API and frontend connection.
- Connect to LM Studio.
- Display basic conversations.

**Milestone 2 — Identity and Memory**
- Implement configurable personality.
- Add SQLite persistence.
- Add memory inspection and deletion.

**Milestone 3 — Technician**
- Implement read-only diagnostic tools.
- Normalize tool outputs.
- Add error handling and tests.

**Milestone 4 — Suggestions and Permissions**
- Add explicit tool registration.
- Implement approval workflows.
- Add audit logging and security tests.

**Milestone 5 — Deployment and Evaluation**
- Document Bazzite setup and recovery.
- Test startup, shutdown, and model-unavailable behavior.
- Review resource usage and security boundaries.

Future milestones may introduce desktop packaging, voice, and Raspberry Pi integrations.

## 10. Development Rule

Before implementing a feature:

1. Identify the relevant requirements.
2. Define acceptance criteria and failure cases.
3. Update architecture or API documentation if needed.
4. Implement the smallest coherent change.
5. Add automated tests.
6. Verify the acceptance criteria.
7. Update the documentation.

Architectural changes should be recorded as short decision documents explaining the context, decision, alternatives, and consequences.

**The specification describes intended behavior; the implementation and tests must demonstrate that behavior.**