# Foundry Build Spec - AML Investigation Portal

## What we are building in Foundry

### The product surfaces (3 apps)
- Alert Triage Workspace (L1/L2): prioritized queue, explainable risk score, disposition + override rationale.
- Case Investigation Workspace (L2/L3): transactions + evidence, behavioral metrics, network view, case package.
- AI Infra Admin Dashboard (Platform/FinOps/Compliance): cost, latency, model registry, governance status, audit export.

### The data foundation (4 layers)
- Raw (Immutable): source-aligned transaction + KYC + sanctions + adverse media + SAR history.
- Curated (Normalized): unified transaction model + customer 360 + counterparty registry.
- Feature Store (ML-ready): behavioral windows + typology features + network features.
- Serving: alerts, cases, SAR drafts, audit logs, infra metrics.

## 2) Existing workflow mapping (how AI fits into current processes)

### Current-state steps (typical AML ops)
Transaction monitoring generates alerts →
L1 triage →
L2 investigation (evidence, pattern analysis) →
RFI →
L3 escalation / SAR narrative →
BSA Officer approval →
Filing + audit/exam support

### AI insertion points (target-state)
- Risk detection: typology flags + risk score + auto-escalation.
- Decision support: executive context summary + recommended outcome + confidence.
- Workflow enforcement: mandatory override rationale, SLA timers, audit logging.
- SAR drafting acceleration: pre-populated narrative + evidence compilation.
- Infra governance: model registry + validation + cost/latency monitoring.

## 3) Foundry implementation blueprint

### 3.1 Ontology (core objects + links)

Define these as ontology objects so apps can be built fast and consistently:

#### Objects

**Customer**
- Customer Id, segment, risk Rating, KYC dates, expected profile, PEP, sanctions status

**Account**
- Account Id (masked), customer Id, product Type, open Date

**Payment Record (canonical transaction)**
- from your extended schema (transaction Id/external Ref, parties, amount USD, type/channel, geo, risk indicators, rawMessageRef)

**Alert**
- alertId, priority (P1-P4), aiRiskScore (0-100), typologyTags, evidenceFlags[], recommendation, SLA, executiveContext

**Case**
- caseId, status, assignedTo, triggeringAlert, VaR, transactionSummary, behavioralMetrics, networkAnalysis, analystDecision

**RFI**
- rfiId, caseId, status, dueDate, questions, response metadata

**SAR**
- SAR Id, case Id, status, version history, narrative, institution info, subject info, model attribution

**Model**
- Model Id, owner, version, hosting env, metrics (precision/recall/F1/AUC), validation dates

**Inference Event**
- per-model inference telemetry: latency Ms, tokens (if applicable), cost estimate, CPU/GPU, outcome, confidence

**Audit Log**
- immutable event log: timestamp, eventType, userId/role, entityId/type, action, details, metadata

#### Key relationships
- Customer has Accounts
- Customer/Account has Payment Records
- Payment Records trigger Alerts
- Alert opens Case
- Case produces RFI and/or SAR
- Models generate Inference Events contributing to Alert/Case/SAR
- All actions emit Audit Log events (linked to entity Id)

### 3.2 Datasets (Foundry tables to build)

#### Raw layer (immutable)
- raw_cbs_transactions
- raw_swift_messages
- raw_ach_files
- raw_card_transactions
- raw_kyc_profiles
- raw_sanctions_results
- raw_negative_news_hits
- raw_historical_sars (synthetic for exercise)

#### Curated layer (normalized)
- curated_payment_record (unified canonical PaymentRecord)
- curated_customer_360
- curated_counterparty_registry
- curated_geo_risk_reference (FATF risk tiers, sanctions countries — synthetic)
- curated_typology_rules (flag definitions + thresholds)

#### Feature store
- fs_behavioral_features_30d
- fs_behavioral_features_90d
- fs_velocity_features
- fs_structuring_features
- fs_geo_features
- fs_network_features (graph edges + derived scores)
- fs_labels (synthetic "SAR filed" / "closed-no-sar")

