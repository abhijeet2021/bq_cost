# BigQuery Billing Intelligence Dashboard PRD

## 1. Purpose
Build a BigQuery Billing Intelligence dashboard and insight layer that makes cloud query spend understandable at both executive and operator level.

This solution should answer three things clearly:

1. **Who is driving cost?**  
   Identify IAM users, service accounts, or workloads contributing the most BigQuery spend.

2. **What is driving cost?**  
   Surface the most expensive queries, recurring cost culprits, waste patterns, and abnormal spikes.

3. **Where should leaders act first?**  
   Convert raw billing + query telemetry into simple, ranked, decision-ready insights.

The output must be **CEO-consumption ready**, but still useful for engineering, data platform, and project owners.

This dashboard is not meant to be a generic billing report. It should function as a **BigQuery Cost Governance Cockpit** that makes accountability, waste detection, and savings opportunities visible in seconds.

---

## 2. Design Principles

### 2.1 Executive-first
The dashboard should feel like a decision screen, not an analyst worksheet.

- Minimal noise
- Sharp hierarchy of information
- Very few visuals per page
- Strong use of KPI tiles, pills, ranked lists, and compact trend cards
- Every visual must answer a real business question

### 2.2 Not squarish
The layout should avoid heavy boxy dashboard styling.

Use:
- Rounded cards
- Pill-based filters
- Capsule KPI chips
- Soft sections with breathing space
- Horizontal story flow where possible
- Compact strip trends instead of dense chart grids

Avoid:
- Harsh square containers
- Too many tables on landing page
- Dense matrix visuals unless on drilldown pages
- Excessive legends and filter clutter

### 2.3 Dummy-proof
A non-technical leader should understand the dashboard in under 60 seconds.

That means:
- Plain language labels
- Business wording over platform jargon where possible
- Tooltips that explain why something matters
- Clear red/yellow/green signals for abnormal spend
- Direct action-oriented captions like:
  - `Top cost driver this month`
  - `Largest spike contributor`
  - `Repeated expensive workload`
  - `Likely optimization target`

### 2.4 Low AI noise
AI must assist interpretation, not dominate the dashboard.

Use AI only for:
- concise insight summaries
- anomaly explanation suggestions
- query categorization
- likely waste pattern tagging
- recommended action phrasing

Do **not** use AI for:
- long essay summaries
- chatty dashboard text
- over-explaining obvious metrics
- speculative claims without evidence

AI copy should read like this:
- `Spend spike mainly came from 3 users running large scan jobs on customer analytics datasets.`
- `User A shows repeated full-scan query behavior across 7 days.`

Not like this:
- `Our intelligent engine has identified a meaningful cost optimization opportunity...`

### 2.5 Governance-first framing
The dashboard must connect **cost control to data governance**.

Every major section should help answer at least one of these questions:
- Which IAM identities need review?
- Which workloads show poor query hygiene?
- Which spend lacks clear ownership?
- Which optimization action will reduce cost fastest?
- How much avoidable spend exists today?

---

## 3. Primary Users

### 3.1 CEO / CTO / Finance leadership
Need fast answers on:
- monthly spend trend
- who caused the increase
- whether the spike is one-time or recurring
- which teams or users need intervention
- where savings are likely

### 3.2 Data Platform / BI / Governance team
Need:
- query-level evidence
- IAM/user-level tracking
- recurring waste patterns
- user education targets
- enforcement opportunities

### 3.3 Project owners / engineering managers
Need:
- visibility into their users or service accounts
- expensive queries tied to owner
- drilldown into datasets / workloads / frequency
- guidance on what to fix first

---

## 4. Core Business Questions

The dashboard must answer these questions without needing SQL.

### Executive questions
- What is the total BigQuery cost this month?
- How does it compare to last month?
- Which IAM identities contributed most to the increase?
- Is the spike driven by a few users or broadly distributed?
- Which teams or workloads should be reviewed first?

### Governance questions
- Which IAM users or service accounts consistently generate high query cost?
- Which queries are most expensive by total cost?
- Which queries are expensive because of repeated execution versus one-time large scans?
- Which identities show sudden month-over-month increase?
- Which workloads likely indicate poor query hygiene?
- Which workloads should be prioritized for governance action?
- Which spend appears avoidable?

### Diagnostic questions
- Which query text / normalized query pattern is the main culprit?
- Which tables or datasets are most often scanned by high-cost jobs?
- Are expensive queries tied to dashboards, ad hoc exploration, pipelines, or service accounts?
- Are there users whose cost is high but business output is low?

---

## 5. Recommended Data Model

Assumption: billing table already exists in BigQuery.

To make insights reliable, create a semantic model with these layers.

### 5.1 Source layers

#### A. Billing fact
Main billing export table with at least:
- billing_date
- project_id
- service_description
- sku_description
- cost
- currency
- labels / tags if available
- invoice month
- usage_start_time
- usage_end_time
- resource metadata if available

#### B. BigQuery jobs metadata
From `INFORMATION_SCHEMA.JOBS*` or audit logs where possible:
- job_id
- creation_time
- end_time
- project_id
- user_email
- principal subject / service account
- statement_type
- query text
- total_bytes_processed
- total_slot_ms if available
- total_bytes_billed
- cache hit
- destination table
- referenced tables
- reservation info if relevant
- labels

