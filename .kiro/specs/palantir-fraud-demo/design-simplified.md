# Enterprise Fraud & Risk Intelligence Platform - Technical Design (DEMO VERSION)

## 0. Demo Philosophy: Maximum Impact, Minimum Build

**Critical Principle: This is a DEMO, not production.**

We are building ONE compelling fraud story that demonstrates:
- ✅ Connected intelligence (ontology graph)
- ✅ Explainable scoring (risk factors)
- ✅ Operational action (escalate case, freeze account)
- ✅ AI grounded in ontology (summarize risk, draft report)

Everything else is optional and can be skipped.

**The Golden Path:**
A suspicious transaction from a high-risk country → linked to a shared device → and a high-risk merchant → escalated to investigation → explained by AI.

That's the entire demo. 15-20 minutes. Maximum impact.

## 1. Simplified Architecture

### 1.1 Why Not Traditional Fraud Systems?

**Rule Engines Are Static** - Cannot adapt to evolving fraud patterns
**Graph Databases Lack Governance** - No enterprise data governance or audit trails
**BI Tools Lack Operational Actions** - Cannot trigger actions like freeze account
**ML Models Lack Explainability** - Fail regulatory requirements (GDPR, FCRA, ECOA)

**Palantir Foundry Differentiators:**
- Governed data integration with lineage
- Ontology-based connected intelligence
- Operational workflows built-in
- Explainable AI traceable to source data

### 1.2 Demo Architecture (Minimal Viable)

```
┌─────────────────────────────────────────────────────────────┐
│              Workshop UI | Network Graph | AIP              │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│         2 Actions | 1 Risk Function | Case Management       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              5 Objects: Customer | Account |                │
│         Transaction | Device | Case (+ Merchant optional)   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              4 Simple Pipelines (minimal transforms)         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│         4 CSVs: Customers | Accounts | Transactions |       │
│         Devices (+ Merchants optional)                       │
└─────────────────────────────────────────────────────────────┘
```

**What We Removed:**
- ❌ Cards object (not needed for fraud story)
- ❌ Alert object (use risk_score > 70 filter instead)
- ❌ Complex data quality framework
- ❌ 5 actions (keep only 2: Escalate to Case, Freeze Account)
- ❌ 5-factor risk scoring (simplify to 3 rules)
- ❌ Full AI pattern library (keep 2 queries)
- ❌ MCC enrichment pipelines
- ❌ External watchlist integration
- ❌ Performance testing

**What We Kept:**
- ✅ Core ontology (5 objects minimum)
- ✅ Network graph visualization
- ✅ Simple risk scoring with explanation
- ✅ 2 operational actions
- ✅ 2 AI queries
- ✅ Full demo narrative

## 2. Simplified Data Integration

### 2.1 Minimal Pipeline Strategy

**Pipeline 1: Customer Data**
- Source: raw_customers.csv
- Minimal transforms:
  - Standardize date format (onboarding_date)
  - Keep risk_rating, kyc_status, country as-is
- Output: customers_clean

**Pipeline 2: Account Data**
- Source: raw_accounts.csv
- Minimal transforms:
  - Standardize date format (open_date)
  - Keep account_type, status as-is
- Output: accounts_clean

**Pipeline 3: Transaction Data**
- Source: raw_transactions.csv
- Minimal transforms:
  - Standardize timestamp
  - Keep amount, currency, country, channel as-is
- Output: transactions_clean

**Pipeline 4: Device Data**
- Source: raw_devices.csv
- Minimal transforms:
  - Standardize date format (first_seen)
  - Normalize fraud_flag to boolean (TRUE/FALSE)
- Output: devices_clean

**Pipeline 5 (Optional): Merchant Data**
- Source: raw_merchants.csv
- Minimal transforms:
  - Keep merchant_name, mcc, country as-is
- Output: merchants_clean

**Skip Entirely:**
- ❌ Data profiling
- ❌ Error datasets
- ❌ Complex validation rules
- ❌ External watchlist integration
- ❌ MCC description enrichment

## 3. Simplified Ontology Design

### 3.1 Core Objects (5 Required)

**Customer Object**
- Primary Key: customer_id
- Properties:
  - full_name (string)
  - email (string)
  - risk_rating (string: Low, Medium, High)
  - kyc_status (string: Verified, Pending, Failed)
  - country (string)
- Computed Properties (optional):
  - total_accounts (count linked accounts)
  - total_devices (count linked devices)

**Account Object**
- Primary Key: account_id
- Properties:
  - account_type (string: Checking, Savings, CreditLine)
  - status (string: Active, Closed, Frozen)
- Computed Properties (optional):
  - transaction_count (count linked transactions)

**Transaction Object**
- Primary Key: transaction_id
- Properties:
  - amount (decimal)
  - currency (string)
  - timestamp (timestamp)
  - country (string)
  - channel (string: POS, Online, Mobile, ATM)
- Computed Properties (REQUIRED):
  - risk_score (function-backed)
  - risk_factors (function-backed)

**Device Object**
- Primary Key: device_id
- Properties:
  - ip_address (string)
  - device_type (string: iOS, Android, Windows, Mac)
  - fraud_flag (boolean)
- Computed Properties (optional):
  - account_count (count linked accounts)

**Case Object**
- Primary Key: case_id
- Properties:
  - status (string: Open, InProgress, Resolved, Closed)
  - priority (string: Low, Medium, High, Critical)
  - assigned_to (string)
  - created_date (timestamp)
  - resolution_notes (string)

**Merchant Object (Optional)**
- Primary Key: merchant_id
- Properties:
  - merchant_name (string)
  - mcc (string)
  - country (string)

### 3.2 Relationships (Minimal)

**Required:**
- Customer → Account (one-to-many)
- Account → Transaction (one-to-many)
- Customer → Device (many-to-many)
- Transaction → Case (many-to-one) - for escalation

**Optional:**
- Transaction → Merchant (many-to-one)

**Skip:**
- ❌ Alert object and relationships
- ❌ Card object and relationships

## 4. Simplified Risk Scoring

### 4.1 3-Rule Risk Scoring Function

**Implemented as Foundry Function on Transaction object:**

```python
def calculate_risk_score(transaction):
    score = 0
    factors = []
    
    # Rule 1: High-risk geography (40 points)
    HIGH_RISK_COUNTRIES = ['KP', 'IR', 'RU', 'AE']
    if transaction.country in HIGH_RISK_COUNTRIES:
        score += 40
        factors.append(f"High-risk geography: {transaction.country}")
    
    # Rule 2: Device fraud history (30 points)
    device = transaction.account.customer.devices[0]  # simplified
    if device and device.fraud_flag == True:
        score += 30
        factors.append("Device linked to prior fraud")
    
    # Rule 3: Large transaction (30 points)
    if transaction.amount > 10000:
        score += 30
        factors.append(f"Unusual transaction size: {transaction.amount}")
    
    return {
        "risk_score": score,
        "risk_factors": factors
    }
```

**That's it. No:**
- ❌ Account average calculations
- ❌ Behavioral baselines
- ❌ Historical analysis
- ❌ Cross-border logic
- ❌ Merchant risk scoring

**This simple 3-rule system is enough to demonstrate explainability.**

## 5. Simplified Network Analysis

### 5.1 One Fraud Pattern: Shared Device

**Demo Pattern:**
- Find devices linked to multiple accounts (device.account_count > 3)
- Show in graph: Device node → Multiple Account nodes → Transactions
- Highlight: "This device is shared across 5 accounts - fraud ring indicator"

**Skip:**
- ❌ Merchant concentration patterns
- ❌ IP address clustering
- ❌ Cross-border ring analysis
- ❌ Complex graph algorithms

**One pattern is enough for demo impact.**

## 6. Simplified Actions

### 6.1 Two Actions Only

**Action 1: Escalate to Case**
- Trigger: From Transaction object
- Logic: Create new Case object, link Transaction to Case
- Parameters: priority (enum), assigned_to (string)
- Audit: Log user, timestamp

**Action 2: Freeze Account**
- Trigger: From Account object
- Logic: Update Account.status = "Frozen"
- Parameters: reason (string)
- Audit: Log user, timestamp, reason

**Skip:**
- ❌ Request Additional KYC
- ❌ Close Case
- ❌ Report to Compliance

**Two actions demonstrate operational power. That's enough.**

## 7. Simplified AI Integration

### 7.1 Two AI Queries Only