#### Serving layer
- serving_alerts
- serving_cases
- serving_sars
- serving_rfis
- serving_audit_logs
- serving_infra_metrics
- serving_model_registry

## 4) Synthetic data plan (what the data team should generate)

### 4.1 Guiding rules
- Must look realistic, but contain no real customer data.
- Must support the portal flows end-to-end: triage → case → RFI → SAR → audit → infra admin.
- Must include both positives and negatives so model accuracy + drift monitoring can be demonstrated.

### 4.2 Minimum viable synthetic volumes (exercise-ready)
- Customers: 5,000 (Retail 70%, SME 25%, Corp 5%)
- Accounts: 8,000
- Transactions: 2–5 million over 6–12 months (enough for rolling windows)
- Alerts: 15,000 (with ~10–20% "true suspicious" label for SAR)
- Cases: 3,000
- SARs: 800 (version history included)
- RFIs: 1,200
- Audit logs: 500k+ events (system + user actions)

### 4.3 Synthetic typology scenarios to bake in

Create "scenario packs" so models and UI have deterministic examples:

- **Structuring pack**: repeated cash deposits $8.5k–$9.99k, multi-branch, clustering windows
- **Velocity pack**: 3–5x baseline frequency spikes, round-amount patterns
- **Geo risk pack**: new high-risk corridor, inconsistent with operating geographies
- **Network pack**: shared counterparties with historical SAR nodes, shared identifiers (address/phone)
- **TBML pack**: invoice value anomalies (even if simplified)

Each scenario should generate:
- transactions + counterparties
- expected feature patterns
- expected flags (FLAG-STR-xxx etc.)
- expected alert priority outcomes

## 5) Model & rules build plan (what to implement for the exercise)

### 5.1 Models (aligned to your inventory)

For Foundry exercise, implement as two-stage scoring:

**Stage A — Deterministic rules (fast, CPU-first)**
- Structuring rules (threshold proximity, frequency, multi-branch)
- Velocity rules (baseline deviation multipliers, peer percentile)
- Geo rules (risk tier, novelty, inconsistency vs KYC)
- Composite priority mapping (your matrix)
- Output: rule_flags[], rule_risk_score, rule_priority

**Stage B — ML scoring (optional / simplified)**
- Baseline: Gradient Boosted Trees / Logistic Regression on engineered features
- Optional: Graph score for network risk (start with heuristic graph metrics; reserve GPU for later)
- Output: ml_risk_score, feature_attribution (top drivers)

**Final scoring**
- aiRiskScore = blend(rule, ml) with weights configurable
- produce:
  - evidenceFlags[]
  - systemRecommendation { outcome, rationale, modelContributions[] }
  - executiveContext (templated summary for exercise; LLM optional)

### 5.2 Accuracy framework (what to calculate)

Compute and store per model + segment:
- Precision, Recall, F1, AUC
- FPR/FNR
- Drift checks (weekly feature distribution deltas)
- Bias monitoring (simple parity checks across synthetic demographics)
- Validation schedule fields (last/next due)

Persist into: serving_model_registry + serving_infra_metrics

## 6) App requirements → Foundry app mapping (what to build where)

### 6.1 Alert Triage Workspace (L1/L2)

**Core views**
- Prioritized queue (default sort by risk desc)
- Filters: typology tags, priority, SLA risk, segment
- Alert detail:
  - executiveContext
  - recommendation (Escalate/Close/RFI)
  - evidenceFlags (confidence + data source)
  - decision control: Accept/Override (override rationale required)

**Foundry dependencies**
- Reads: serving_alerts, curated_payment_record, curated_customer_360
- Writes: serving_audit_logs, serving_cases (on "Open Case")

### 6.2 Case Investigation Workspace (L2/L3)

**Core views**
- Transaction table with highlight
- Behavioral deviation panel (rolling windows vs baseline)
- Network visualization (nodes/edges + risk badges + SAR links)
- Value-at-risk summary (amount, window, deviation%, peer percentile)

