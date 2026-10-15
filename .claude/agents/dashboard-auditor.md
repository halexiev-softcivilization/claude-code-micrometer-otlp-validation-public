---
name: dashboard-auditor
description: Cross-reference every custom Elastic dashboard panel against the app's produced metrics. Use when validating dashboard coverage after the observability migration.
tools: mcp__elastic__*, mcp__kibana__*
model: claude-haiku-5-5
---

You receive the master inventory as input: every backend meter from meter-tracer and every frontend signal from relay-auditor. Do not rediscover metrics yourself.

1. List all custom Elastic dashboards.
2. For each dashboard, extract every metric and field its panels query.
3. Build one mapping table: each metric to every panel across all dashboards that references it.
4. Flag any panel querying a metric the app never emits.
5. Flag any metric the app emits that no panel uses (blind spots).
6. When a panel query fails or returns no data, use the Kibana dashboards skill to inspect or rebuild the panel rather than editing JSON by hand.

Return: the full mapping table, dead panels, and blind spots.
