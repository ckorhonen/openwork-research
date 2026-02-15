# Structured Task Management System for OpenClaw: Design Document

## System Overview

OpenClaw agents currently suffer from fragmentation across the filesystem—protocols live in one directory, active tasks in another, completed work scattered across dated folders, and agent state locked in ephemeral configurations. This architecture creates operational blind spots where humans cannot easily audit agent activity, agents duplicate effort because they cannot discover existing tasks, and critical context is lost when sessions terminate.

The proposed Task Management System (TMS) introduces a unified, persistent layer that serves as the single source of truth for all agent-directed work. Rather than treating task tracking as an afterthought bolted onto file operations, this design positions the TMS as a first-class component with dedicated APIs, human-readable persistence, and explicit integration points with OpenClaw's existing workflow primitives. The system prioritizes three core principles: transparency (humans can always understand what agents are doing and why), recoverability (state persists across restarts without complex serialization), and composability (tasks can reference other tasks, protocols, and external resources as first-class entities).

## Architecture

The recommended architecture employs a hybrid approach combining Git-backed version control for human accessibility with a lightweight SQLite database for structured querying and agent operations. This dual-store pattern leverages the strengths of each technology: Git provides diffable history, branch-based experimentation, and natural integration with existing developer workflows, while SQLite offers ACID transactions, complex query capabilities, and millisecond-latency updates that agents require.

The system operates through three primary layers. The storage layer maintains a `/openclaw/tms/` directory containing a SQLite database (`tasks.db`), a `protocols/` directory with version-controlled task templates, and a `state/` directory for serialized agent checkpoints. The API layer exposes a Python module (`openclaw_tms`) providing `Task`, `Protocol`, and `AgentSession` classes with methods for CRUD operations, dependency management, and status transitions. The integration layer connects the TMS to OpenClaw's existing message bus and file operation hooks, enabling automatic task creation when agents encounter new objectives and status updates when protocols complete.

For external integrations, the architecture supports three sync adapters: a GitHub Issues bridge for teams already using issue trackers, a Todoist API connector for personal productivity workflows, and a minimal REST API for custom dashboard implementations. Each adapter operates bidirectionally with conflict resolution handled through a canonical ordering based on timestamps and explicit merge keys.

## Data Model

The core schema centers on four interconnected entities. The `tasks` table stores the fundamental work units with columns for `task_id` (UUID primary key), `title`, `description`, `status` (enum: pending, in_progress, blocked, completed, failed), `priority` (integer 1-5), `assignee_agent`, `parent_task_id` (for hierarchical decomposition), `created_at`, `updated_at`, `protocol_ref` (foreign key to templates), and `metadata` (JSON for extensibility).

The `protocols` table defines reusable task templates with `protocol_id`, `name`, `version`, `parameter_schema` (JSON Schema for validation), `step_sequence` (ordered list of task templates), and `author`. Protocols enable agents to instantiate consistent task trees for common operations rather than constructing ad-hoc workflows each time.

The `agent_sessions` table tracks operational context with `session_id`, `agent_type`, `start_time`, `current_task_id`, and `heartbeat`. This table enables human operators to understand which agents are active, what they're working on, and whether sessions have become stale.

The `file_annotations` table bridges the TMS to the filesystem, recording `path`, `associated_task_id`, `annotation_type` (generated, consumed, modified), and `timestamp`. This table answers questions like "which task created this file?" and "what tasks depend on this output?"

## Human Interface

Humans interact with the TMS through three complementary interfaces. The command-line interface (`claw-tms`) provides quick operations: `claw-tms status` shows active tasks across all agents, `claw-tms tree <task_id>` displays hierarchical decomposition, `claw-tms log --agent=<name>` filters history by agent, and `claw-tms edit <task_id>` opens the task definition in EDITOR for natural editing. The CLI prioritizes parseable output formats (JSON, tabular) suitable for scripting and monitoring pipelines.

The monitoring dashboard, accessible via a local web server, presents several views. The operational view shows real-time agent activity with task progress bars, blocking relationships, and recent state transitions. The analytical view provides velocity metrics, bottleneck identification (tasks blocked for excessive durations), and trend analysis over configurable time windows. The audit view presents searchable, filterable history with diff capability for comparing task specifications across revisions.

For direct database access during debugging or migration, the SQLite file at `/openclaw/tms/tasks.db` can be inspected with any standard tool. This design choice ensures vendor independence and eliminates proprietary formats from critical infrastructure.

## Agent Interface

Agents interact with the TMS through a well-defined Python API. The primary interaction pattern follows this sequence:

```python
from openclaw_tms import TaskContext, TaskStatus

# Acquire context for current session
ctx = TaskContext(agent_id="research-agent-03")

# Claim and begin work on a task
task = ctx.claim_next_task(criteria={"status": "pending", "priority__gte": 3})
task.update_status(TaskStatus.IN_PROGRESS)

# Report progress and outputs
task.add_output("/openclaw/data/analysis/results.csv")
task.add_artifact("key_findings", {"metric": 0.87, "significance": "p<0.01"})
task.checkpoint()  # Persist state to recovery point

# Mark completion with summary
task.complete(summary="Generated comprehensive analysis with 94% confidence")
```

The API provides atomic operations for concurrent safety, automatic retry logic for transient failures, and exponential backoff for integration adapters. Agents receive structured notifications through callback registrations, enabling reactive workflows where one task's completion triggers another's initialization.

## Implementation Plan

Phase one establishes the foundation: creating the directory structure, initializing the SQLite schema, and implementing basic CRUD operations. This phase requires two weeks and produces a functional storage layer without integration hooks.

Phase two builds agent integration over three weeks, implementing the Python API, adding message bus subscriptions for automatic task creation, and developing the recovery checkpoint mechanism. During this phase, existing OpenClaw workflows continue unchanged while new code adopts the TMS pattern.

Phase three delivers human interfaces across four weeks: the CLI tool, the monitoring dashboard, and documentation. This phase includes comprehensive onboarding materials explaining migration procedures for teams with established folder structures.

Phase four conducts migration and stabilization over two weeks, transitioning production workloads, validating performance under load, and addressing issues identified during shadow operation. The migration path explicitly supports incremental adoption—teams can begin using the TMS for new tasks while gradually migrating historical context.

## Pros/Cons Analysis

The Git-backed hybrid architecture offers significant advantages: human-readable history enables audit and compliance requirements without exporting to separate systems, branch-based experimentation allows testing task template variations without affecting production, and the universal availability of Git tools eliminates training overhead. The primary drawback involves potential write contention during high-frequency agent operations, mitigated through SQLite's transaction isolation and careful retry configuration.

A pure Todoist API solution would provide excellent mobile accessibility and familiar interfaces, but introduces external dependencies, rate limits unsuitable for high-frequency updates, and data residency concerns for sensitive workflows. A GitHub Issues approach offers powerful label and project management features but struggles with hierarchical task decomposition and imposes GitHub-specific authentication requirements that limit deployment flexibility.

The chosen architecture sacrifices some simplicity for comprehensive capability, accepting that the implementation requires more upfront investment than lightweight alternatives. This tradeoff aligns with OpenClaw's positioning as a serious tool for production workloads where reliability and auditability justify architectural investment.

## Migration Path

The transition from scattered file-based organization proceeds through several stages. First, an inventory script scans existing directories for task markers (dated folders, TODO files, protocol indicators) and generates a mapping file in the migration format. Second, operators review the mapping, consolidating duplicates and resolving ambiguities through the CLI. Third, the migration script imports validated entries into