**Dependencies**
- Reads: serving_cases, curated_payment_record, fs_*, raw_historical_sars
- Writes: serving_audit_logs, serving_rfis, serving_sars (draft)

### 6.3 RFI Management
- RFI create from case (pre-populated)
- Status tracking: Draft/Sent/Received/Overdue
- Manager dashboard: response rate, overdue counts

### 6.4 SAR Drafting & Filing (exercise = drafting + approvals)
- Generate SAR draft package (templated narrative)
- Workflow: Draft → Review → Approved → Filed (role-based)
- Version history + diff (can be simplified to "version list" in exercise)

### 6.5 AI Infra Admin Dashboard

**Core widgets**
- Cost MTD, latency P95 vs SLA, volume, system health
- Filters: workload, time granularity, environment, cost metric
- Top cost drivers (expandable root cause)
- Model registry table (version, latency, error rate, hosting env, health)
- Governance footer: pending validation, cost thresholds, auto-throttling, audit logging

**Dependencies**
- Reads: serving_infra_metrics, serving_model_registry, serving_inference_events
- Writes: serving_audit_logs (admin actions), governance_events (optional)

## 7) Zero-copy data architecture strategy (Foundry + Rackspace)

Principle: keep data where it lives; provide governed access via logical layers.

In exercise: data is generated in Foundry, but structure it as if sources were remote:
- raw tables represent "source-aligned" ingestion (CBS/SWIFT/ACH/Card/KYC).

Zero-copy posture:
- Avoid proliferating duplicate datasets; use curated/feature/serving as logical products with lineage.
- Enforce RBAC at dataset + ontology object level (e.g., masked account numbers, restricted SAR narratives).

## 8) Infrastructure design (CPU-first, VMware, Dell, GPU-optional)

### CPU-first default (what runs on CPU)
- Rules engine + feature generation
- ML baseline scoring (LR/GBTs)
- Most dashboard queries + serving APIs
- Audit log enrichment

### GPU only when justified
- Network GNN at scale (if you truly implement GNN inference)
- Optional LLM drafting at high volume (otherwise keep templated narrative for exercise)

### VMware + Dell alignment (implementation intent)
Host data + pipelines + inference runtime in Rackspace Private Cloud / VMware Foundation

Dell compute sizing guidance:
- CPU nodes sized for steady-state inference + feature generation
- GPU pool isolated for network model workloads only (if enabled)

## 9) Model hosting & inference visibility framework (mandatory)

### What to log per inference (store in serving_inference_events)
- requestId, timestamp, modelId/version
- input entity IDs (customerId, transactionIds)
- output score, decision, confidence
- latencyMs, errorCode
- CPU/GPU utilization snapshot (coarse)
- cost estimate per inference (or per 1k)

### Dashboards
- **SLA**: p50/p95 latency, error rate, throughput
- **FinOps**: cost per 1k, top workloads/models
- **Governance**: validation due, drift alerts, override rates

## 10) Security & compliance considerations (exercise-ready but realistic)

- **RBAC/SSO roles**: L1, L2, L3, Compliance Officer, Auditor, Platform Admin
- **Data minimization**: mask account numbers, restrict SAR narratives
- **Immutable audit**: append-only audit log dataset + export capability
- **Model risk controls**:
  - champion/challenger registry fields
  - validation schedule enforcement + "pending validation" indicators
- **Segmentation**: isolate "SAR" objects and access-controlled views

## 11) Risk-weighted scoring + auto-escalation (implementation spec)

Implement exactly as a configuration table (curated_typology_rules) plus a scoring job.

### Weights
- Structuring 0.7
- Velocity 0.5
- Geography 0.4
- Network 0.6
- TBML 0.3

### Escalation
- 85–100 → L3
- 70–84 → L2
- 50–69 → L1
- <50 → none

Store:
- riskFactorScores{} + weightedScore + priority + autoEscalationLevel

## 12) Champion / Challenger governance (exercise spec)

Create these tables:
- serving_model_registry (champion/challenger flags)
- model_eval_runs (metric snapshots, dataset version, segment breakdown)
- model_promotion_requests (approvals, sign-offs)

