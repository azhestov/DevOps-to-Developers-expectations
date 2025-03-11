# DevOps-to-Developers-expectations
DevOps to Developers expectations to the service writed.

## Documentation

- A short description of the service (what it is, what it does, and why) at the beginning of `README.md`.
- Instructions on how to start, stop, safely restart, and restart dependencies in `README.md`.
  - Covers both direct dependencies (e.g., "restart me when the database restarts") and known reverse dependencies (e.g., "if restarted, also restart service X").
- List of supported and required environment variables in `README.md`.
- Description of service endpoints and metrics in `README.md`.
- Repository link in `README.md`.
- Link to documentation and API documentation in `README.md`.
- Contact information of the responsible developer in `README.md`.

> **Example:** Run the following command in a Linux console:
>
> ```bash
> man curl
> ```

---

## Monitoring

- The service exposes metrics endpoints upon request.
- The service can send metrics autonomously.
- **Health check status:**
  - **Minimal:** Response:
    - "OK / Not OK"
  - **Preferred:**
    - "Running, ready to respond" (**Liveness**)
    - "Operational / Experiencing technical issues (e.g., logging stopped, configuration errors on reload)"
    - "All good / Issues with dependencies (e.g., DB connection, missing backend, queue overflow)" (**Readiness**)
- **Metrics:**
  - **Minimal:**
    - Number of requests/operations per time interval.
    - Average processing time per interval.
    - JVM parameters (for Java applications).
  - **Preferred:**
    - Processing time percentiles.
    - Error count per interval.
    - Any additional operational insights.
- The service returns different error codes for different types of errors.
- Error codes comply with standard HTTP codes/Linux exit codes.

---

## Configurability & Logging

- The service retrieves parameters from environment variables.
- Environment variables take precedence over config file parameters.
- All units of measurement (especially time) are standardized or explicitly defined (e.g., `Xd Yh` for Java `Duration`).
- The service supports "request trace ID":
  - Accepts, logs, and propagates it through subrequests.
- When a `User-Agent` is present, requests to other services include at least the service name and version.
- The service writes logs to `STDOUT/STDERR`.
- The service logs start/stop/reload events concisely to `syslog`.
- The service writes logs to dedicated log files (configurable).
- Human-readable logs for operations.
- Separate `access` and `error` logs.
- Stack traces are logged in `error.log`.
- The service handles errors and logs meaningful error messages instead of raw stack traces.
- Logs are categorized into system logs and business logic logs.
- The service supports "transport logs":
  - Raw incoming requests with parameters.
  - Processed requests with substitutions.
  - All headers, request bodies, and responses.
- If the service needs to handle log rotation itself, it does so with configurable settings (preferably with compression, otherwise documented in `README.md`).
- Configurable logging levels.
- Independent log level configuration per log type.
- Logging settings can be updated without restart (via env variable, config reload, API request, or system signal).
- Support for remote log storage (Elasticsearch, `rsyslog`).
- Supports simultaneous local and remote logging.
- Different log types can be stored separately.
- Supports resource limits (configurable, documented in `README.md`).

---

## Build & Operation

- The service is packaged as a deployable entity (package, archive, `venv`, container, etc.).
- The deployable entity includes `README.md`, ideally with contents available via `man`.
- All dependencies are explicitly versioned (`>=` at a minimum).
- The service handles `SIGTERM` and other OS signals, ensuring graceful shutdown after completing the current task.
- Horizontally scalable (no race conditions or monopolization of shared resources).
- The service prevents multiple instances on the same host **or** supports concurrent instances on the same host correctly.
- If consuming queues/events, format changes should not cause crashes.
- Failed tasks are retried `N` times:
  - Retries and intervals are configurable.
  - Failures are logged or moved to a **Dead Letter Queue**.
  - The service can reprocess DLQ on demand.
  - Failures do not cause crashes.
- Supports an environment variable to limit the max number of tasks processed before shutdown.
- No hardcoded IPs, hostnames, service names, or ports.
- **Database & queue handling:**
  - Supports read replicas.
  - Can identify and switch between master and replicas.
  - Handles database/queue reconnection without crashing.
  - If a connection fails, it returns a meaningful error and attempts reconnection (`X` interval, configurable).
  - If external load balancing is required, this is explicitly documented in `README.md`.
- **Service-to-service communication:**
  - Supports multiple instances on different hosts.
  - Can detect and handle service availability changes.
  - If all service instances are unavailable, it returns a meaningful error and retries (`X` interval, configurable).
  - If external load balancing is required, this is explicitly documented in `README.md`.

---

## Database & Deployment

- Deployment planning must include database migration strategy.
- For **24/7 services** with no full downtime:
  - DB changes should follow either:
    - **DB first, then code** (ensure old code works with the new DB).
    - **Code first, then DB** (ensure new code works with the old DB).
- Compatibility logic should be added where necessary.
  - Temporary compatibility code should be removed in the next release **or** prepared in two sequential releases.
- **Destructive changes** (dropping fields/tables, mass updates/deletes) should be a separate deployment phase after the main release.