#### C. Identity mapping
Optional but highly recommended:
- IAM principal → human owner / team / function
- service account → owning application / team
- groupings like BI, DS, Marketing, Engineering, Finance

This is critical for CEO-ready views because raw service account names are ugly and confusing.

### 5.2 Curated model tables

#### `fact_bq_cost_by_job`
One row per job with estimated or allocated cost.

Suggested fields:
- job_date
- job_month
- project_id
- user_email
- identity_type
- owner_name
- team_name
- service_account_flag
- job_id
- statement_type
- normalized_query_hash
- query_text_short
- total_bytes_billed
- total_slot_ms
- estimated_job_cost
- dataset_list
- table_list
- query_category
- waste_pattern_tag
- spike_flag

#### `fact_bq_cost_by_identity_daily`
Daily aggregated spend by IAM identity.

Suggested fields:
- date
- month
- user_email
- owner_name
- team_name
- identity_type
- query_cost
- query_count
- bytes_billed
- avg_cost_per_query
- costly_query_count
- anomaly_score

#### `fact_bq_cost_by_identity_monthly`
Monthly rollup for leadership and MoM tracking.

Suggested fields:
- month
- user_email
- owner_name
- team_name
- total_cost
- previous_month_cost
- mom_change_cost
- mom_change_pct
- query_count
- costly_query_count
- rank_by_cost
- rank_by_spike
- avoidable_cost_estimate
- governance_alert_count

#### `fact_expensive_queries`
Repeated or extreme individual query patterns.

Suggested fields:
- normalized_query_hash
- representative_query_text
- owner_name
- primary_user_email
- team_name
- total_cost
- execution_count
- avg_cost
- max_cost
- first_seen
- last_seen
- primary_tables
- waste_pattern_tag
- recommended_action

#### `dim_identity`
Readable IAM mapping table.

Suggested fields:
- principal_id
- user_email
- display_name
- owner_name
- team_name
- business_function
- manager_name
- identity_type
- active_flag

---

## 6. Cost Allocation Logic

This section matters because billing export is not always directly tied to exact query/user granularity.

### 6.1 Preferred approach
If job-level cost can be estimated using bytes billed and known pricing, allocate cost per job directly.

### 6.2 Alternative approach
If only aggregate billing cost is available, allocate cost proportionally using one of these drivers:
- bytes billed share per job
- slot-ms share per job
- reservation consumption share

### 6.3 Recommended rule
Use the most explainable model, not the fanciest one.

For leadership dashboards, it is better to say:
- `Estimated user cost allocated based on query bytes billed share`

than to present exact-looking numbers that nobody trusts.

### 6.4 Required transparency note
Include a small info note in dashboard footer or tooltip:

`User-level and query-level costs are allocated estimates derived from BigQuery job metadata and billing records.`

This protects trust.

---

## 7. Key Insight Categories

### 7.1 Top IAM cost drivers
Goal: show which identities consumed the most cost.

Metrics:
- total monthly cost
- share of total cost
- query count
- avg cost per query
- costly query count
- month-over-month delta

Visual style:
- ranked pill list with avatar/initial + name + cost + delta pill
- optional small sparkline beside each user

### 7.2 Cost spike contributors
Goal: show who caused the month spike.

Metrics:
- incremental cost vs previous month
- spike contribution %
- whether spike is one-time or recurring

Visual style:
- waterfall or ranked delta bars
- badge pills: `new spike`, `recurring`, `stabilized`

### 7.3 Expensive query culprits
Goal: identify high-cost query patterns and surface specific optimizations.

Metrics:
- top single-run cost
- top repeated query cost
- total bytes billed
- execution count
- primary IAM identity
- dataset/table touched
- estimated savings from optimization
- optimization priority and effort

Visual style:
- compact ranked cards
- query text preview in 1–2 lines
- tag pills like `full scan`, `repeated`, `ad hoc`, `dashboard`, `likely optimization`
- inline optimization box per query showing: recommended action, plain-language alternative, savings pill, priority pill, effort pill
- total savings banner at top of Query Explorer page
- "View SQL" button opens modal with full query + optimization detail

### 7.4 Waste pattern insights
Goal: highlight likely avoidable spend.

Potential rules:
- repeated full-table scans
- very high bytes billed with low output frequency
- same query executed many times with little variation
- heavy ad hoc exploration by same identity
- dashboard queries with weak caching behavior
- broad `SELECT *` patterns
- queries scanning non-partitioned large tables
- development / testing behavior on production-scale tables

Visual style:
- insight cards with icon + plain language explanation

### 7.5 Monthly trend by identity
Goal: show overall cost movement and ownership.

Metrics:
- monthly total cost
- monthly top contributors
- number of identities with abnormal growth

Visual style:
- overall trend line on top
- stacked contribution by top identities or teams
- small pills for top spike drivers in selected month

### 7.6 Governance signals
Goal: convert technical telemetry into governance actions.

Examples:
- queries scanning non-partitioned tables
- queries reading entire datasets
- workloads with no owner mapping
- sudden new entrants into top spenders
- repeated queries with poor filtering
- excessive service account-driven cost
- scan-heavy queries with low business value indicators

Output style:
- concise signal card
- estimated cost impact
- suggested action
- severity pill: `needs review`, `high impact`, `urgent`