### Promotion criteria (from your definition):
- Precision > 75%
- Recall > 85%
- F1 > 78%
- AUC > 0.85
- FPR < 25%
- FNR < 12%

## 13) Pilot-to-production acceleration plan (how to run the exercise like a real delivery)

### Phase 1 — Foundation (0–30 days)
**Deliverables:**
- Ontology + raw/curated/serving datasets wired
- Audit logging operational
- Baseline rules scoring + alert generation

**Exit criteria:**
- End-to-end demo: transactions → alerts → case → audit trail
- Observability dashboards live (latency, volume, errors)

### Phase 2 — Intelligence (30–90 days)
**Deliverables:**
- Feature store + baseline ML scoring
- Segment-level metrics + drift checks
- RFI + SAR draft workflows

**Exit criteria:**
- Champion/challenger table + evaluation runs
- Override analytics + examiner-style audit exports

### Phase 3 — Outcomes (90–180 days)
**Deliverables:**
- Network risk scoring (heuristic → ML → optional GNN)
- Cost optimization + auto-throttling policies
- SLA workload routing (CPU-first, GPU gated)

**Exit criteria:**
- KPI improvement tracking visible
- Production-style governance gates satisfied

## 14) KPIs to instrument (so it's measurable)

### Operational
- Avg time per case, alerts processed per analyst/day, SLA breach rate

### Risk
- Alert→SAR conversion rate, false positive rate, override rate + override "agree/disagree" with AI

### Model
- Precision/recall/F1 by segment; drift detection time

### Infra/FinOps
- Cost per 1k, p95 latency, error rate, CPU utilization %, GPU utilization % (if used)

### Audit
- % decisions logged with rationale; export completeness

## 15) Rackspace differentiated value proposition (how to position this)

- AI embedded into execution (not a standalone model demo)
- Private Cloud + VMware-first delivery pattern
- Zero-copy aligned data strategy (logical products + lineage)
- CPU-first economics with GPU reserved only for high-value workloads
- Full inference observability + examiner-grade auditability
- Governed pilot-to-production path (clear gates, KPIs, validation)

(Directly aligned to the Rackspace AI Execution Framework and Value Proposition.)

## "Starter Pack" for the Foundry team (action list)

1. Build ontology objects + relationships (Customer, PaymentRecord, Alert, Case, SAR, AuditLog, Model, InferenceEvent).
2. Generate synthetic datasets + scenario packs (structuring/velocity/geo/network/TBML).
3. Create pipelines: raw → curated → features → scoring → serving.
4. Implement rules scoring + risk-weighted escalation first (CPU).
5. Add baseline ML scoring + model registry + eval runs.
6. Build 3 apps: Triage, Case, Infra Admin.
7. Wire audit logging everywhere; add exports.
8. Add champion/challenger governance and validation schedule indicators.

---

# README — Foundry Requirements (AML Investigation Portal)

## 0) Purpose

This repository defines the **data products, ontology, pipelines, model interfaces, observability, and app requirements** to implement the AML Investigation Portal as a **Palantir Foundry exercise**.

Focus: **end-to-end workflow realism** (Alert → Case → RFI → SAR → Audit → Infra Admin), **CPU-first**, **governed**, **auditable**, and **production-shaped**.

---

## 1) Scope (What we are building)

### 1.1 Apps (3)
1. **Alert Triage Workspace (L1/L2)**
2. **Case Investigation Workspace (L2/L3)**
3. **AI Infra Admin Dashboard (Platform/FinOps/Compliance)**

### 1.2 Data Layers (4)
- **Raw (Immutable)**
- **Curated (Normalized)**
- **Feature Store (ML-ready)**
- **Serving (App-ready)**

### 1.3 Core Outputs
- `serving_alerts` (prioritized queue + explainability)
- `serving_cases` (investigation packages)
- `serving_rfis` (customer outreach tracking)
- `serving_sars` (draft + approvals + versioning)
- `serving_audit_logs` (immutable audit trail)
- `serving_infra_metrics` + `serving_model_registry` + `serving_inference_events` (observability & governance)

