# Project

Spring Boot backend + Angular frontend. Observability migrated from the Elastic APM agent to Spring Boot's Micrometer observation layer exporting OTLP to a local Elastic stack. The Angular frontend sends metrics, traces, and logs to a backend relay controller that forwards them to Elastic. Both the frontend-to-backend relay and the backend-to-Elastic OTLP path are in scope as two separate stages, each fully verified before the other.

Local app and local Elastic stack are both running. Goal: validate the migration against best practices and reach parity with the old APM-agent behavior, then move the same setup to the cloud environment.

# Commands

- Start the app with the project's usual start script.
- Run tests with the project's test command.
- Query Elastic only through the Elastic MCP — never curl by hand when MCP is available.
- Elastic skills are installed: the Elastic onboarding skill and the Kibana dashboards skill. Subagents must invoke them through the skill mechanism rather than reinventing index setup or dashboard creation. `elastic-checker` uses the onboarding skill's index-template and data-stream knowledge when a query returns empty, to distinguish a missing signal from a mapping mismatch. `dashboard-auditor` leans on the Kibana skill when inspecting or rebuilding a panel.

# Conventions

- Micrometer meters use the global composite registry unless explicitly constructed otherwise.
- OTLP exporter endpoint and headers live in application config.
- Frontend relay posts to the backend controller path — find it in code rather than assuming it.
- Subagent definitions live in `.claude/agents/`.

# Workflow

Main session is controller only: delegate every investigation, verification, and fix to subagents. Never read files or run commands yourself. Spawn subagents, receive summaries, decide next steps, keep context to decisions and results.

1. Spawn `meter-tracer` and `relay-auditor` in parallel.
2. Spawn `elastic-checker`, passing it the meter list from meter-tracer.
3. Spawn `dashboard-auditor`, passing it the master inventory (backend meters + frontend signals from relay-auditor) as its starting input. It maps each metric to every panel across all dashboards in one pass.
4. If anything is broken, spawn `fixer` with a single-item brief, then re-verify with the relevant checker. Repeat until clean or stuck.
5. Stuck rule: if fixer has cycled three times on the same item without resolution, stop, report the item with everything tried, and wait for the user.

Reporting: only what is broken. If everything checks out, say "all good" and stop.

# Hard constraints

- Do not rename a metric or attribute key without flagging it first — dashboards depend on them.
- Do not edit generated or build directories.
- Do not deploy or push anything.
- Misconfigurations (wrong exporter setting, missing registry binding, relay dropping on forwarding errors, dashboard query using a stale field name) are fixed in place by `fixer`, not merely reported.