---

## 8. Dashboard Information Architecture

Recommend a **3-layer dashboard**.

## Page 1: Executive Overview
Purpose: fast story for CEO / CTO / Finance.

### Top strip KPIs
- Total BigQuery cost this month
- MoM change %
- Top spike contributor
- Number of high-risk IAM identities
- Potential savings opportunity estimate
- Governance alerts count

These should be in long rounded cards or pills, not square tiles.

### Main visuals
1. **Monthly cost trend**  
   Show last 12 months with spike markers.

2. **Who drove this month’s cost**  
   Top 5 IAM identities by cost.

3. **Who caused the increase**  
   Top 5 IAM identities by MoM incremental cost.

4. **Key insight strip**  
   3–5 plain-language bullet insights generated from rules/AI.

Example insight wording:
- `47% of this month’s increase came from 2 identities in the analytics project.`
- `One repeated query pattern contributed 11% of monthly cost.`
- `Service accounts account for 63% of total spend.`

### Page 1 design note
This page should feel like a narrative briefing. No dense tables.

---

## Page 2: IAM / User Drilldown
Purpose: governance and accountability.

### Core sections
1. **Top users by cost**
2. **Top users by cost spike**
3. **User trend over time**
4. **User efficiency view**
   - high cost + low query count
   - high cost + high repeat frequency
   - high avg cost per query
5. **Governance review queue**
   - users with likely avoidable spend
   - users with sudden spikes
   - users with repeated bad query hygiene

### Recommended controls
- month picker
- project pill filter
- team pill filter
- identity type pill (`human`, `service account`, `unknown`)
- cost severity pill (`all`, `high`, `critical`)

### Detail panel per selected user
- total cost
- query count
- avg cost per query
- top expensive query patterns
- top datasets scanned
- last 30-day trend
- recommended action
- avoidable cost estimate

---

## Page 3: Query Culprit Explorer
Purpose: pinpoint expensive workloads.

### Core sections
1. Top single-run expensive queries
2. Top repeated expensive query patterns
3. Query waste tags
4. Table/dataset hotspots
5. Query text preview with drill link

### Recommended columns for drill table
- user / service account
- estimated cost
- bytes billed
- executions
- query category
- waste tag
- dataset list
- first seen / last seen
- optimization recommendation

### Query text handling
Avoid dumping full raw SQL on main screen.

Use:
- shortened preview
- expandable modal or drill panel
- normalized query grouping

---

## 9. Visual Design System

### 9.1 Layout direction
The design language should be premium, light, and modern.

Use:
- rounded corners
- pill filters
- capsule badges
- clean typography
- muted separators
- strong spacing
- horizontal section rhythm

### 9.2 KPI cards
Preferred form:
- long rounded chips
- one main number
- one comparison pill
- one short caption

Example:
- `₹12.4L`  
  `+18% MoM`  
  `Driven mainly by analytics workloads`

### 9.3 Pills and badges
Use pills extensively for:
- filter controls
- identity type
- cost severity
- insight tags
- anomaly state
- team labels
- recurring vs one-time classification

Example pills:
- `High impact`
- `Recurring`
- `Full scan`
- `Service account`
- `Top 10%`
- `Needs review`

### 9.4 Chart style guidance
Prefer:
- soft line charts
- ranked horizontal bars
- contribution bars
- compact sparklines
- waterfall for spike attribution

Avoid:
- pie charts unless very limited categories
- cluttered legends
- crowded scatter plots on executive pages

### 9.5 Copy style
Every title must be plain and direct.

Good:
- `Top cost drivers this month`
- `Largest monthly spike contributors`
- `Most expensive repeated queries`
- `Users needing review`

Avoid:
- `Cloud analytics cost performance observability layer`

---

## 10. Insight Logic Rules

These rules can power both static insights and Claude Code orchestration.

### 10.1 High-cost IAM rule
If user monthly cost > threshold or in top N percentile:
- tag as `high-cost identity`
- explain with trend context

### 10.2 Spike driver rule
If user MoM increase contributes > X% of account/project spike:
- tag as `spike contributor`

### 10.3 Repeated expensive query rule
If normalized query runs many times and cumulative cost is high:
- tag as `repeated culprit`

### 10.4 Single expensive job rule
If one job cost crosses severe threshold:
- tag as `one-time large scan`

### 10.5 Waste suspicion rule
If query has very high bytes billed and known anti-pattern signatures:
- tag as `likely optimization target`

### 10.6 Service account dominance rule
If service accounts contribute majority of spend:
- surface separately from human users

### 10.7 Volatility rule
If identity has erratic monthly pattern:
- tag as `volatile spender`

### 10.8 Dormant-to-spike rule
If user historically low-cost suddenly jumps materially:
- tag as `new anomaly`

### 10.9 Governance action rule
If a query or identity is both high-cost and repeatedly inefficient:
- attach a clear remediation suggestion
- estimate avoidable spend
- elevate to action queue

---

## 11. Claude Code Orchestration Role

Claude Code can be used as the orchestration and transformation layer for insight generation, semantic labeling, and dashboard-ready output preparation.

### 11.1 What Claude Code should do
- generate scheduled transformation scripts
- normalize identity naming
- group similar queries using query hashing / normalization
- tag likely waste patterns
- generate concise executive insight text
- build dashboard-ready summary tables
- create anomaly narratives
- rank actions by likely savings impact