---

## 2) Roles & Personas (RBAC)

Minimum roles:
- **L1 Analyst**: triage/close low-risk, open cases, add notes
- **L2 Analyst**: investigate, override AI with rationale, draft RFIs
- **L3 Analyst**: escalate, SAR drafting, approvals (if delegated)
- **Compliance Officer / BSA Officer**: approve SARs, review overrides
- **Auditor**: search/export audit logs
- **AI Platform Admin**: model registry, infra dashboards, thresholds

RBAC must enforce:
- **Masked account numbers** only in UI and curated/serving layers
- SAR narrative fields restricted to L3+ / Compliance
- Audit logs append-only and exportable

---

## 3) Ontology (Objects & Relationships)

### 3.1 Objects

**Customer**
- `customerId`, `customerType` (Retail/SME/Corp), segment, riskRating, KYC dates, expected profile, PEP, sanctions status

**Account**
- `accountIdMasked`, `customerId`, productType, openDate

**PaymentRecord** (canonical transaction)
- identifiers: `transactionId`, `externalRef`, `batchId`
- parties: originator, beneficiary, intermediary
- details: amount, currency, amountUSD, dates/timestamps, paymentType, channel, narrative/purpose, MCC
- risk indicators: riskScore, flagged, flagReasons, sanctions result, pepIndicator, highRiskGeography
- compliance metadata: ctrRequired/ctrFiled, structuringIndicator, velocityFlag
- source: sourceSystem, rawMessageRef

**Alert**
- `alertId`, generatedAt, priority (P1–P4), aiRiskScore (0–100)
- typologyTags[], evidenceFlags[], recommendation, executiveContext
- customer info, valueAtRisk block, SLA block

**Case**
- `caseId`, status, openedDate, assignedTo
- customer block, triggeringAlert, typology, valueAtRisk
- transactionSummary, behavioralMetrics[], networkAnalysis
- analystDecision {state, aiRecommendation, action, rationale, timestamp}

**RFI**
- `rfiId`, `caseId`, status (Draft/Sent/Received/Overdue), dueDate, questions, response metadata

**SAR**
- `sarId`, `caseId`, status (Draft/Review/Approved/Filed), version history, narrative, subject/institution info, model attribution

**Model**
- `modelId`, owner, version, architecture, hostingEnvironment, validation dates, metrics

**InferenceEvent**
- requestId, timestamp, modelId/version
- entity refs: customerId, transactionIds, alertId/caseId
- output: score, decision, confidence
- telemetry: latencyMs, errorCode, cpuOrGpu, costEstimate, tokens (optional)

**AuditLog**
- logId, timestamp, eventType, userId, userRole
- entityId/entityType, action, details, ipAddress, sessionId, metadata

### 3.2 Relationships
- Customer **has** Account(s)
- Customer/Account **has** PaymentRecord(s)
- PaymentRecord(s) **contribute to** Alert(s)
- Alert **opens** Case
- Case **produces** RFI and/or SAR
- Model **produces** InferenceEvent(s) that **explain** Alerts/Cases/SARs
- Every action **emits** AuditLog linked to entityId

---

## 4) Foundry Data Products (Tables)

### 4.1 Raw (immutable, source-aligned)
- `raw_cbs_transactions`
- `raw_swift_messages`
- `raw_ach_files`
- `raw_card_transactions`
- `raw_kyc_profiles`
- `raw_sanctions_results`
- `raw_negative_news_hits`
- `raw_historical_sars` *(synthetic)*

### 4.2 Curated (normalized)
- `curated_payment_record`
- `curated_customer_360`
- `curated_counterparty_registry`
- `curated_geo_risk_reference` *(synthetic reference)*
- `curated_typology_rules` *(configuration)*

