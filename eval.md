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

## Tracking & Measurement Specialist

- **Scope:** mem20's measurement plan, event taxonomy, attribution, KPI definitions and tracking
  gaps — the layer that decides whether any number the project reports can be trusted. Evidence:
  `docs/gtm/launch-instrumentation.md`, `docs/gtm/utm-registry.csv`, `docs/gtm/mem20-campaign-plan.md`,
  `docs/index.html`, `2.0_roadmap/metrics_framework.md`, `2.0_roadmap/instrumentation_plan.md`,
  `mcp/health.py`, `mcp/server.py` (`metrics()`), `mcp/install_tracker.py`, `mcp/tools/marketing_tools.py`,
  `scripts/check_launch_gate.py`, `scripts/check_gtm_claims.py`, `docs/ops/metrics.json`,
  `docs/ops/surface-measurement-2026-10-31.md`, `.github/workflows/mcp-verify.yml`.

- **The headline: there is no attribution, and the campaign's primary KPI is therefore unmeasurable.**
  - `docs/index.html` (the live public page) carries **zero** analytics loaders — I re-verified at this
    HEAD: 0 matches for `gtag` / `googletagmanager` / `plausible` / `umami` / `posthog` / `matomo` /
    `fathom`. A `utm_*` query string is recorded **nowhere**.
  - Consequence: all 12 tracked links in `docs/gtm/utm-registry.csv` are **inert**. They are correct and
    they survive the redirect chain, but they produce no per-channel data. The campaign plan's primary
    KPI ("referral sessions, split by channel") cannot be produced at all.
  - The repo says this itself (`launch-instrumentation.md` §1) — that honesty is a genuine strength, but
    it does not change the fact that the measurement plan is a plan, not a measurement.

- **The event taxonomy is a specification, not something that fires.**
  - `page_view`, `cta_click_github`, `install_copy`, `partition_demo_view`, `promotion_gate_view`,
    `benchmark_view`, `limits_view` are defined in `mem20-campaign-plan.md` §3 and explicitly labelled
    "spec for the next page version".
  - There is **no dataLayer, no `event_id`, no consent-mode signal** anywhere in the repo (grep: 0 hits in
    a tracking context). So there is no dedup key, no consent gating, and no event contract to implement
    against — only a list of names.

- **The attribution rule is unsatisfiable as shipped.** `mem20-campaign-plan.md` §4 states: "every
  reported number must trace to a `utm_content` value or a platform-native metric." With no snippet, the
  primary metric cannot trace to `utm_content`; the rule can only be met by platform-native rank. A rule
  that the primary metric cannot satisfy is a rule that will be quietly broken at readout time.

- **The funnel is undefined end-to-end.** `launch-instrumentation.md` §5 splits S1–S9 honestly:
  - **S4 (star/fork)** is the only cleanly observable conversion.
  - **S5 (clone)** is observable but **automation-contaminated** — 901 unique cloners against 8 unique
    viewers is not human behaviour. Reporting clones as interest would be a miscount.
  - **S6–S9 (install, first handshake, activation, D1/D7/D30 retention)** are **unobservable**. Activation
    is *defined* (source install + server start + one successful tool call against a non-empty store) but
    has no event at that boundary, so activation rate cannot be reported.

- **KPI definitions are not operationalized.**
  - `metrics_framework.md` sets a north-star ("recall trust × safe grounding rate") but **neither factor
    is emitted as a number** — it is aspirational, not observable.
  - Targets like "error rate < 1%" and "per-tool error < 2%" are **not computable**: `metrics()` returns
    process-lifetime counters with no window and no rate, and a restart zeroes them. A raw count is not a
    rate.
  - `contamination_rate` is listed as a "0.0 hard gate" but is only *reported*; nothing fails on it.

- **Four different numbers for the same KPI — the surface count has no single source of truth.**
  | Source | Says |
  |---|---|
  | `.github/workflows/mcp-verify.yml:47` (CI pin) | **311** tools |
  | `docs/gtm/claim-ledger.md:7` | **270** tools / **27** domains |
  | `docs/ops/surface-measurement-2026-10-31.md` | **270** tools / **28** registration groups; "27 domains" **REFUTED as a code-derived number** |
  | `docs/index.html` | **311** tools across **39** domains |
  | `docs/gtm/launch-instrumentation.md` §7 | records CI pin **272** vs ledger **270** |
  This is the exact failure mode that matters: a KPI with four definitions cannot be tracked, and a
  miscounted surface misleads every downstream claim. The measurement doc even names the counting method
  as the unresolved variable (272 registration sites / 271 literals / 270 distinct keys / 282 unique
  quoted names).

- **The advertised analytics tool cannot measure anything.** `mcp/tools/marketing_tools.py`
  `_mkt_analytics` returns the strings "Requires GA4 API credentials." / "Requires Plausible API key." —
  it performs no read. `_mkt_social_analytics` is the same. The MCP surface advertises web/social
  analytics that do not exist, which is a tracking gap *inside the product*.

- **Install telemetry is opt-in, off by default, and has no server-side dedup.**
  - `mcp/install_tracker.py` fires only if `MEM20_INSTALL_WEBHOOK` is set; `MEM20_NO_TELEMETRY` disables
    it. So installs / retention / DAU are **not measured** — and `metrics.json` correctly reports them as
    absent rather than estimating.
  - **Good primitive:** a stable `install_id` plus a `first_run_reported` marker gives real client-side
    dedup for first-run events.
  - **Gap:** the network beacon carries **no `event_id` / idempotency key**, so a retried POST would
    double-count installs at the collector. There is also no retention policy for `installs.jsonl`.