### 11.2 What Claude Code should not do
- invent unsupported explanations
- present uncertain attributions as fact
- overwhelm dashboards with long text
- replace core SQL metrics logic

### 11.3 Best operating pattern
Use deterministic SQL first. Use Claude Code second.

Suggested split:
- **SQL / dbt / BigQuery**: aggregation, joins, thresholds, anomaly metrics, cost allocation
- **Claude Code**: summarization, human-readable insight phrasing, tagging support, orchestration, doc generation

---

## 12. Suggested Dashboard Output Tables for Claude Code

Claude Code orchestration should produce clean output tables/views such as:

### `dashboard_exec_monthly_summary`
- month
- total_cost
- mom_change_pct
- top_driver_name
- top_driver_cost
- top_spike_driver_name
- top_spike_driver_cost
- service_account_share_pct
- potential_savings_estimate
- governance_alerts_count
- summary_line_1
- summary_line_2
- summary_line_3

### `dashboard_identity_rankings`
- month
- identity
- owner_name
- team_name
- total_cost
- cost_share_pct
- mom_change_cost
- mom_change_pct
- costly_query_count
- avoidable_cost_estimate
- anomaly_tag
- action_priority

### `dashboard_query_culprits`
- month
- normalized_query_hash
- query_preview
- primary_identity
- primary_team
- total_cost
- avg_cost
- execution_count
- waste_pattern_tag
- primary_dataset
- recommendation
- optimization.headline (one-line fix summary)
- optimization.bullets (list of 4–6 specific SQL best-practice steps)
- optimization.sql_example (copy-pasteable BigQuery SQL)
- optimization.estimated_savings (USD estimate)
- optimization.savings_pct (% of current cost recoverable)
- optimization.priority (Critical / High / Medium)
- optimization.effort (Low / Medium / High)

### `dashboard_monthly_spike_attribution`
- month
- identity
- incremental_cost
- spike_contribution_pct
- spike_type
- narrative_label

### `dashboard_governance_actions`
- month
- identity
- issue_type
- severity
- estimated_cost_impact
- recommended_action
- owner_name
- status

---

## 13. CEO-ready Insight Examples

These are the kinds of sentences the final dashboard should produce.

- `BigQuery spend rose 18% this month, with 54% of the increase driven by 3 identities.`
- `The largest contributor was the customer analytics workload owned by Team X.`
- `One repeated query pattern ran 186 times and accounted for ₹2.1L in estimated cost.`
- `Service accounts now drive most of the spend, suggesting optimization focus should shift from ad hoc users to pipeline workloads.`
- `Potential savings are concentrated in repeated large-scan queries rather than broad user behavior.`
- `42% of total monthly spend came from 3 IAM identities.`
- `One service account executed repeated scan-heavy queries causing a major cost spike.`
- `Approximately $X of spend appears reducible through query optimization and table partitioning.`
- `Several high-cost workloads currently lack ownership mapping.`

This is the tone to maintain: concise, factual, calm.

---

## 14. Recommended Drill Actions

For each flagged user or query, provide one clear next step.

Examples:
- Review partition filters on top 3 scanned tables
- Check whether dashboard refresh frequency is excessive
- Convert repeated ad hoc analysis into scheduled aggregate table
- Validate whether service account workload should use materialized summary tables
- Review query text for `SELECT *` and unbounded date scanning
- Add owner mapping for unidentified service accounts
- Limit direct raw-table access where repeated misuse is detected
- Retire dormant scheduled jobs still generating spend

The goal is not just blame. The goal is action.

---

## 15. Success Criteria

The dashboard is successful if:

### Executive success
- CEO can explain monthly spike in under 2 minutes
- Finance can identify top cost owners without SQL help
- CTO can see whether spike is systemic or concentrated

### Governance success
- Data platform team can identify top 10 costly IAM identities quickly
- repeated expensive query patterns are visible without manual log review
- service accounts and human users are clearly separated
- governance actions can be prioritized by savings impact
- avoidable spend is visible and trackable

### UX success
- dashboard feels premium and uncluttered
- insights are understandable without technical interpretation
- pills and rounded layout reduce stiffness and improve scanability

---

## 16. Risks and Guardrails

### Risk: misleading precision
Per-user cost may be estimated, not exact.

**Guardrail:** clearly label cost allocation basis.

### Risk: ugly IAM naming
Service accounts may make the dashboard unreadable.

**Guardrail:** maintain identity mapping table.

### Risk: noisy executive page
Too many visuals will kill adoption.

**Guardrail:** keep Page 1 to a briefing format only.

### Risk: AI-generated fluff
Insight text may become verbose and low-trust.

**Guardrail:** enforce short structured templates.

### Risk: query text overload
Raw SQL can overwhelm leadership.

**Guardrail:** show summarized previews and drill only when needed.

---

## 17. Suggested Build Phases

### Phase 1: Data foundation
- validate billing export fields
- join jobs metadata
- build identity map
- create job-level cost allocation model

### Phase 2: Insight layer
- monthly user rollups
- query culprit rollups
- anomaly and spike rules
- waste pattern tagging
- governance signal tagging
- avoidable spend estimation

### Phase 3: Dashboard UX
- executive overview page
- IAM drilldown page
- query culprit explorer
- pill-based filters and narrative cards
- governance actions panel
- savings tracker

### Phase 4: Claude Code orchestration
- summary text generation
- alert message generation
- recurring insight refresh workflow
- dashboard metadata documentation

---

## 18. Implementation Decisions (Confirmed)

The following decisions are confirmed for this implementation.

### BI Rendering Layer
The dashboard will be built using **Claude Artifact–generated UI components**, meaning layout flexibility is high and design can fully adopt the pill-based modern layout described earlier.

### Data Source
A **single raw query output** will be provided which already joins billing data and query/job metadata. This raw dataset becomes the primary input table for the insight layer.

Claude Code must follow a **download-first working pattern**:

1. execute or receive the raw extraction query once
2. download/export the result to a local working file such as CSV or parquet
3. perform profiling, transformation, aggregation, tagging, and narrative generation from the downloaded local dataset
4. avoid repeated BigQuery reads unless an explicit refresh is requested

This is important for three reasons:
- avoids unnecessary BigQuery cost from repeated development queries
- gives stable reproducible input while iterating dashboard logic
- reduces latency while Claude builds insight tables and UI artifacts

Recommended local workflow:
- raw extract file: `bq_billing_raw_monthly.parquet`
- curated working files: monthly summary and culprit tables derived locally
- only re-query BigQuery when source refresh is intentionally triggered

Claude Code will orchestrate transformations from this raw dataset into curated insight tables.

### Cost Attribution Scope
Cost will be attributed **only at IAM identity level**.

Dataset, table, and project attribution may exist in raw data but are not required for the core executive dashboard.

### Identity Scope
Focus is on **IAM users only**.

Service accounts may exist in the dataset but the primary analysis dimension is the IAM user responsible for queries.

### Time Grain
The dashboard will focus on **monthly analysis**.

Supporting daily calculations may exist internally but the primary executive view is:

- monthly cost
- month-over-month change
- monthly spike drivers

### Currency
All financial metrics will be presented in **USD**.

### Identity Mapping
A **user mapping table already exists** mapping IAM users to owners / teams.

This mapping will be used to convert raw IAM emails into readable names for leadership views.

### Narrative Insight Generation
Claude Code will generate **short narrative insight text** used in the executive dashboard.

Narratives must follow strict rules:

- maximum 1–2 sentences per insight
- factual and data-driven
- no speculation
- no marketing language

Example output style:

`Three IAM users contributed 52% of this month’s BigQuery spend.`

`One repeated analytics query pattern ran 214 times and accounted for $8,420 in estimated cost.`

Claude must never generate long paragraphs.

### Artifact Output Tables
Claude Code should orchestrate generation of these tables:

- `monthly_cost_summary`
- `monthly_identity_rank`
- `monthly_spike_drivers`
- `expensive_query_patterns` (includes `optimization` block per query)
- `identity_cost_trend`
- `governance_action_queue`
- `savings_tracker`

### Query Optimization Engine
Script: `skills/generate_query_optimizations.py`

Runs after `generate_insights.py`. Reads `top_queries` from `executive_summary.json`, applies a pattern rule engine, and writes back optimization recommendations inline.

Pipeline order:
1. extract_bq_data.py
2. transform_cost_data.py
3. generate_insights.py
4. **generate_query_optimizations.py** ← new step
5. upload_to_gcs.py
6. build_dashboard_artifact.py

These will feed the Artifact dashboard components.

### Required Orchestration Rule
Before any analysis or UI generation begins, Claude Code should first materialize the source data locally and treat that download as the working snapshot for the run.

Recommended run sequence:

1. Run raw extraction query once
2. Download result locally
3. Validate schema and row counts
4. Build transformed monthly tables locally
5. Generate insight text from transformed tables
6. Render Artifact dashboard from those curated outputs

This rule should be treated as mandatory unless the user explicitly asks for a live refresh.

---

## 19. Claude Execution Environment and Folder Structure

To ensure deterministic operation and avoid repeated BigQuery queries, Claude Code should operate within a structured local project layout. This allows the system to download the raw dataset once and perform all subsequent analysis locally.

### Recommended Project Structure

```text
bq_cost_dash/

connection/
  bigquery_connection.md
  service_account.json

ref/
  raw_query.sql
  iam_user_mapping.csv
  cost_rules.md

 data/
  bq_billing_raw.parquet
  monthly_identity_cost.parquet
  monthly_spike_drivers.parquet
  expensive_queries.parquet
  identity_cost_trend.parquet

skills/
  transform_cost_data.py
  generate_insights.py
  build_dashboard_artifact.py

output/
  executive_summary.json
  dashboard_dataset.json
```

### Folder Responsibilities

**connection/**
Contains connection configuration and credentials used only for the initial data extraction.

**ref/**
Contains reference materials required for transformation and analysis:
- raw extraction SQL query
- IAM user mapping
- cost classification rules

**data/**
Local working data storage. Claude must first download the raw dataset here and perform all transformations using this local snapshot.

**skills/**
Reusable transformation and insight generation modules. These scripts must operate only on local datasets and must not query BigQuery.

**output/**
Final datasets and summaries used by the dashboard artifact layer.

---

## 20. Claude Execution Workflow

Claude must follow this deterministic execution sequence.

### Step 1 — Extract raw data

Run the extraction SQL located in:

`ref/raw_query.sql`

The result should be exported and saved locally as:

`data/bq_billing_raw.parquet`

This step must execute only once per refresh cycle.

### Step 2 — Local transformations

Load the raw dataset locally and generate derived insight tables:

- `data/monthly_identity_cost.parquet`
- `data/monthly_spike_drivers.parquet`
- `data/expensive_queries.parquet`
- `data/identity_cost_trend.parquet`

These transformations include aggregation, ranking, spike detection, and query grouping.

### Step 3 — Insight generation

Claude should read the transformed datasets and produce a concise executive narrative file:

`output/executive_summary.json`

Example structure:

```json
{
 "month":"2026-03",
 "total_cost":12400,
 "mom_change":18,
 "top_driver":"analytics@company.com",
 "top_driver_cost":4200,
 "spike_driver":"marketing@company.com",
 "insights":[
   "Three IAM users contributed 54% of monthly BigQuery cost.",
   "One repeated analytics query pattern ran 186 times costing $2,100.",
   "Cost spike primarily came from marketing analytics workloads."
 ]
}
```

### Step 4 — Artifact dashboard generation

Claude should construct the final dashboard UI using curated outputs stored in:

`output/dashboard_dataset.json`

The UI must follow the pill-based design language defined earlier in this document.

---

## 21. Mandatory BigQuery Query Rule

Claude must never repeatedly query BigQuery during analysis or dashboard generation.

The rule is:

- Run the extraction query once
- Save the dataset locally
- Perform all analysis from the downloaded dataset

Repeated querying is only allowed when the user explicitly requests a refresh.

---

## 22. Additional Derived Metrics

To strengthen leadership insights, the following metrics must be computed during transformation.

### Cost per Query

```text
cost_per_query = total_cost / query_count
```

This helps detect inefficient workloads that produce high cost per execution.

### Cost Concentration

```text
cost_concentration = top_5_users_cost / total_cost
```

This indicates whether spend is driven by a few users or broadly distributed across the organization.

### Cost per TB Scanned per IAM

```text
cost_per_tb_scanned = total_cost / (bytes_billed / 1099511627776)
```

This is a high-value governance metric because it quickly exposes wasteful query behavior, poor filtering, and repeated scan-heavy workloads.

### Avoidable Spend Estimate

Use rule-based estimation on workloads tagged as:
- repeated full scans
- unbounded date scanning
- repeated expensive ad hoc queries
- non-partitioned table scans

This should be surfaced as a directional estimate, not an exact financial promise.

---

## 23. Governance KPIs and Savings Tracker

The executive dashboard should include a governance KPI layer, not just billing KPIs.

Recommended KPIs:
- Total monthly cost
- Month-over-month change
- Top IAM contributor
- Largest spike driver
- Governance alerts count
- Estimated avoidable spend
- Service account spend share
- Cost concentration ratio

### Savings Impact Tracker

Track:
- cost before governance action
- cost after remediation
- monthly savings realized
- cumulative savings
- open governance actions
- closed governance actions

Narrative examples:
- `Partitioning three high-scan tables reduced monthly BigQuery cost by 14%.`
- `Retiring two repeated dashboard scan patterns saved an estimated $1,240 this month.`

---

## 24. Claude Skill Definition (6-Step Skill Building Framework)

This project should follow the **6-Step Skill Building Framework** so Claude Code can reliably execute the workflow without ambiguity.

### 1. Name + Trigger

**Skill Name:** BigQuery Cost Intelligence Builder

**Trigger examples:**
- "Build BigQuery cost dashboard"
- "Analyze BigQuery billing data"
- "Generate IAM cost insights"
- "Create BigQuery FinOps dashboard"

This skill activates whenever cost analysis or dashboard generation is requested for BigQuery billing datasets.

---

### 2. Goal

Generate a CEO-ready BigQuery cost intelligence dashboard that clearly identifies:

- which IAM users drive cost
- which queries are expensive
- what caused monthly cost spikes
- where optimization actions should be prioritized

All insights must be derived from a **single downloaded dataset snapshot** to prevent repeated BigQuery queries.

---

### 3. Step-by-Step Process

Claude must follow this exact workflow.

#### Step 1 — Run extraction query
Execute:

`ref/raw_query.sql`

Export results locally to:

`data/bq_billing_raw.parquet`

This step runs **only once per refresh**.

#### Step 2 — Validate dataset

- confirm schema
- verify row count
- check required fields exist

Required columns typically include:

- job_date
- user_email
- query_text
- bytes_billed
- estimated_cost

#### Step 3 — Build monthly IAM aggregates

Generate:

`data/monthly_identity_cost.parquet`

Metrics:

- total_cost
- query_count
- cost_per_query
- bytes_billed
- cost_per_tb_scanned

#### Step 4 — Detect spike drivers

Generate:

`data/monthly_spike_drivers.parquet`

Identify identities with largest month-over-month cost increases.

#### Step 5 — Detect expensive query patterns

Generate:

`data/expensive_queries.parquet`

Group similar queries using normalized query hashing.

Metrics include:

- execution_count
- total_cost
- avg_cost

#### Step 5b — Generate query optimizations

Run `skills/generate_query_optimizations.py`.

For each expensive query, a rule engine detects SQL anti-patterns and attaches a full optimization record:

- `optimization.headline` — one-line summary of the fix
- `optimization.bullets` — 4–6 bulleted, specific, actionable steps using SQL best practices
- `optimization.sql_example` — copy-pasteable BigQuery SQL showing the before/after fix
- `optimization.estimated_savings` — dollar estimate based on pattern severity
- `optimization.savings_pct` — % of current cost recoverable
- `optimization.priority` — Critical / High / Medium
- `optimization.effort` — Low / Medium / High

**Pattern detection rules and corresponding fixes:**

| Pattern | Detection | Recommendation |
|---|---|---|
| Extreme polling loop (≥10K runs) + queue scan | `WHERE executed=0 ORDER BY ... LIMIT` | Replace with Pub/Sub or Cloud Tasks; poll a micro pending_jobs table |
| Extreme polling loop + COUNT(*) | `SELECT COUNT(*)` called 8K+ times | Replace with INFORMATION_SCHEMA.TABLE_STORAGE (zero scan) |
| Extreme polling loop + SELECT DISTINCT | DISTINCT called 8K+ times | Materialize to scheduled summary table refreshed hourly |
| Extreme polling loop + UPDATE | UPDATE run 50K+ times | Batch all updates into one MERGE using UNNEST(@id_array) |
| High-frequency DELETE + IN subquery | DELETE WHERE IN (SELECT) 500+ times | Replace with single MERGE upsert; add partition filter on both sides |
| High-frequency full table overwrite on staging | CREATE OR REPLACE staging 500+ times | Partition staging table by date; INSERT only today's partition |
| High-frequency UPDATE loop | UPDATE 300+ times individually | Buffer updates; run scheduled MERGE per interval |
| High-frequency CTE | WITH ... AS ... 370+ times | Materialize CTE output as daily aggregate table via Scheduled Query |
| Repeated full table overwrite | CREATE TABLE IF NOT EXISTS 100+ times | Add PARTITION BY + CLUSTER BY; switch to incremental append |
| Repeated INSERT + large scan | INSERT scanning 500+ GB per run | Partition source + target; filter WHERE date = CURRENT_DATE() |
| Repeated dedup with ROW_NUMBER | ROW_NUMBER full table 50+ times | Scope dedup to latest partition only using _PARTITIONDATE filter |
| BI tool query | Looker/Tableau column alias pattern | Enable BI Engine reservation; add materialized view; add clustering |
| SELECT * + 2-year date range | SELECT * + INTERVAL 2 YEAR | Explicit column list; narrow date window; partition source table |
| CTE heavy + large scan | WITH clause + avg >100 GB | Materialize CTE to scheduled table; add run_date partition column |
| Generic large scan | avg >50 GB, no specific pattern | Add DATE partition + CLUSTER BY; replace SELECT * with column list |

**Dashboard rendering (Query Explorer tab):**
- Each query card shows an inline optimization box below the query preview
- Box contains: headline (bold), bulleted list of specific actions, expandable SQL example block (collapsible `<details>`), priority/effort/savings pills
- Modal (View SQL) shows full query text + full optimization detail including SQL example
- Savings banner at top of page shows total estimated savings across all queries
- Team column removed from IAM identities table (replaced by Type pill only)

Output is written into `output/executive_summary.json` under each query's `optimization` key.
Total potential savings stored as `total_potential_savings` at root level.

#### Step 6 — Generate insights

Produce:

`output/executive_summary.json`

Example:

```json
{
 "month":"2026-03",
 "total_cost":12400,
 "mom_change":18,
 "top_driver":"analytics@company.com",
 "insights":[
  "Three IAM users contributed 54% of monthly cost.",
  "One query pattern accounted for $2,100 in spend.",
  "Cost spike mainly came from analytics workloads."
 ]
}
```

#### Step 7 — Build Artifact dashboard

Render the UI using pill-based layout components with rounded sections and minimal clutter.

---

### 4. Reference Files

Claude must load these reference files before execution.

Location: `ref/`

Files include:

- `raw_query.sql` → BigQuery extraction query
- `iam_user_mapping.csv` → IAM → owner/team mapping
- `cost_rules.md` → cost classification rules

These references help convert technical metadata into leadership-readable insights.

---

### 5. Rules (Guardrails)

Claude must follow these constraints.

**Rule 1 — Query BigQuery only once**

Run the extraction query once and store the dataset locally.

All further processing must use the downloaded dataset.

**Rule 2 — Avoid AI verbosity**

Insights must be:

- factual
- concise
- maximum two sentences

**Rule 3 — CEO readability**

Avoid technical jargon like:

- slot utilization
- scan operators

Instead use phrasing such as:

"Large table scans" or "Repeated analytics queries".

**Rule 4 — No speculative claims**

Insights must always be derived from measurable metrics.

**Rule 5 — Prefer ranking over raw tables**

Dashboards should prioritize:

- top cost drivers
- spike contributors
- expensive queries

---

### 6. Self-Improvement

Claude should continuously improve the skill by storing validated outputs.

Examples of improvement artifacts:

- previously approved executive summaries
- improved cost classification rules
- common expensive query patterns

These can be stored in:

`ref/insight_patterns.md`

Over time this allows Claude to better detect:

- repeated cost culprits
- recurring spike sources
- common waste patterns

---

## 25. Final Positioning Statement

This dashboard should not look like a cloud billing report.

It should look like a leadership control surface for BigQuery spend, where within seconds anyone can see:
- what the business spent
- who drove it
- which queries caused it
- where action is needed first
- what savings can be realized

That is the standard for this build.

---

## 26. GitHub Pages Deployment

### 26.1 Repository

Dashboard is hosted as a static site on GitHub Pages:

- **Repo:** https://github.com/abhijeet2021/bq_cost
- **Live URL:** https://abhijeet2021.github.io/bq_cost/
- **Branch:** `main` / root (`index.html`)

The dashboard is a fully self-contained HTML file — no backend, no server, no external API calls. All data is embedded directly inside the HTML at build time.

### 26.2 Initial Setup (one-time)

```bash
cd "C:/Users/trade/OneDrive/Desktop/bq_cost_dash/output"
git init
cp bigquery_cost_dash.html index.html
git add bigquery_cost_dash.html index.html
git commit -m "Deploy BQ cost dashboard"
git branch -M main
git remote add origin https://github.com/abhijeet2021/bq_cost.git
git push -u origin main
```

Then in GitHub: **Settings → Pages → Source → Deploy from branch → main / root → Save**

### 26.3 Refresh / Update Workflow

Run after each data refresh cycle to push the latest dashboard live:

```bash
# Step 1 — Rebuild the dashboard (skip BQ extraction if raw data unchanged)
cd "C:/Users/trade/OneDrive/Desktop/bq_cost_dash"
python skills/run_pipeline.py --skip-bq

# Step 2 — Copy artifact to index.html and push
cd output
cp bigquery_cost_dash.html index.html
git add index.html bigquery_cost_dash.html
git commit -m "Dashboard refresh $(date +%Y-%m-%d)"
git push
```

To force a fresh BigQuery pull (e.g. monthly):
```bash
python skills/run_pipeline.py   # no --skip-bq flag
```

The page goes live ~1 minute after push.

### 26.4 Pipeline Files Reference

| File | Purpose |
|---|---|
| `skills/extract_bq_data.py` | Queries BigQuery INFORMATION_SCHEMA.JOBS_BY_PROJECT, saves `data/bq_billing_raw.parquet` |
| `skills/transform_cost_data.py` | Transforms raw parquet into monthly/identity/query parquets |
| `skills/generate_insights.py` | Produces `output/executive_summary.json` |
| `skills/generate_query_optimizations.py` | Injects per-query optimization blocks into executive_summary.json |
| `skills/build_dashboard_artifact.py` | Embeds JSON into HTML template, outputs `output/bigquery_cost_dash.html` |
| `skills/upload_to_gcs.py` | Uploads dataset to GCS bucket `bq-cost-dash-radixbi` (optional) |
| `skills/run_pipeline.py` | Master runner — orchestrates all steps above in order |
| `output/bigquery_cost_dash.html` | Final self-contained dashboard artifact (212 KB) |
| `output/index.html` | Copy of above, served by GitHub Pages |
| `output/executive_summary.json` | Intermediate data file — embedded into HTML at build time |
| `output/bigquery_cost_dash_template.html` | Base HTML template (do not edit manually) |

### 26.5 Technical Notes

- **No external fetch**: Claude Artifact sandbox and GitHub Pages both serve static files. All data is embedded at build time — no runtime API calls.
- **`</script>` injection fix**: JSON embedded in `<script>` blocks has all `</script>` occurrences escaped to `<\/script>` to prevent HTML parser breakage.
- **Service account**: `prod-prj-n8n@radixbi-249015.iam.gserviceaccount.com` — requires `BigQuery Resource Viewer` + `Storage Admin` roles.
- **GCS bucket**: `bq-cost-dash-radixbi` — used for optional cloud copy of dashboard_dataset.json (not required for GitHub Pages).

---

## 27. Dashboard Design Decisions (Final)

### 27.1 IAM Table — Team Column Removed

The **Team** column has been removed from the IAM Identities table (Page 2). Identity type is conveyed via a pill (Service Account / User). This keeps the table clean and avoids unmaintained team mappings.

### 27.2 Full Query Text in Modal

The **View SQL** modal shows the complete representative query text — not a truncated preview. The `executive_summary.json` must include the `representative_query` field (full text) from `expensive_queries.parquet`, not just `query_preview` (120 chars). The `generate_insights.py` script must copy `representative_query` for each query into the JSON output.

### 27.3 Optimization Recommendations — Depth Standard

Each expensive query must receive a deep, actionable optimization block with:

- **Headline** — one-line fix statement
- **Bullets** — 4–6 specific steps using BigQuery best practices (partitioning, clustering, MERGE, materialized views, BI Engine, Pub/Sub, INFORMATION_SCHEMA micro-tables)
- **SQL example** — copy-paste ready BigQuery SQL inside a collapsible `<details>` block
- **Pills** — Priority (Critical/High/Medium), Effort (Low/Medium/High), Estimated savings ($)

Total identified savings as of last run: **$1,397.08** across 20 queries.
Stored as `total_potential_savings` in `executive_summary.json`.

### 27.4 Savings Banner

Query Explorer page shows a banner at the top with total potential savings across all detected query patterns.