### 4.3 Feature Store (ML-ready)
- `fs_behavioral_features_30d`
- `fs_behavioral_features_60d`
- `fs_behavioral_features_90d`
- `fs_structuring_features`
- `fs_velocity_features`
- `fs_geo_features`
- `fs_network_features` *(edges + derived metrics)*
- `fs_labels` *(synthetic ground truth for eval)*

### 4.4 Serving (app-ready)
- `serving_alerts`
- `serving_cases`
- `serving_rfis`
- `serving_sars`
- `serving_audit_logs` *(append-only)*
- `serving_inference_events`
- `serving_model_registry`
- `serving_infra_metrics`

---

## 5) Synthetic Data Requirements (Exercise)

### 5.1 Volume Targets (minimum viable)
- Customers: **5,000**
- Accounts: **8,000**
- Transactions: **2–5 million** across **6–12 months**
- Alerts: **15,000**
- Cases: **3,000**
- RFIs: **1,200**
- SARs: **800** (with version history)
- Audit logs: **500k+** events
- Inference events: **>= alerts × models** (per-model contributions)

### 5.2 Scenario Packs (must exist)
- **Structuring**: below-threshold deposits, multi-branch, clustering windows
- **Velocity**: 3–5x baseline spikes, round-amount patterns
- **Geo risk**: new high-risk corridors, inconsistent with KYC operating geographies
- **Network**: shared counterparties with "historical SAR" nodes, shared identifiers
- **TBML**: invoice value anomalies (simplified is OK)

Each scenario pack must produce:
- transactions + counterparties + customer profiles
- expected typology flags (FLAG-STR/VEL/GEO/NET/TBM)
- at least one P1 and one non-P1 example
- ground-truth labels for evaluation (`fs_labels`)

---

## 6) Scoring & Alert Generation (Implementation Spec)

### 6.1 Stage A — Rules Engine (CPU-first)

Rules compute:
- typology flags (`flagId`, `flagName`, `confidence`, `details`, `dataSource`)
- risk factor scores: structuring, velocity, geo, network, tbml
- **risk-weighted score** and priority mapping

### 6.2 Risk-Weighted Scoring Matrix

Weights (configuration-driven in `curated_typology_rules`):
- Structuring **0.7**
- Velocity **0.5**
- Geography **0.4**
- Network **0.6**
- TBML **0.3**

Auto-escalation:
- **85–100 → L3**
- **70–84 → L2**
- **50–69 → L1**
- **<50 → none**

Persist into `serving_alerts`:
- `aiRiskScore`, `priority`, `autoEscalationLevel`
- `evidenceFlags[]`, `systemRecommendation`, `executiveContext`, `valueAtRisk`, `sla`

### 6.3 Stage B — ML Score (optional for exercise)

Baseline ML (CPU) using feature store:
- Logistic Regression / Gradient Boosted Trees
- Output: `mlRiskScore`, top drivers (simple feature importance)

Blend:
- `aiRiskScore = w_rule * ruleScore + w_ml * mlScore` (configurable)

### 6.4 System Recommendation Contract

Required in `serving_alerts`:
- `systemRecommendation.outcome`: Escalate / Close / RFI Required
- `systemRecommendation.rationale`: templated (LLM optional)
- `modelContributions[]`: modelId, modelName, contribution %

---

## 7) App Requirements (Acceptance Criteria)

### 7.1 Alert Triage Workspace

Must support:
- prioritized alert queue (default sort risk desc)
- filters: typology tags, segment, priority, SLA at-risk
- alert detail shows: executiveContext, recommendation, evidence flags
- analyst action: accept/override recommendation
- override requires rationale (>= 20 chars)
- "Open Case" creates `serving_cases` record + audit log

### 7.2 Case Investigation Workspace

Must support:
- transaction list with flagged highlights
- behavioral metrics panel (vs baseline)
- network view (nodes/edges, risk levels, historical SAR link)
- value-at-risk summary
- create RFI from case (pre-populated fields)
- generate SAR draft package (pre-populated narrative + evidence)

### 7.3 SAR Workflow
- statuses: Draft → Review → Approved → Filed
- approvals restricted to L3+ / Compliance
- versioning required (at least version list; diff optional)
- every transition emits audit log

