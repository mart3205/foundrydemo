# Enterprise Fraud & Risk Intelligence Platform - Implementation Tasks (DEMO VERSION)

## Demo Build Philosophy

**Target: 8-13 days total build time**
**Focus: ONE compelling fraud story (The Golden Path)**
**Principle: Maximum impact, minimum build effort**

---

## Phase 1: Data Integration (1-2 days)

### 1. Set Up Foundry Environment
- [ ] 1.1 Create new Foundry project: "XYZ Bank Fraud Intelligence Demo"
- [ ] 1.2 Set up Code Repository for data pipelines
- [ ] 1.3 Create folder structure: /raw-data, /clean-data, /ontology

### 2. Upload Raw Data
- [ ] 2.1 Upload raw_customers.csv to Foundry dataset
- [ ] 2.2 Upload raw_accounts.csv to Foundry dataset
- [ ] 2.3 Upload raw_transactions.csv to Foundry dataset
- [ ] 2.4 Upload raw_devices.csv to Foundry dataset
- [ ] 2.5* Upload raw_merchants.csv to Foundry dataset (OPTIONAL)

### 3. Build Minimal Pipelines
- [ ] 3.1 Create Python transform for customers dataset
  - [ ] 3.1.1 Standardize onboarding_date to ISO 8601
  - [ ] 3.1.2 Keep risk_rating, kyc_status, country as-is (no validation)
- [ ] 3.2 Create Python transform for accounts dataset
  - [ ] 3.2.1 Standardize open_date to ISO 8601
  - [ ] 3.2.2 Keep account_type, status as-is
- [ ] 3.3 Create Python transform for transactions dataset
  - [ ] 3.3.1 Standardize timestamp to ISO 8601
  - [ ] 3.3.2 Keep amount, currency, country, channel as-is
- [ ] 3.4 Create Python transform for devices dataset
  - [ ] 3.4.1 Standardize first_seen to ISO 8601
  - [ ] 3.4.2 Normalize fraud_flag to boolean (TRUE/FALSE)
- [ ] 3.5* Create Python transform for merchants dataset (OPTIONAL)
- [ ] 3.6 Run all pipelines and validate data loads

---

## Phase 2: Ontology Configuration (2-3 days)

### 4. Define Core Objects (5 Required)
- [ ] 4.1 Create Customer object
  - [ ] 4.1.1 Configure primary key: customer_id
  - [ ] 4.1.2 Add properties: full_name, email, risk_rating, kyc_status, country
  - [ ] 4.1.3 Link to customers_clean dataset
  - [ ] 4.1.4* Add computed property: total_accounts (OPTIONAL)
  - [ ] 4.1.5* Add computed property: total_devices (OPTIONAL)

- [ ] 4.2 Create Account object
  - [ ] 4.2.1 Configure primary key: account_id
  - [ ] 4.2.2 Add properties: account_type, status
  - [ ] 4.2.3 Link to accounts_clean dataset

- [ ] 4.3 Create Transaction object
  - [ ] 4.3.1 Configure primary key: transaction_id
  - [ ] 4.3.2 Add properties: amount, currency, timestamp, country, channel
  - [ ] 4.3.3 Link to transactions_clean dataset

- [ ] 4.4 Create Device object
  - [ ] 4.4.1 Configure primary key: device_id
  - [ ] 4.4.2 Add properties: ip_address, device_type, fraud_flag
  - [ ] 4.4.3 Link to devices_clean dataset
  - [ ] 4.4.4* Add computed property: account_count (OPTIONAL)

- [ ] 4.5 Create Case object
  - [ ] 4.5.1 Configure primary key: case_id (auto-generated)
  - [ ] 4.5.2 Add properties: status, priority, assigned_to, created_date, resolution_notes
  - [ ] 4.5.3 Create backing dataset for cases

- [ ] 4.6* Create Merchant object (OPTIONAL)
  - [ ] 4.6.1 Configure primary key: merchant_id
  - [ ] 4.6.2 Add properties: merchant_name, mcc, country
  - [ ] 4.6.3 Link to merchants_clean dataset

### 5. Configure Relationships (4 Required)
- [ ] 5.1 Create "has account" link: Customer → Account (one-to-many)
- [ ] 5.2 Create "executed transaction" link: Account → Transaction (one-to-many)
- [ ] 5.3 Create "uses device" link: Customer → Device (many-to-many)
- [ ] 5.4 Create "escalated to case" link: Transaction → Case (many-to-one)
- [ ] 5.5* Create "paid to merchant" link: Transaction → Merchant (OPTIONAL)

### 6. Implement Simplified Risk Scoring
- [ ] 6.1 Create Foundry Function: calculate_risk_score
- [ ] 6.2 Implement Rule 1: High-risk geography (40 points)
  - [ ] 6.2.1 Define HIGH_RISK_COUNTRIES = ['KP', 'IR', 'RU', 'AE']
  - [ ] 6.2.2 Check if transaction.country in list
- [ ] 6.3 Implement Rule 2: Device fraud history (30 points)
  - [ ] 6.3.1 Check if linked device has fraud_flag = True
- [ ] 6.4 Implement Rule 3: Large transaction (30 points)
  - [ ] 6.4.1 Check if transaction.amount > 10000
- [ ] 6.5 Return risk_score and risk_factors array
- [ ] 6.6 Attach function to Transaction object
- [ ] 6.7 Test function on 5+ sample transactions

---

## Phase 3: Workshop UI Development (2-3 days)

### 7. Build Investigation Dashboard
- [ ] 7.1 Create new Workshop application: "Fraud Investigation Console"
- [ ] 7.2 Add Object Set: High-Risk Transactions (filter: risk_score > 70)
- [ ] 7.3 Add Object Table widget showing transaction details
- [ ] 7.4 Add filters: date range, country, risk_score
- [ ] 7.5 Add sorting by risk_score descending
- [ ] 7.6 Configure click-through to transaction detail view

### 8. Build Transaction Detail View
- [ ] 8.1 Create detail page for Transaction object
- [ ] 8.2 Add property panel: amount, timestamp, country, channel
- [ ] 8.3 Add risk score visualization (gauge or progress bar)
- [ ] 8.4 Add risk factors explanation panel (bullet list)
- [ ] 8.5 Add related entities section: Customer, Account, Device
- [ ] 8.6 Add action buttons: Escalate to Case, Freeze Account

### 9. Build Network Graph Visualization
- [ ] 9.1 Add Graph widget to transaction detail view
- [ ] 9.2 Configure starting node: selected Transaction
- [ ] 9.3 Add expansion rules:
  - [ ] 9.3.1 Transaction → Account → Customer
  - [ ] 9.3.2 Customer → Devices
  - [ ] 9.3.3 Customer → Other Accounts
  - [ ] 9.3.4 Other Accounts → Other Transactions (1 hop only)
  - [ ] 9.3.5* Transaction → Merchant (OPTIONAL)
- [ ] 9.4 Configure node styling: color by risk level (green/yellow/red)
- [ ] 9.5 Add legend explaining node colors
- [ ] 9.6 Test graph renders in <3 seconds

### 10. Build Case Detail View
- [ ] 10.1 Create detail page for Case object
- [ ] 10.2 Add case properties: status, priority, assigned_to
- [ ] 10.3 Add linked transaction panel
- [ ] 10.4 Add notes section for investigator comments

---

## Phase 4: Actions & Workflows (1-2 days)

### 11. Implement "Escalate to Case" Action
- [ ] 11.1 Create Action in Ontology Manager
- [ ] 11.2 Configure trigger: Transaction object
- [ ] 11.3 Add parameters: priority (enum: Low, Medium, High, Critical), assigned_to (string)
- [ ] 11.4 Implement logic: Create new Case object, link Transaction to Case
- [ ] 11.5 Add audit logging: user, timestamp
- [ ] 11.6 Test action on sample transaction

### 12. Implement "Freeze Account" Action
- [ ] 12.1 Create Action in Ontology Manager
- [ ] 12.2 Configure trigger: Account object
- [ ] 12.3 Add parameter: reason (string, required)
- [ ] 12.4 Implement logic: Update Account.status = "Frozen"
- [ ] 12.5 Add audit logging: user, timestamp, reason
- [ ] 12.6 Test action on sample account

---

## Phase 5: AI Investigation Assistant (1-2 days)

### 13. Set Up AIP Integration
- [ ] 13.1 Enable AIP in Foundry project
- [ ] 13.2 Configure ontology context for AIP
- [ ] 13.3 Grant AIP access to all object types

### 14. Configure Risk Summarization Query
- [ ] 14.1 Create AIP query template: "Summarize why this customer is high risk"
- [ ] 14.2 Configure context: Customer object + linked Devices + Transactions with high risk_score
- [ ] 14.3 Define output format: bullet points with data citations
- [ ] 14.4 Test query on 3+ high-risk customer samples
- [ ] 14.5 Validate response accuracy and grounding

### 15. Configure Report Generation Query
- [ ] 15.1 Create AIP query template: "Draft a compliance summary report"
- [ ] 15.2 Configure context: Case object + linked Transaction + Customer + risk factors
- [ ] 15.3 Define report structure: executive summary, key facts, risk factors, recommendation
- [ ] 15.4 Test query on 2+ sample cases
- [ ] 15.5 Validate report completeness

### 16. Build AIP Chat Interface
- [ ] 16.1 Add AIP Chat widget to Transaction Detail view
- [ ] 16.2 Configure pre-defined query buttons for 2 queries
- [ ] 16.3 Test end-to-end chat experience

---

## Phase 6: Demo Preparation & Rehearsal (1 day)

### 17. Identify Demo Scenario
- [ ] 17.1 Find 1 high-risk transaction (risk_score > 70) with:
  - [ ] 17.1.1 High-risk country (KP, IR, RU, or AE)
  - [ ] 17.1.2 Device with fraud_flag = True
  - [ ] 17.1.3 Large amount (>10,000)
- [ ] 17.2 Verify this transaction has clear network relationships
- [ ] 17.3 Document transaction_id for demo

### 18. Test Demo Flow - End to End
- [ ] 18.1 Step 1: Show 4 CSV files (2 minutes)
- [ ] 18.2 Step 2: Show Ontology Manager with 5 objects (3 minutes)
- [ ] 18.3 Step 3: Open Workshop, expand network graph (5 minutes)
- [ ] 18.4 Step 4: Show risk score breakdown (3 minutes)
- [ ] 18.5 Step 5: Execute "Escalate to Case" and "Freeze Account" actions (3 minutes)
- [ ] 18.6 Step 6: Ask AI 2 questions, show responses (4 minutes)
- [ ] 18.7 Time total duration (target: 20 minutes)

### 19. Prepare Demo Backup
- [ ] 19.1 Take screenshots of each demo step
- [ ] 19.2 Record video walkthrough as backup
- [ ] 19.3 Test demo on different browser

### 20. Create Demo Script
- [ ] 20.1 Write talking points for each step
- [ ] 20.2 Prepare answers to common questions
- [ ] 20.3 Create one-pager summarizing demo value

---

## Phase 7: Final Validation (Half day)

### 21. Technical Validation
- [ ] 21.1 Verify all 5 object types are configured correctly
- [ ] 21.2 Test risk scoring function returns expected results
- [ ] 21.3 Confirm network graph renders in <3 seconds
- [ ] 21.4 Test both actions execute successfully
- [ ] 21.5 Validate AI responses are grounded in data

### 22. Demo Readiness Check
- [ ] 22.1 Run full demo flow without errors
- [ ] 22.2 Confirm narrative is clear and compelling
- [ ] 22.3 Verify backup materials are ready
- [ ] 22.4 Demo is ready for delivery

---

## Summary: What We're NOT Building

To keep this demo-focused, we are explicitly SKIPPING:

❌ Cards object and pipelines
❌ Alert object (using risk_score filter instead)
❌ MCC enrichment
❌ External watchlist integration
❌ Complex data quality rules
❌ Data profiling dashboards
❌ 5-factor risk scoring (using 3 simple rules)
❌ Account average calculations
❌ Behavioral baselines
❌ Multiple fraud patterns (focusing on shared device only)
❌ 5 actions (keeping only 2)
❌ Full AI pattern library (keeping only 2 queries)
❌ Performance testing
❌ Load testing
❌ Security hardening

**Total Tasks: 22 main tasks (vs. 52 in production version)**
**Total Time: 8-13 days (vs. 4-6 weeks in production version)**
**Impact: Same compelling demo story, 70% less build effort**

---

## Task Execution Notes

- Tasks marked with `*` are OPTIONAL and can be skipped if time is limited
- Focus on completing Phases 1-5 first (core functionality)
- Phase 6 (Demo Rehearsal) is critical - do not skip
- If running behind schedule, skip all OPTIONAL tasks
- Minimum viable demo: Complete all non-optional tasks in Phases 1-6
