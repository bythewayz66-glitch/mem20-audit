# mem20 — Group Evaluation

Shared evaluation log for the **mem20** project (`github.com/bythewayz66-glitch/mem20`).
Each contributor appends their own `## <name>` section at the bottom. Append-only — never
rewrite or reorder another contributor's section. `mem20` is read-only source of truth.

---

## Analytics Reporter

- **Scope:** mem20's data/analytics surface — exposed metrics & KPIs, telemetry/logging,
  dashboard data flow, reporting gaps, and data trustworthiness. Evidence: `metrics_collector.py`,
  `mcp/health.py`, `mcp/server.py` (`metrics()`), `mcp/install_tracker.py`, `dashboard/`,
  `docs/ops/metrics.json`, `2.0_roadmap/metrics_framework.md`, `2.0_roadmap/instrumentation_plan.md`,
  `docs/finance/compute_model.py`.

- **What the project exposes (KPIs):**
  - **Runtime `/metrics`** (`mcp/health.py` → `server.metrics()`): `uptime_seconds`, `request_count`,
    `request_errors`, `tools_registered`, `memory_system_available`, per-tool `tool_calls`/`tool_errors`
    + `tool_calls_total`/`tool_errors_total`, and an `event_taxonomy` (`tool.call`/`tool.success`/`tool.error`).
  - **Engine audit block** (same endpoint, 30s cache via `_refresh_audit`): `contamination_rate`,
    `grounding_coverage`, `grounded_facts`, `simulated_facts`, `engine_audit_available`.
  - **Repo/ops metrics** (`metrics_collector.py` repo mode → `docs/ops/metrics.json`): `commit_velocity`
    (commits/day, distinct authors, busiest day), `pr_cycle_time`, `issues` (open/closed), `branch_divergence`
    (ahead/behind, at-risk commits), `tracked_file_growth`, `test_collection`.
  - **Install telemetry** (`mcp/install_tracker.py`): opt-in install pings (install_id, version, platform,
    channel) → local JSONL + optional `MEM20_INSTALL_WEBHOOK`; collector serves `/metrics`, `/recent`, `/export`, HTML dashboard.
  - **Finance model** (`docs/finance/compute_model.py`): ACV/TAM/SAM/SOM with a self-verifying, append-only reconciliation ledger.

- **Findings:**
  - **Two disjoint metric worlds, no join key.** Runtime `/metrics` (process counters) and repo metrics
    (`metrics.json`, git-derived) never share an identifier or timestamp, so no single view answers
    "is the product healthy *and* is the work progressing". `dashboard/` reads only the runtime side.
  - **Dashboard shows a fraction of what exists.** `dashboard/src/App.jsx` renders only
    `tools_registered`, `request_count`, `request_errors`, `memory_system_available`, `simulated_facts` —
    it drops `contamination_rate`, `grounding_coverage`, `tool_errors_total`, and the whole event taxonomy.
    The two headline safety KPIs are collected but never displayed.
  - **Counters are process-lifetime only.** `metrics()` returns in-memory counters with no persistence,
    no reset window, and no rate — a restart zeroes them, so `request_errors` is a raw count, not an error *rate*.
    The framework's own target ("error rate < 1%") is therefore not computable from the endpoint as shipped.
  - **No durable sink.** `MEM20_METRICS_SINK` (`none`/`prometheus`/`otlp`) is planned in
    `instrumentation_plan.md` but not implemented — no time series, so no trend, alerting, or regression gate.
  - **Telemetry is opt-in and off by default.** `install_tracker.py` fires only if `MEM20_INSTALL_WEBHOOK`
    is set; `metrics.json` states plainly "no visitor or install telemetry". So installs/retention/DAU are
    **not measured** — and correctly reported as absent, not estimated.
  - **Honest nulls and reconciliation are a genuine strength.** `metrics.json` distinguishes
    "not measured" from zero (rate-limited GitHub API → `null`), reports both UTC and author-local day buckets,
    and keeps a `reconciliation` block that *discloses* disagreements (3 of 8 checks disagree) instead of smoothing them.
  - **`test_collection` is a count, not a pass rate** — whole-repo collection aborts (`exit 3`), scoped run
    collects 1814 with 29 errors. Easy to misread as "1814 passing".
  - **No data-quality/validation layer.** No schema check, freshness SLA, or anomaly detection on any metric;
    `metrics.json` is a point-in-time snapshot (`generated_at`) with no automated refresh cadence.

- **Concrete suggestions:**
  1. **Add a `metric_contract` block** to `metrics.json` (and `/metrics`): for each metric — unit, window,
     source, `measured|not_measured`, and a `stale_after`. Makes "null ≠ 0" machine-checkable.
  2. **Implement `MEM20_METRICS_SINK=prometheus`** (text exposition on `/metrics`) as the first durable sink —
     cheapest path to trends, alerting, and a release regression gate.
  3. **Emit rates, not just counts:** `request_error_rate = request_errors/request_count` and
     `tool_error_rate` per tool, with a rolling window, so the framework's <1% / <2% targets are directly testable.
  4. **Surface the safety KPIs in the dashboard:** add `contamination_rate` and `grounding_coverage` cards
     (they are the north-star "recall trust × safe grounding rate") and a per-tool error table.
  5. **Join the two worlds:** stamp every `metrics.json` run and every `/metrics` scrape with the same
     `head_sha` + `generated_at`, so a dashboard can correlate a code change with a runtime effect.
  6. **Add a data-quality gate** (CI job): assert `metrics.json` parses, required keys present, `generated_at`
     within SLA, and `reconciliation.disagree` count is reported — fail the build on silent drift.
  7. **Label `test_collection` explicitly as "collected, not passed"** in the dashboard/README to prevent the
     pass-rate misread; add a real pass-rate metric once whole-repo collection is fixed.
  8. **Define the north-star as a single computed number** (`recall_trust × safe_grounding_rate`) and publish it
     in `/metrics`, so the roadmap's stated goal is observable rather than aspirational.

- **Severity:** medium — the instrumentation is honest and well-documented, but it is **not yet actionable**:
  no durable sink, no rates, no trend, no join between runtime and repo metrics, and the two most important
  safety KPIs are collected but not displayed. Trust is high; actionability is the gap.

- **Open questions:**
  - Is `MEM20_METRICS_SINK` intended for 2.0 GA, and is Prometheus the chosen default?
  - Should install telemetry stay opt-in (privacy-first) or move to opt-out with anonymized aggregates?
  - Who owns the weekly feedback loop (`instrumentation_plan.md` §4) and where does the export land?
  - Is `contamination_rate` a hard gate (0.0) in CI, or only reported?

---