### 7.4 Audit Logging & Export
- search/filter by date, eventType, user, entity
- export CSV/PDF for case or date range
- override events must be searchable with rationale text

### 7.5 AI Infra Admin Dashboard

Required widgets:
- inference cost MTD, latency p95 vs SLA, volume, system health
- filters: workload, time grain, environment, cost metric
- top 5 cost drivers with expandable details
- model registry table (version, latency, error, hosting, health)
- governance footer:
  - models pending validation
  - cost thresholds status
  - auto-throttling status
  - audit logging status

---

## 8) Observability (Inference Visibility Framework)

### 8.1 Required telemetry per inference (`serving_inference_events`)
- requestId, timestamp
- modelId/version, hosting env
- entity refs: customerId, transactionIds, alertId/caseId
- outputs: score, decision, confidence
- telemetry: latencyMs, errorCode, cpuOrGpu, costEstimate
- tokens (optional, if LLM used)

### 8.2 Operational SLOs (exercise targets)
- P95 latency < **500ms** (or documented threshold)
- error rate < **1%**
- uptime target: **99.9%** (reported in infra metrics)

---

## 9) Model Governance (Champion/Challenger)

### 9.1 Registry

`serving_model_registry` must track:
- champion vs challenger flag
- owners, hosting env, architecture
- validation dates (last/next due)
- metrics: precision/recall/F1/AUC, FPR/FNR (overall + by segment)

### 9.2 Promotion Criteria (config)
- Precision > 75%
- Recall > 85%
- F1 > 78%
- AUC > 0.85
- FPR < 25%
- FNR < 12%

---

## 10) Security & Compliance (Exercise Guardrails)

- Mask PII/financial identifiers by default in curated/serving
- Strict role-based access for SAR narratives and approvals
- Append-only audit logs, exportable
- Dataset lineage must be traceable from raw → serving

---

## 11) Delivery Plan (Pilot-to-Production Shaped)

**Phase 1 — Foundation**
- ontology + raw/curated/serving
- rules scoring + alert triage + audit logs
- infra dashboard skeleton

**Phase 2 — Intelligence**
- feature store + baseline ML
- RFI + SAR workflows + model registry metrics

**Phase 3 — Outcomes**
- network scoring maturity (heuristic → advanced)
- cost optimization + auto-throttling policies

---

## 12) Definition of Done (Exercise)

A reviewer can:
1. trace an alert back to source transactions and KYC fields
2. see evidence flags + recommendation + risk scoring
3. open a case, draft an RFI, draft a SAR, approve it
4. export a complete audit trail for that case
5. view model registry metrics and inference telemetry in infra admin

---

# README — Business Context & Value (AML Investigation Portal)

## 0) Executive Summary

The AML Investigation Portal is a unified, AI-augmented compliance operations platform that reduces manual investigation effort, improves suspicious activity detection quality, and provides examiner-ready audit transparency.

It combines:
- **Detection & prioritization** (rules + ML risk scoring, typology flags)
- **Human-in-the-loop workflows** (triage → case → RFI → SAR)
- **Governance & auditability** (immutable logs, model validation tracking)
- **Infrastructure observability** (latency, cost, CPU/GPU allocation, health)

---

## A) Industry Context (BFSI pressures)

Financial institutions face:
- escalating alert volumes and staffing constraints
- strict regulatory expectations for BSA/AML compliance
- requirements for explainability, repeatability, and audit evidence
- ongoing model risk management (validation, drift, challenger testing)
- cost pressure to run monitoring and AI efficiently in private environments

---

## B) Customer Pain Points

1. **Manual investigation burden**: evidence gathering is repetitive and time-consuming.
2. **False positives**: too many low-value alerts consume capacity.
3. **Fragmented data**: payments, KYC, sanctions, adverse media, SAR history are siloed.
4. **Weak audit readiness**: hard to reconstruct "why" a decision was made.
5. **Model governance overhead**: validation schedules, drift detection, approvals are inconsistent.
6. **Infra cost opacity**: unclear cost drivers by workload/model; hard to manage SLA and spend.

