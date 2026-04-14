# 078 Triggering Runs via API in Airflow

Canonical documentation for 078 Triggering Runs via API in Airflow. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of triggering runs via API is to enable event-driven orchestration and external control over workflow execution. While Airflow is primarily known for time-based scheduling, the API interface allows the platform to function as a reactive component within a larger distributed system. This mechanism addresses the need for integrating Airflow with external CI/CD pipelines, webhooks, upstream data signals, and manual administrative overrides from third-party interfaces.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural behavior of the Airflow REST API rather than specific client-side library implementations.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The mechanism of the POST request to the DAG Runs endpoint.
* Data structures for triggering runs (payloads and parameters).
* State transitions initiated via API calls.
* Authentication and authorization requirements for API access.
* Idempotency and run identification.

**Out of scope:**
* Specific cloud provider wrappers (e.g., MWAA or Cloud Composer specific CLI tools).
* Legacy Experimental API (deprecated).
* Internal TriggerDagRunOperator (which operates within the scheduler context).

## Definitions
| Term | Definition |
|------|------------|
| **DAG Run** | A specific instance of a Directed Acyclic Graph (DAG) in time, representing a single execution of the workflow. |
| **Logical Date** | (Formerly Execution Date) The timestamp assigned to a run, used for idempotency and data interval referencing. |
| **Run ID** | A unique identifier for a specific DAG run. If not provided via API, the system generates one (usually prefixed with `manual__`). |
| **Configuration (conf)** | A JSON-formatted payload passed during the API call that provides runtime parameters to the DAG tasks. |
| **Endpoint** | The specific URL path (typically `/dags/{dag_id}/dagRuns`) used to interface with the Airflow web server. |
| **State** | The status of the triggered run (e.g., `queued`, `running`, `success`, `failed`). |

## Core Concepts

### The RESTful Interface
Triggering a run is handled via a standard RESTful POST request. The API acts as a gateway to the Airflow Metadata Database. When a request is validated, a new entry is created in the `dag_run` table with a state of `queued`.

### Asynchronicity
API triggers are inherently asynchronous. A successful API response (HTTP 200 or 201) indicates that the run has been successfully *accepted and queued* by the scheduler. It does not indicate that the workflow has completed or even started execution.

### Payload Structure
The API accepts a JSON body containing:
1.  **conf**: A dictionary of key-value pairs accessible within the DAG via Jinja templating or the task context.
2.  **dag_run_id**: An optional custom string to identify the run.
3.  **logical_date**: An optional ISO 8601 timestamp.
4.  **note**: An optional descriptive string for administrative tracking.

## Standard Model

The standard model for triggering a run follows a strict sequence:
1.  **Authentication**: The client provides credentials (Basic Auth, Kerberos, or JWT) to the API.
2.  **Validation**: The Airflow Web Server verifies that the `dag_id` exists and is active.
3.  **Persistence**: A new record is inserted into the metadata database.
4.  **Scheduling**: The Airflow Scheduler detects the `queued` run and assigns tasks to workers based on priority and slot availability.
5.  **Execution**: Tasks execute, utilizing the `conf` parameters provided in the initial API call.

### Idempotency
To ensure a run is not triggered multiple times by mistake (e.g., due to network retries), the API enforces uniqueness on the combination of `dag_id` and `logical_date` (or `dag_run_id`). Attempting to trigger a run with a duplicate ID will result in a 409 Conflict error.

## Common Patterns

### Event-Driven Webhooks
An external system (like a GitHub Action or a cloud storage event) sends a POST request to the Airflow API upon completion of an upstream event. This transforms Airflow from a "pull" (polling) system to a "push" (reactive) system.

### Parameterized Execution
Using the `conf` payload to pass dynamic values such as S3 bucket paths, database connection IDs, or specific user IDs. This allows a single DAG definition to handle multiple distinct execution contexts without code changes.

### Cross-Environment Triggering
A "Master" Airflow instance in one environment triggers a "Worker" DAG in a separate Airflow instance (e.g., across different VPCs or regions) via the API to maintain isolation.

## Anti-Patterns

*   **Polling for Status via API**: Repeatedly calling the GET DAG Run endpoint to check for completion. This creates unnecessary load on the web server. Use **Callbacks** (Slack, Email) or **External Sensors** instead.
*   **Hardcoding Logical Dates**: Providing the same `logical_date` in every API call, which causes subsequent triggers to fail due to uniqueness constraints.
*   **Bypassing the Scheduler**: Attempting to trigger runs by direct database manipulation instead of using the API. This leads to database corruption and bypasses validation logic.
*   **Large Payloads**: Passing massive amounts of data (e.g., a 10MB JSON) in the `conf` field. The metadata database is not designed for large blob storage; pass a reference (like a file path) instead.

## Edge Cases

*   **Triggering Paused DAGs**: By default, triggering a run on a paused DAG will create the run in a `queued` state, but it will not execute until the DAG is unpaused.
*   **Backfilling via API**: While possible to trigger historical dates via API, this does not behave exactly like the `airflow dags backfill` CLI command, as it does not respect certain dependency-ignore flags.
*   **Race Conditions**: If two API calls arrive simultaneously for the same `dag_id` without a specified `dag_run_id`, the system must generate unique IDs to prevent collisions.
*   **Max Active Runs**: If the DAG has reached its `max_active_runs` limit, an API-triggered run will remain in the `queued` state until a slot becomes available.

## Related Topics
*   **012 DAG Scheduling and Timetables**: Understanding how API triggers interact with existing schedules.
*   **045 Airflow Security Model**: Details on API authentication (OAuth2, Basic Auth).
*   **089 Jinja Templating**: How to access `{{ dag_run.conf }}` within tasks.
*   **102 Airflow CLI**: Alternative methods for triggering runs from the command line.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |