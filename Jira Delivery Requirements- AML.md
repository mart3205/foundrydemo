# Jira Delivery Requirements - AML Investigation Portal

## EPIC 0 — Jira Setup & Governance

### US-0.1 Create Jira "AI Use Case – BFSI – AML Investigation Portal" issue type + required fields
R: Business Team
Prototype Handoff: Sarelle must see workflow link + business value + success metrics fields populated.

### US-0.2 Configure workflow stages in Jira: Discovery → Prototype → Testing → Production → Monitoring
R: Business Team + Jira Admin (if separate)
Testing clarity impact: gates ensure QA knows entry/exit criteria.

### US-0.3 Establish artifact linking rules (Figma URL, GitHub branch/PR, prompt/config repo, test report)
R: Sarelle (Prototype) + Benjamin (Testing) + Prasana (Production)
Definition: Jira ticket becomes the single index for all artifacts.

## EPIC 1 — Data Foundation (Zero-copy shaped data products in Foundry)

### US-1.1 Build Raw Layer datasets (CBS/SWIFT/ACH/Card/KYC/Sanctions/Adverse Media/SAR History)
R: Data Team
Testing stage: Benjamin validates schema completeness + sample record quality.

### US-1.2 Build Curated Layer: curated payment record + curated_customer_360 + curated counterparty_registry
R: Data Team
Dependencies: raw ingestion complete.

### US-1.3 Create reference/config datasets: curated_geo_risk_reference, curated_typology_rules
R: Data Team
Governance note: rules/weights live here (not in code).

### US-1.4 Create Serving Layer tables: serving_alerts, serving_cases, serving_rfis, serving_sars, serving_audit_logs
R: Data Team
Handoff: enables prototype UI to bind to stable schemas.

## EPIC 2 — Synthetic Data Factory (Scenario packs + labels)

### US-2.1 Generate synthetic customers/accounts with segment mix (Retail/SME/Corp)
R: Data Team

### US-2.2 Generate synthetic transactions at scale (6–12 months; rolling windows supported)
R: Data Team

### US-2.3 Create typology scenario packs (Structuring / Velocity / Geo / Network / TBML)
R: Data Team
Testing relevance: ensures QA has known "truth cases" to validate.

### US-2.4 Create ground-truth label dataset (fs_labels) for model evaluation
R: Data Team
Note: essential for accuracy metrics + champion/challenger comparison.

## EPIC 3 — Detection & Scoring (Rules + optional ML)

### US-3.1 Implement rules engine for typology flags (FLAG-STR/VEC/GEO/NET/TBM)
R: Data Science / ML Team (or Data Team if combined)
Testing: Benjamin validates expected flags for scenario pack records.

### US-3.2 Implement risk-weighted scoring + auto-escalation (85–100 L3, etc.) using config table
R: Data Science / ML Team
Dependency: curated_typology_rules exists.

### US-3.3 Produce serving_alerts output contract (priority, evidenceFlags, recommendation, executiveContext, SLA)
R: Data Science / ML Team
Prototype dependency: Sarelle needs this to render queue + detail.

### US-3.4 (Optional) Implement baseline ML score using feature store (CPU-first) and blend with rules
R: Data Science / ML Team
Testing: Benjamin validates performance doesn't break deterministic scenarios.

## EPIC 4 — Alert Triage Workspace (L1/L2)

### US-4.1 Prioritized alert queue sorted by aiRiskScore with P1–P4 and SLA countdown
R: Sarelle (Prototype) → Prasana/Sarelle (Production)
Testing: Benjamin tests sorting, filters, SLA colors, and data binding.

### US-4.2 Alert detail view shows executiveContext + recommendation + evidence flags with confidence
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates required fields render + missing-field handling.

### US-4.3 Analyst decision: accept/override recommendation with mandatory rationale (>=20 chars)
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates override enforcement + audit log creation.

### US-4.4 Filtering by typology tags (Structuring, Velocity, Network, Geo, TBML)
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates multi-select logic and counts.

### US-4.5 Manager SLA breach risk indicators and dashboard counts
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates threshold logic (<24h red, <48h amber).

## EPIC 5 — Case Investigation Workspace (L2/L3)

### US-5.1 Case opens from alert with pre-populated context (customerId, segment, typology, VaR)
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates linkage + record creation + audit.

### US-5.2 Transactions table with flagged highlights + core columns (date, amount, type, counterparty, geo)
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates highlighting + pagination + filters.

### US-5.3 Behavioral deviation metrics panel (frequency, cash usage, counterparty reuse, avg size)
R: Data Science / ML Team (feature calc) + Sarelle (UI)
Testing: Benjamin validates computed metrics vs scenario pack expectations.

### US-5.4 Network view renders nodes/edges with risk badges + historical SAR connections
R: Data Science / ML Team (graph features) + Sarelle (UI)
GPU note: keep heuristic graph CPU-first for exercise.

### US-5.5 Value-at-risk summary: total amount, time window, deviation %, peer percentile
R: Data Science / ML Team (calc) + Sarelle (UI)
Testing: Benjamin validates thresholds (>300% critical, >95th extreme).

## EPIC 6 — RFI Management

### US-6.1 Draft RFI from case with pre-populated customer + typology context
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates auto-fill + persistence.

### US-6.2 Track RFI status (Draft/Sent/Response/Overdue) with due dates
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates overdue logic and status transitions.

### US-6.3 Manager dashboard: response rate, avg response time, overdue count
R: Sarelle → Prasana/Sarelle
Testing: Benjamin validates KPI rollups.

## EPIC 7 — SAR Drafting & Filing Workflow

### US-7.1 Generate SAR draft package (subject, transaction summary, evidence, AI rationale, narrative template)
R: Data Science / ML Team (package assembly) + Sarelle (UI)
Testing: Benjamin validates narrative template + evidence completeness.

### US-7.2 SAR workflow: Draft → Review → Approved → Filed with role gating (L3+)
R: Prasana/Sarelle (Production implementation)
Testing: Benjamin validates permissions + audit events.

### US-7.3 SAR version history (at least version list; diff optional)
R: Prasana/Sarelle
Testing: Benjamin validates version increments + restore/view.

## EPIC 8 — Audit Logging & Compliance Reporting

### US-8.1 Append-only audit log for key events (alert creation, case open, override, RFI, SAR transitions)
R: Data Team (pipeline) + Prasana/Sarelle (app emits events)
Testing: Benjamin validates event coverage and immutability constraints.

### US-8.2 Override event reporting with rationale and original recommendation
R: Data Team + Sarelle
Testing: Benjamin validates searchable fields.

### US-8.3 Export audit logs for case/date range (CSV/PDF)
R: Prasana/Sarelle
Testing: Benjamin validates export formatting and content completeness.

## EPIC 9 — AI Infra Admin (Cost, latency, model registry, governance)

### US-9.1 Summary cards: Inference Cost MTD, Latency vs SLA, Volume, System Health
R: Prasana/Sarelle
Testing: Benjamin validates calculations + thresholds.

### US-9.2 Cost timeline filters (workload, granularity, env, cost metric)
R: Prasana/Sarelle
Testing: Benjamin validates filter correctness.

### US-9.3 Top cost drivers list with root-cause drilldown
R: Prasana/Sarelle
Testing: Benjamin validates ranking and expansion behavior.

### US-9.4 Model registry table (versions, hosting env, latency, error rate, health badge)
R: Prasana/Sarelle
Testing: Benjamin validates data completeness and sort.

### US-9.5 Governance footer: pending validation, cost thresholds, auto-throttling, audit logging
R: Prasana/Sarelle
Testing: Benjamin validates each indicator toggles based on dataset.

## EPIC 10 — Model Governance (Champion/Challenger + validation)

### US-10.1 Implement model registry dataset with validation schedule + segment metrics
R: Data Science / ML Team
Testing: Benjamin validates metrics fields exist and refresh.

### US-10.2 Champion vs Challenger evaluation run + promotion criteria checks
R: Data Science / ML Team
Testing: Benjamin validates reproducibility + threshold gate outputs.

### US-10.3 Drift monitoring (weekly) on key features vs training baseline
R: Data Science / ML Team
Monitoring owner: TBD (AI Platform Admin until named)

## EPIC 11 — Testing Framework (so QA knows exactly what to test)

### US-11.1 Create "AI Test Checklist" template attached to every AML use-case ticket
R: Benjamin
Covers: functional, performance, bias/fairness (if applicable), security, regression, end-to-end.

### US-11.2 Generate test cases from user stories + scenario packs (20 sample inputs)
R: Benjamin
Dependency: scenario packs + expected outcomes published by Data Team.

### US-11.3 Produce "Test Coverage Report" + "Edge Cases Documented" artifact for handoff
R: Benjamin
Handoff gate: required before moving Testing → Production.

## EPIC 12 — Deployment & Monitoring (Production readiness)

### US-12.1 GitHub repo structure + branching convention + Jira linking
R: Prasana/Sarelle
Governance: "version control everything" enforced.

### US-12.2 Establish baseline performance + monitoring dashboard link in Jira
R: Prasana/Sarelle
Monitoring owner: TBD

### US-12.3 Implement alerting on SLA latency / error rate / cost drift thresholds
R: TBD (AI Platform Admin / Ops)
Testing: Benjamin validates alerts configured (dry run).

## Quick "Owner Summary" by area

- **Data Team**: raw/curated/serving tables, synthetic data, lineage, audit dataset
- **Data Science / ML**: typology rules engine, risk scoring, features, model metrics, drift, champion/challenger
- **Sarelle**: prototype UI flows (triage/case/RFI/SAR/admin), Jira artifact linking, handoff fields
- **Benjamin**: test checklist, test cases, execution, test coverage & edge cases artifacts
- **Prasana/Sarelle**: production implementation, deployments, monitoring baseline, infra admin hardening
- **Monitoring TBD**: ongoing ops, alerts, drift triage, weekly review rhythm


---

# Synthetic Data Specification for Data Team

Below is a data-team–ready synthetic data specification for the AML Investigation Portal Foundry exercise. It tells you exactly what tables to generate, relationships, distributions, scenario packs, labels, and "expected outcomes" so models + QA can validate deterministically.

## 0) Synthetic Data Design Goals

### What the synthetic data must enable end-to-end
- Raw → Curated → Features → Alerts → Cases → RFIs → SARs → Audit logs → Infra Admin metrics
- Deterministic, testable typology scenarios (Structuring, Velocity, Geo, Network, TBML)
- Enough time series depth to compute 30/60/90-day rolling baselines
- Graph connectivity to support network risk + "prior SAR connections"
- Ground truth labels to evaluate precision/recall and champion/challenger comparisons

### Non-negotiables
- Referential integrity across all keys (customerId → accounts → transactions → alerts → cases → sarId).
- Time coherence: KYC review dates precede transactions; SAR filed dates after case open; audit events after actions.
- PII safety: fake but realistic; never use real names/addresses.

## 1) Target Volumes (Exercise Minimum)

| Entity | Target Count | Notes |
|--------|--------------|-------|
| Customers | 5,000 | Segment mix below |
| Accounts | 8,000 | Multiple accounts per customer |
| Counterparties | 25,000 | Beneficiaries/originators external registry |
| Transactions | 2–5 million | Over 6–12 months |
| Alerts | ~15,000 | ~0.3%–1% of txns depending on scenario density |
| Cases | ~3,000 | From subset of alerts |
| RFIs | ~1,200 | From subset of cases |
| SARs | ~800 | With version history (2–4 versions each) |
| Audit logs | 500k+ | Includes system + user actions |
| Inference events | 45k+ | Alerts × models (3 models min) |

## 2) Master Keys & ID Conventions

Use consistent prefixes for easy debugging:

- customerId: C-000001 … C-005000
- accountId: A-00000001 …
- counterpartyId: CP-00000001 …
- transactionId: TXN-YYYYMMDD-00000001
- alertId: ALT-YYYY-000001
- caseId: CASE-YYYY-000001
- rfiId: RFI-YYYY-000001
- sarId: SAR-YYYY-000001
- logId: LOG-00000001
- requestId (inference): REQ-YYYYMMDDHHMMSS-xxxx

## 3) Core Tables to Generate (Raw + Curated + Serving)

### 3.1 RAW tables (source-aligned)

These are "realistic inputs," even though synthetic.

#### raw_kyc_profiles
1 row per customer

Key fields:
- customerId, customerType (Retail/SME/Corp), legalName, incorporationCountry, incorporationDate (for entities)
- businessType/NAICS (for SME/Corp), annualRevenue, employeeCount
- registeredAddress (city/state/zip/country)
- operatingGeographies (list)
- expectedTransactionProfile (monthlyVolume/monthlyValue, primaryChannels, paymentTypes, expectedCounterpartiesGroup)
- riskRating (Low/Medium/High)
- lastKYCReview, nextKYCReviewDue
- pepStatus boolean
- sanctionsScreenDate, sanctionsScreenResult

Distributions:
- customerType: Retail 70%, SME 25%, Corp 5%
- riskRating by type:
  - Retail: Low 65%, Med 30%, High 5%
  - SME: Low 35%, Med 45%, High 20%
  - Corp: Low 20%, Med 50%, High 30%
- operatingGeographies: mostly 1–3 states for US entities; 1–4 countries for corp

#### raw_cbs_transactions
Covers: cash deposits/withdrawals, checks, internal transfers.

- transactionId, customerId, accountId
- amount, currency, timestamp, branchId (if branch), channel
- txnType (CASH_DEPOSIT / CASH_WITHDRAWAL / CHECK_DEPOSIT / INTERNAL_TRANSFER)
- location (state/city), counterpartyName (optional)

#### raw_swift_messages
Covers international wires (MT103-like)

- externalRef (UETR), orderingCustomer, beneficiary, senderBIC, receiverBIC
- amount, currency, valueDate, instructionDate, narrative/purpose
- countryFrom/countryTo
- mapping keys to customerId/accountId when internal

#### raw_ach_files
NACHA-like records

- traceNumber (externalRef), SECCode, originator, receiver, amount, effectiveDate, batchId

#### raw_card_transactions
ISO8583-like

- maskedPAN token, merchantId, MCC, amount, currency, timestamp
- terminalId, merchantGeo, authCode

#### raw_sanctions_results
- customerId OR counterpartyId
- screeningDate, listSource (OFAC/EU/UN), matchScore (0–100)
- result (Clear/Potential/Confirmed)

#### raw_negative_news_hits
- customerId OR counterpartyId
- hitDate, sourceCategory, riskTopic (fraud, corruption, trafficking, etc.)
- severity (Low/Med/High), excerpt (synthetic text)

#### raw_historical_sars
- sarId, filedDate, subjects (customerId/counterpartyId), typologies, narrativeSummary

Used for "prior SAR connections" in network scenarios.

### 3.2 CURATED tables (normalized canonical layer)

#### curated_customer_360
Derived from raw KYC + risk results:

- customerId, segment, riskRating, pepStatus, sanctionsSummary, adverseMediaSummary
- expected profile flattened (monthlyVolume/value, primary channels/types)

#### curated_counterparty_registry
- counterpartyId, name, country, riskTier (Low/Med/High), shellCompanyIndicator bool
- identifiers: addressHash, phoneHash (synthetic), beneficialOwnerHash (synthetic)

#### curated_payment_record (canonical PaymentRecord)
1 row per transaction across all channels

Include:
- transactionId, externalRef, batchId
- originator/beneficiary (names, masked accounts, bankBIC, country, customerId if internal)
- amount, currency, amountUSD, exchangeRate
- instructionDate, valueDate, executionTimestamp
- paymentType, channel
- purpose/narrative, MCC
- risk fields (initially empty or default): riskScore, flagged, flagReasons[], sanctionsScreenResult, pepIndicator, highRiskGeography
- compliance metadata: ctrRequired, ctrFiled, structuringIndicator, velocityFlag
- sourceSystem, rawMessageRef

#### curated_geo_risk_reference
- countryCode, riskTier (Low/Med/High), fatfStatus (Normal/Monitored/High-Risk)

(Synthetic reference table—needed for geo flags)

#### curated_typology_rules
Store:
- flag definitions (FLAG-STR-001 etc.)
- thresholds (e.g., "within $1,500 of CTR threshold", ">3 deposits/week")
- risk factor weights (structuring 0.7, velocity 0.5, geo 0.4, network 0.6, tbml 0.3)
- escalation cutoffs

### 3.3 FEATURE STORE tables (ML-ready)

#### Rolling behavior windows (per customerId)
fs_behavioral_features_30d, _60d, _90d

Fields:
- txnCount, txnValueUSD, avgTxnSizeUSD
- cashDepositCount/value, cashRatio
- distinctCounterpartyCount, counterpartyReuseRate
- channelDiversity (count), newChannelIndicator
- geoDiversity, newGeoIndicator
- baselineTxnCount/value (prior window), deviationMultiplier
- peerPercentileTxnValue (by segment) (synthetic peer model ok)

#### Typology features
- fs_structuring_features: nearCTRCount, nearCTRValue, multiBranchCount, clusterWindowScore, irregularDigitRate
- fs_velocity_features: freqSpikeScore, valueSpikeScore, roundAmountRate
- fs_geo_features: highRiskCountryCount, corridorNoveltyScore
- fs_network_features: graphDegree, sharedCounterpartyCountWithSAR, hop2RiskScore, shellEntityNeighborCount

#### Labels
fs_labels

Fields:
- entityType (Alert/Case/Customer/Transaction)
- entityId
- label (0/1) suspicious confirmed
- labelSource (synthetic "SAR filed", "closed-no-sar", "lookback")
- labelDate

### 3.4 SERVING tables (app-ready)

#### serving_alerts
- alertId, generatedAt, priority, aiRiskScore
- primaryRiskDriver, typologyTags[]
- customer block, valueAtRisk block, evidenceFlags[]
- systemRecommendation { outcome, rationale, modelContributions[] }
- executiveContext, sla { deadline, hoursRemaining, priority }

#### serving_cases
- caseId, status, openedDate, assignedTo
- customer block, triggeringAlert, typology, valueAtRisk
- transactionSummary, behavioralMetrics[], networkAnalysis
- analystDecision state + rationale

#### serving_rfis
- rfiId, caseId, status, createdDate, dueDate, questions[], response metadata

#### serving_sars
- sarId, caseId, status, version, createdDate, filingType
- institutionInfo, subjectInfo, suspiciousActivity (date range, amount, narrative)
- modelAttribution

#### serving_audit_logs (append-only)
- logId, timestamp, eventType, userId, userRole
- entityId, entityType, action, details, metadata

#### serving_inference_events
- requestId, timestamp, modelId/version
- alertId/caseId/customerId + transactionIds[]
- score, decision, confidence
- latencyMs, cpuOrGpu, costEstimate, errorCode

#### serving_infra_metrics
- time period rollups: cost MTD, cost per 1k, latency p50/p95, volume/day, system health
- efficiency metrics: slmCpuPercentage, llmEscalationRate, gatedRequests%, avgTokens, gpuUtilization

## 4) Realism: Distributions, Patterns, and Defaults

### 4.1 Transaction mix (overall)
Across all transactions:
- ACH: 35–45%
- Card: 25–35%
- CBS (cash/check/internal): 15–25%
- Wires (domestic+intl): 5–10%
- P2P/RTP: 3–8% (optional but useful for "new channel" flags)

### 4.2 Amount distributions (rough)
- Retail card: $5–$250 (heavy tail up to $2k)
- Retail ACH: $50–$2k
- SME ACH: $200–$15k
- Wires: $5k–$250k (corp up to $1M)
- Cash deposits:
  - normal: $100–$4k
  - structuring scenarios: $8.5k–$9.99k clustered

### 4.3 Time distributions
- Build a daily timestamp series across 6–12 months
- Introduce weekly seasonality (business days higher volume)
- Introduce month-end spikes for SME/corp

### 4.4 Customer expected profiles (KYC)
For each customer, generate:
- expected monthlyVolume and monthlyValue
- primary channels/types consistent with segment

Then generate actual txns around expected with noise except scenario packs.

## 5) Scenario Packs (Deterministic "Known Truth" Cases)

Create scenario packs as "fixtures" with exact expectations. Each pack should output:
- impacted customerIds (and related counterparties)
- transactionIds included
- expected flags + expected priority
- whether it should produce SAR label

### Pack A — Structuring (P1/P2)
How many customers: 60–120

Pattern:
- 8–30 cash deposits over 10–25 days
- amounts: $8,500–$9,999
- 60–90% of deposits within $1,500 of $10k threshold
- multi-branch: 2–4 branches (branchId changes)
- clustering: 3–6 deposits inside 4-day windows

Expected flags:
- FLAG-STR-001, STR-002, STR-003; optionally STR-004

Expected outcomes:
- aiRiskScore: 80–95
- priority: P1 if combined with velocity or network; else P2

Labeling:
- 40–60% of these customers lead to SAR

### Pack B — Velocity Spike (P1/P2/P3)
How many customers: 150–250

Pattern:
- baseline: 2–6 wires/month → spike: 10–25 wires/month OR ACH volume spike 3–5x
- deviation multiplier > 3x baseline for 90d
- optionally: round amounts ($50k, $100k)

Expected flags:
- FLAG-VEL-001, optionally VEL-004

Outcomes:
- P1 when combined with Geo; otherwise P2

Labeling:
- 20–40% SAR

### Pack C — Geographic Risk (P1/P3)
How many customers: 120–200

Pattern:
- new corridor to "high-risk/monitored" countries from geo reference
- inconsistent with operatingGeographies
- repeat payments 6–20 over 30–60 days

Expected flags:
- FLAG-GEO-001 or GEO-002 or GEO-004

Outcomes:
- P1 when combined with velocity or shell; else P3/P2

Labeling:
- 15–30% SAR

### Pack D — Network / Prior SAR Connections (P1)
How many customers: 80–150

Pattern:
- build a graph where subjects share:
  - 2 counterparties with historical SAR subjects
  - shared identifiers (addressHash/phoneHash)
  - 2–3 hop connections within a "ring"

Expected flags:
- FLAG-NET-001, NET-004; optionally NET-003 (shell indicators)

Outcomes:
- P1 almost always, auto-escalate L3

Labeling:
- 50–70% SAR

### Pack E — TBML (P1/P2)
How many customers: 40–80 (mostly SME/corp import/export)

Pattern:
- wires tied to "invoice" narrative with manipulated values
- deviation from "market comparable" (synthetic) > 30%
- phantom shipment: no trade doc reference (synthetic field null)

Expected flags:
- FLAG-TBM-001 and/or TBM-003

Outcomes:
- P2 unless combined with geo/network → P1

Labeling:
- 30–50% SAR

### Pack F — Clean Controls (Negative Examples)
How many customers: 500–1,000

Pattern:
- normal behavior near expected profile, no typology triggers

Outcome:
- either no alerts or low score < 50

Labeling:
- label = 0

## 6) How to Generate Alerts/Cases/SARs From Synthetic Transactions

### 6.1 Alert generation rules (exercise baseline)
For each customer daily:
- Compute rolling features (30/60/90)
- Apply rule triggers (structuring/velocity/geo/network/tbml)
- Compute risk factor scores (0–100 per factor)
- Weighted score:
  - weighted = 0.7*STR + 0.5*VEL + 0.4*GEO + 0.6*NET + 0.3*TBM
  - normalize to 0–100
- Map to priority + escalation

Alert density target: ~15,000 total alerts.

### 6.2 Case creation
Create cases from:
- all P1 alerts (or 70–90% of them)
- 20–40% of P2 alerts

Assign:
- L2 handles most; L3 for P1/85+ score

### 6.3 RFIs
Create RFIs from:
- ~40% of cases where evidence incomplete (e.g., new geo corridor, ambiguous purpose)

Status distribution:
- Draft 15%, Sent 45%, Response Received 30%, Overdue 10%

### 6.4 SARs + versioning
Create SARs from:
- 60–80% of P1 cases
- 10–25% of P2 cases

SAR status distribution (final snapshot):
- Filed 50%, Approved 10%, Review 15%, Draft 25%

Versioning:
- Each SAR: 2–4 versions
- Version changes: narrative edits, added evidence, corrected dates/amounts

## 7) Ground Truth Labels (fs_labels) — Make QA + Metrics Possible

### 7.1 Label sources (synthetic)
- SAR filed → positive labels
- Closed as non-suspicious → negative labels
- Lookback review → can add 5–10% "found missed suspicious" to simulate examiner findings

### 7.2 What to label
- Alert-level label: was it true suspicious?
- Case-level label: did it end in SAR?
- Customer-level label: risk class for long-term analytics

## 8) Graph / Network Data (how to build it)

### 8.1 Graph entities
Nodes:
- Customers (internal)
- Counterparties (external)
- Accounts (optional nodes)

Edges:
- payment edges (customer → counterparty) with counts/value/time ranges
- shared identifier edges (counterparty ↔ counterparty) using addressHash/phoneHash
- "prior SAR subject" links from raw_historical_sars

### 8.2 Graph construction requirements
For network scenario customers:
- ensure at least 1 connection to a prior SAR subject within ≤ 3 hops
- ensure sharedCounterpartyCountWithSAR > 2 for FLAG-NET-001
- include 1–3 shell entities flagged in counterparty registry

## 9) Sanctions + Adverse Media (for realism + geo/network combos)

### 9.1 Sanctions result distribution
Across all customers/counterparties:
- Clear 97–99%
- Potential match 1–2.5%
- Confirmed match 0.1–0.5% (rare)

Use matchScore:
- Clear: 0–30
- Potential: 60–85
- Confirmed: 90–100

### 9.2 Negative news distribution
- 2–5% of counterparties have adverse hits
- 0.5–2% of customers have adverse hits

Severity:
- Low 60%, Med 30%, High 10%

## 10) Audit Logs (append-only) — Required Events

Generate audit logs for:
- Alert creation (system)
- Alert viewed
- Disposition action (accept/override) + rationale
- Case opened, assigned, status changes
- RFI created/sent/received/overdue
- SAR created, version updates, status transitions, approval, filed
- Export events (audit export performed)

Volume guidance: average 15–30 logs per case, 5–10 per alert, plus infra/admin events.

## 11) Inference Events + Infra Metrics (so AI Admin dashboard works)

### 11.1 Inference events
For each alert, generate 3 model events:
- MODEL-001 Structuring (SLM/CPU)
- MODEL-002 Velocity (rules/CPU)
- MODEL-003 Network (GPU optional; for exercise can be CPU with "cpuOrGpu=CPU")

Fields:
- latencyMs distributions:
  - CPU models: p50 80–150ms, p95 250–450ms
  - "GPU model": p50 120–220ms, p95 300–600ms
- costEstimate:
  - CPU per inference: $0.0005–$0.002
  - GPU per inference: $0.002–$0.01 (only if you want GPU economics)
- errorCode: <1% errors, clustered on a few days to simulate incidents

### 11.2 Infra metrics (daily + MTD)
Generate:
- inferenceVolume per day: 50k–150k
- p50/p95 latency daily rollups
- inferenceCost daily + MTD
- efficiency:
  - slmCpuPercentage ~ 60–80%
  - llmEscalationRate ~ 20–40% (even if no LLM used, keep as "simulated gating")
  - gpuUtilization if GPU used: 60–85%
  - idleGpuTime: 15–40%

## 12) Edge Cases the Synthetic Data Must Include (for QA)

Create explicit records for:
- missing optional fields: narrative null, intermediary missing, MCC missing
- duplicate externalRef collisions (rare) to test dedup logic
- timezone boundary days (month-end, DST period)
- currency conversions (EUR/GBP/JPY) with amountUSD normalization
- very large transactions (corp wires up to $1–2M)
- empty transaction lists for a dormant customer (should not create alerts)
- sanctions "potential match" with false positive resolution

## 13) Deliverables Checklist (What the Data Team Should Hand Off)

- All tables generated (raw/curated/features/serving)
- Scenario pack manifest:
  - list of customerIds per pack
  - expected flags and priorities
  - expected SAR outcomes
- Label dataset (fs_labels) aligned with scenario packs
- Data dictionary for each table (columns + types + constraints)
- "Known good" validation queries:
  - top 20 P1 alerts with evidence flags
  - a network customer with ≤3 hop SAR connection
  - a structuring customer with clustered deposits
- Seed + generator config committed (reproducible runs)

## 14) Recommended Build Order (so nothing blocks)

1. Generate customers + counterparties + geo reference
2. Generate accounts
3. Generate baseline transactions (normal behavior)
4. Inject scenario packs (structuring/velocity/geo/network/tbml)
5. Generate raw sanctions + adverse media
6. Create historical SARs and wire to network pack
7. Build curated_payment_record and customer_360
8. Compute feature store tables
9. Generate alerts/cases/rfis/sars + audit logs
10. Generate inference events + infra metrics