**Query 1: Risk Summarization**
- User Input: "Summarize why this customer is high risk"
- AIP Logic: Retrieve Customer + linked Devices + Transactions with high risk_score
- Output: Bullet point summary with data citations

**Query 2: Report Generation**
- User Input: "Draft a compliance summary report"
- AIP Logic: Retrieve Case + linked Transaction + Customer + risk factors
- Output: Structured report with key facts

**Skip:**
- ❌ Pattern matching ("What similar cases exist?")
- ❌ Complex similarity search
- ❌ Historical case retrieval

**Two queries demonstrate AI grounding. That's enough.**

## 8. Demo Flow (15-20 Minutes)

### Step 1: Show Disconnected Data (2 minutes)
- Display 4 CSV files side-by-side
- Narrative: "Data is siloed, fraud signals are invisible"

### Step 2: Show Ontology (3 minutes)
- Navigate to Ontology Manager
- Show 5 object types and relationships
- Narrative: "This is a living network of risk relationships"

### Step 3: Network Visualization (5 minutes)
- Open Workshop to high-risk transaction (risk_score > 70)
- Expand graph: Transaction → Customer → Devices → Other Accounts
- Highlight shared device pattern
- Narrative: "Fraud is rarely isolated. It is relational."

### Step 4: Risk Explanation (3 minutes)
- Click on flagged transaction
- Show risk_score = 70+ and risk_factors breakdown
- Narrative: "We are not just predicting fraud — we are explaining it."

### Step 5: Take Action (3 minutes)
- Click "Escalate to Case" action
- Create case, assign investigator
- Click "Freeze Account" action
- Show audit trail
- Narrative: "Foundry is not just analytics — it is operational risk management."

### Step 6: AI Assistant (4 minutes)
- Open AIP chat
- Ask: "Summarize why this customer is high risk"
- Show AI response with data citations
- Ask: "Draft a compliance summary report"
- Show generated report
- Narrative: "AI is grounded in governed, auditable enterprise data."

**Total: 20 minutes. Maximum impact.**

## 9. Implementation Strategy

### Phase 1: Data (1-2 days)
- Upload 4 CSVs to Foundry
- Build 4 simple pipelines with minimal transforms
- Validate data loads correctly

### Phase 2: Ontology (2-3 days)
- Define 5 object types
- Configure 4 relationships
- Implement risk scoring function (3 rules)
- Test computed properties

### Phase 3: Workshop UI (2-3 days)
- Build investigation dashboard
- Configure network graph
- Add filters for high-risk transactions (risk_score > 70)
- Test visualization

### Phase 4: Actions (1-2 days)
- Implement "Escalate to Case" action
- Implement "Freeze Account" action
- Test audit logging

### Phase 5: AIP (1-2 days)
- Set up AIP with ontology context
- Configure 2 query patterns
- Test AI responses

### Phase 6: Demo Rehearsal (1 day)
- Practice full 20-minute flow
- Refine narrative
- Prepare backup screenshots

**Total: 8-13 days for complete demo build.**

## 10. What We Completely Skip

For demo purposes, DO NOT BUILD:

❌ Cards object and pipelines
❌ Alert object (use risk_score filter)
❌ MCC enrichment
❌ External watchlist integration
❌ Complex data quality rules
❌ Data profiling dashboards
❌ 5-factor risk scoring
❌ Account average calculations
❌ Behavioral baselines
❌ Multiple fraud patterns
❌ 5 actions (keep 2)
❌ Full AI pattern library (keep 2 queries)
❌ Performance testing
❌ Load testing
❌ Security hardening

**Focus on the Golden Path. Everything else is distraction.**

## 11. Success Criteria (Demo-Specific)

### Technical Success
- 5 object types configured correctly
- Risk scoring function returns explainable results
- Network graph renders in <3 seconds
- 2 actions execute successfully
- 2 AI queries return grounded responses

### Business Success
- Demo tells compelling fraud story in 20 minutes
- Audience understands Foundry's unique value
- Differentiation from traditional tools is clear
- Regulatory compliance benefits are demonstrated

### Demo Execution Success
- No technical failures during live demo
- Narrative flows smoothly
- Audience asks follow-up questions
- Leads to next conversation

**This is a demo, not production. Optimize for impact, not completeness.**