---

## C) Existing Workflow Mapping (How this integrates into AML Ops)

Current AML operations typically run:
1. Transaction monitoring generates alerts
2. L1 triage
3. L2 investigation and evidence collection
4. RFI (as needed)
5. L3 escalation + SAR narrative drafting
6. Compliance approval
7. Filing and examiner response (audit trail)

Portal integration:
- replaces scattered tools with a **single workflow**
- embeds **AI scoring + explainability** at triage and case levels
- enforces **mandatory override rationale** for audit defensibility
- produces **SAR draft packages** pre-populated with evidence

---

## D) Zero-Copy Data Architecture Strategy (Private + governed)

Principles:
- keep sensitive data within private environments
- minimize duplication by using **logical data products** (raw → curated → features → serving)
- enforce access control at dataset + object level
- preserve lineage for examiner traceability (alert → transactions → features → model outputs)

Why it matters:
- faster compliance reviews
- reduced data sprawl and fewer "shadow copies"
- simpler governance and consistent definitions

---

## E) Infrastructure Design (CPU-first, GPU-optional, VMware + Dell)

Design intent:
- **CPU-first** for rules, features, baseline ML scoring, and most inference workloads
- **GPU only when justified** (e.g., large-scale graph models or heavy LLM usage)
- deploy on **VMware-based private cloud** with Dell-aligned compute sizing patterns:
  - CPU nodes for steady-state scoring + pipelines
  - isolated GPU pool for advanced network workloads (optional)

Operational goals:
- predictable cost per 1k transactions
- SLA control (p95 latency threshold)
- environment isolation (prod/UAT)

---

## F) Model Hosting & Inference Visibility Framework

Non-negotiables for regulated AML:
- per-inference telemetry: modelId/version, decision, confidence, latency, cost estimate, CPU/GPU
- traceability: which models contributed to the alert and by how much
- monitoring: drift, bias checks, segment performance
- governance: validation due dates and "pending validation" indicators

Business impact:
- simplifies regulatory conversations
- prevents "black box" model risk
- enables FinOps cost attribution by workload/model

---

## G) Security & Compliance Considerations

Must-have controls:
- RBAC: L1/L2/L3/Compliance/Auditor/Admin
- default masking of sensitive identifiers
- SAR narratives restricted to privileged roles
- append-only audit logging, exportable for examiners
- model governance aligned to SR 11-7-style practices:
  - challenger testing
  - validation schedules
  - promotion criteria and approvals

---

## H) Pilot-to-Production Acceleration Plan

**Phase 1 — Foundation** (prove end-to-end workflow)
- unified transaction model + alerts + cases + audit log

**Phase 2 — Intelligence** (prove measurable lift)
- feature store + baseline ML scoring
- drift + metrics by segment
- RFI + SAR draft workflows

**Phase 3 — Outcomes** (industrialize + optimize)
- network scoring maturity (heuristic → advanced)
- cost optimization + throttling controls
- operational dashboards as a control plane

---

## I) Rackspace Differentiated Value Proposition

Rackspace delivers:
- private-cloud-first architecture designed for regulated data
- CPU-first economics with GPU reserved for high-value workloads
- production-ready observability for inference, cost, and SLA health
- governance patterns that accelerate pilot-to-production maturity
- a repeatable template for managed service or client-deployed operations

---

## Quantified Business Value (Targets)

Illustrative targets (exercise KPIs):
- investigation time reduction: **40–60%**
- false positives reduced via prioritization: **35–50%**
- alert-to-SAR conversion: increase toward **~18%**
- p95 latency maintained below **500ms**
- audit completeness: **100%** decisions logged with rationale

---

## What Success Looks Like

1. Analysts process fewer alerts but with higher quality
2. SAR drafts are produced faster with better evidence linkage
3. Auditors can reconstruct every decision from immutable logs
4. Model performance and drift are visible by segment
5. Cost per 1k transactions is measurable and reducible without losing coverage