- **The launch gate makes the analytics decision explicit — but only checks presence, not firing.**
  `scripts/check_launch_gate.py` gate 6 requires a privacy-respecting snippet **or** a recorded waiver
  (`--waive-analytics`). That is the right shape: "no attribution" becomes a recorded decision, not a
  silent gap. But the check is a marker-string match on served HTML — a snippet that loads and never
  fires an event would pass. Presence ≠ measurement.

- **The UTM registry is not machine-validated.** `utm-registry.csv` is only referenced by
  `check_gtm_claims.py` as a file to scan; nothing checks that `utm_content` is unique per placement, or
  that the CSV matches the campaign plan's own link table. Two copies of the same 12 links exist
  (`utm-registry.csv` and `mem20-campaign-plan.md` §3) with no cross-check, so they can drift. Note the
  deliberate trap: dev.to and Medium **share** `utm_content=deep_dive` and are separated only by
  `utm_source` — correct by design, but easy to mis-aggregate.

- **Redirect downgrades https → http.** The 301 from `bythewayz66-glitch.github.io/mem20/` to
  `http://mem20.com/` preserves the query string but downgrades the scheme. A client that refuses the
  downgrade never completes the hop — a silent session loss that would look like "no traffic from that
  channel" once a snippet exists.

- **No consent / privacy layer for measurement.** No consent banner, no consent-mode signals, no data
  retention statement for the install log. For a product whose pitch is auditability, adding a snippet
  without consent gating would be a compliance gap (GDPR/CCPA).

- **`test_collection` is a count, not a pass rate.** `metrics.json` reports 1,814 collected with 29 errors
  and a whole-repo collection that aborts (`exit 3`). It is labelled "collection count only" in the JSON,
  but it is trivially misread as "1,814 passing" wherever it is surfaced.

- **Genuine strengths worth crediting:**
  - **Honest nulls.** `metrics.json` distinguishes "not measured" from zero (rate-limited GitHub API →
    `null`), and its `telemetry_disclaimer` states plainly that no visitor/install telemetry exists.
  - **Disclosed disagreement.** The `reconciliation` block reports that 3 of 8 checks disagree instead of
    smoothing them — the right instinct for a measurement layer.
  - **A real measurement plan exists.** `launch-instrumentation.md` is unusually honest about what cannot
    be measured, and the launch gate turns the analytics decision into an explicit, recorded choice.
  - **A deterministic claim gate.** `check_gtm_claims.py` reads the expected surface from the CI pin
    rather than hardcoding it — a gate that cannot drift from its own source.

- **Concrete suggestions:**
  1. **Add a privacy-respecting snippet** (Plausible / Umami / self-hosted counter) to `docs/index.html`.
     This is the single change that makes the campaign measurable; until it lands, mark every per-channel
     KPI "not measured" in the readout rather than reporting a proxy.
  2. **Ship the event taxonomy as a real contract:** for each event — name, trigger, required params
     (`utm_source/medium/campaign/content`, page, placement) and a stable `event_id` for dedup; gate every
     tag behind consent mode before it fires.
  3. **Add a `metric_contract` block** to `/metrics` and `metrics.json`: unit, window, source,
     `measured|not_measured`, `stale_after`. Makes "null ≠ 0" machine-checkable.
  4. **Emit rates with a rolling window**, not process-lifetime counts: `request_error_rate`,
     `tool_error_rate` per tool — so the framework's <1% / <2% targets become directly testable.
  5. **Pin ONE surface number and derive every other mention from it.** Add a CI check that fails when any
     doc's tool/domain count disagrees with the pin (311 vs 270/27 vs 311/39 vs 270/28 today).
  6. **Add an `event_id` / idempotency key to the install beacon** so a retried webhook POST cannot
     double-count installs; keep `install_id` + the first-run marker as the dedup authority.
  7. **Make the launch gate's analytics check a *firing* check, not a presence check:** assert the snippet
     loads **and** at least one `page_view` carrying a `utm_content` is emitted in a headless run.
  8. **Validate `utm-registry.csv` in CI:** unique `utm_content` per placement (or an explicit
     `shared_with` column for dev.to/Medium), and cross-check it against the campaign plan's link table so
     the two cannot drift.
  9. **Fix the https→http redirect downgrade** at the Pages level — a refused downgrade loses the session
     and the `utm_*` with it.
  10. **Join the two metric worlds with a shared key** (`head_sha` + `generated_at` + `utm_content`) so a
      runtime effect can be correlated with a code change; today runtime `/metrics` and repo `metrics.json`
      share no identifier.
  11. **Label `test_collection` "collected, not passed"** everywhere it surfaces, and add a real pass-rate
      metric once whole-repo collection is fixed.
  12. **Add a consent + retention note** for `installs.jsonl` and any future snippet (GDPR/CCPA), since the
      product's pitch is auditability.

- **Severity:** **high** for the campaign — the primary KPI is unmeasurable and the stated attribution rule
  is unsatisfiable, so any per-channel number reported at launch would be unsourced. **Medium** for product
  telemetry — the instrumentation is honest and well-documented but not yet actionable (no rates, no
  window, no durable sink, no join key).

- **Open questions:**
  - Is the analytics snippet an owner decision (as `launch-instrumentation.md` §6 says)? Has the waiver
    been recorded, or is the launch gate currently blocking on gate 6?
  - Which surface number is authoritative — **311** (CI pin) or **270** (claim ledger)? Who owns
    reconciling the four published values?
  - Is `mkt_analytics` intended to be implemented, or should it be removed from the advertised surface?
  - Should install telemetry stay opt-in, and what is the retention policy for `installs.jsonl`?

---
