# Sprint Intelligence System

AI-powered operational intelligence pipeline that converts project management data into standalone HTML dashboards with real KPIs, team-level health indicators, SLA alerts, and executive recommendations.

---

## What it does

At the start of each sprint, the system:

1. **Detects the active sprint** across multiple delivery teams via ClickUp MCP
2. **Retrieves all tasks** with full pagination — statuses, assignees, due dates, clients
3. **Normalizes the data** inline (status mapping, timezone conversion, ownership detection)
4. **Runs the Python pipeline** to calculate 12 KPIs and generate outputs
5. **Produces a standalone HTML dashboard** — no server, no CDN, opens in any browser

---

## Architecture

The system operates in two modes depending on the environment:

### Mode A — Claude.ai (production)

```
User opens new conversation
        │
        ▼
Project Instructions (prompt) loaded automatically
        │
        ▼  (sequential: team 1 → team 2 → ... → team 5)
Agent detects active sprint → ClickUp MCP
Agent retrieves all tasks → ClickUp MCP
        │
        ▼
Agent calculates KPIs in reasoning context
        │
        ▼
Intelligence Report presented in chat
        │
        ▼  (on demand — user types /sprint-report)
sprint-intelligence-report skill
        │
        ▼
HTML Artifact generated in Claude.ai → print to PDF
```

### Mode B — Claude Code CLI (reference / offline)

```
Claude Code (MCP) ─── ClickUp API
        │
        ▼  (sequential, 5 teams)
  Data retrieval + inline normalization
        │
        ▼
  CSV dataset (12 columns)
        │
        ▼
  run_pipeline.py  ──►  sprint_kpis.json
        │              sprint_intelligence.json
        ▼
  Sprint_Dashboard.html  ← standalone, no internet required
```

---

## KPIs calculated

| KPI | Description |
|-----|-------------|
| Total tasks | Level-3 work items only (filters out epics and objectives) |
| % Completed | Completed ÷ total × 100 |
| Overdue | Due date < today + not completed + not paused |
| In progress / Not started / No owner | Count by normalized status |
| Sprint health | Green / Yellow / Red with defined thresholds |
| Delivery rhythm | Completion % ÷ days consumed % — threshold 0.8x |
| SLA alerts by client | Clients with ≥ 2 overdue tasks |
| Workload per person | Assignee breakdown: total, completion %, overdue, clients |
| Load concentration | Top 2 people vs. total — alert if > 60% |
| KPIs per team | Full breakdown by delivery team |

---

## Stack

| Component | Technology |
|-----------|-----------|
| AI orchestrator | Claude.ai with MCP (ClickUp integration) |
| Data retrieval | ClickUp API via MCP — sequential, paginated |
| Normalization | Python inline script (stdlib only, zero dependencies) |
| Pipeline | `run_pipeline.py` — one command, pure stdlib |
| Output | Standalone HTML + JSON (no CDN, no server, no internet) |
| Environment | Windows / Claude Code CLI |

---

## Output files

| File | Contents |
|------|----------|
| `Sprint{N}_Dashboard.html` | Standalone dashboard — embed, share, print to PDF |
| `sprint{N}_kpis.json` | KPIs + workload + team breakdown + SLA alerts |
| `sprint{N}_intelligence.json` | Executive summary, recommendations, early warnings |
| `sprint{N}_ops_clean.csv` | Normalized dataset (level-3 tasks, valid rows only) |

---

## Intelligence payload (sprint_intelligence.json)

```json
{
  "titular_ejecutivo": "Sprint 90% complete · delivery rhythm 0.91x · health GREEN",
  "semaforo_por_equipo": { "team_a": "GREEN", "team_b": "YELLOW" },
  "alertas_sla": ["Client X — 3 overdue tasks"],
  "workload_top": [{ "assignee": "...", "total": 18, "completed_pct": 94 }],
  "recomendaciones": ["Redistribute 2 overdue tasks from person A to person B"],
  "early_warnings": ["3 tasks not started with due date in 2 days"]
}
```

---

## Visual report skill

The system includes a Claude.ai skill (`sprint-intelligence-report`) that generates a branded HTML Artifact directly in the conversation.

**How it works:**

1. The agent retrieves and calculates all KPIs (Mode A flow above)
2. User types `/sprint-report` in the Claude.ai conversation
3. The skill generates a standalone HTML dashboard as a Claude.ai Artifact
4. User opens it in the browser — printable to PDF with one click

**What the Artifact includes:**
- Executive headline + sprint health badge (Green / Yellow / Red)
- KPI summary cards per team
- Overdue task breakdown by assignee and client
- SLA alert list
- Workload table with completion % per person
- Recommendations and early warnings

This means the full report — from raw ClickUp data to a shareable PDF — runs entirely within a single Claude.ai conversation, with no local environment required.

---

## Production validation

Validated on a full 14-day sprint across 5 delivery teams:

| Metric | Result |
|--------|--------|
| Tasks tracked | 843 |
| Completion rate | 90.87% |
| Delivery rhythm | 0.91x (above 0.8x threshold) |
| Sprint health | GREEN |
| Teams covered | 5 |
| Pipeline runtime | < 5 seconds |

---

## Design decisions

**Why sequential retrieval (not parallel)?**
Parallel calls to ClickUp's API cause "Failed to fetch" errors. Each team is retrieved one at a time, with pagination until count < 100 per page.

**Why stdlib only in the pipeline?**
Zero-dependency Python means the pipeline runs on any machine without a virtual environment or pip install.

**Why standalone HTML?**
The dashboard must be shareable via email or Slack, printable to PDF, and viewable without a browser extension or server. No CDN dependency, no external calls.

**Why Claude.ai as orchestrator?**
MCP integration gives the agent direct access to the project management tool. The agent detects the active sprint, handles pagination, normalizes data inline, and runs the pipeline — all from a single prompt.

---

## Author

Hernán Castaño · [LinkedIn](https://www.linkedin.com/in/hernan-castano-triario)
