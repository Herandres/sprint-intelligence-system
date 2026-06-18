# Roadmap

## In production

### v1 — Claude Code pipeline (completed)
- ClickUp MCP retrieval via Claude Code CLI
- Python normalization pipeline (stdlib only, zero dependencies)
- 12 KPIs: completion rate, delivery rhythm, overdue, SLA alerts, workload per person
- Standalone HTML dashboard — no server, no CDN, printable to PDF
- Validated on Sprint 121: 843 tasks, 5 teams, 14-day sprint

### v2 — Claude.ai migration (completed)
- Full retrieval and KPI calculation moved to Claude.ai reasoning context
- No CSV, no local Python — entire flow runs inside a single conversation
- `sprint-intelligence-report` skill generates HTML Artifact on demand (`/sprint-report`)
- 5 new KPIs available: oldest overdue task, clients with zero overdue, risk of carryover to next sprint
- Claude Code retained as offline reference and backup environment
- Validated on Sprint 122: production go-live

---

## Next

### v2.1 — Sprint-over-sprint comparison
- Baseline JSON stored at sprint close
- Trend indicators: completion rate vs. previous sprint, workload shifts, SLA recurrence
- Early signal: clients appearing in SLA alerts two sprints in a row

### v2.2 — Proactive alerts
- Mid-sprint check triggered automatically (day 7 of 14)
- Alert if delivery rhythm drops below 0.7x before the sprint ends
- Notification via Slack MCP when SLA threshold is crossed

### v2.3 — Natural language queries
- "Who has the most overdue tasks this sprint?"
- "Which clients are at SLA risk?"
- "How does this sprint compare to the last three?"
- Agent answers directly from the intelligence payload without re-running the full retrieval

### v3 — Multi-project intelligence
- Extend beyond sprint cadence to project-level tracking
- Cross-sprint workload balance: detect people consistently overloaded across sprints
- Client health score: rolling 4-sprint view of SLA compliance per client
