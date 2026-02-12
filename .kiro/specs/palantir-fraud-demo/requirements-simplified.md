# Enterprise Fraud & Risk Intelligence Platform - Requirements (DEMO VERSION)

## 0. Demo Scope

**This is a DEMO, not production.**

We are building ONE compelling fraud story that demonstrates Foundry's unique value in 15-20 minutes.

**The Golden Path:**
A suspicious transaction from a high-risk country → linked to a shared device → escalated to investigation → explained by AI.

**What We're Demonstrating:**
- ✅ Connected intelligence (ontology graph)
- ✅ Explainable scoring (risk factors)
- ✅ Operational action (escalate case, freeze account)
- ✅ AI grounded in ontology (summarize risk, draft report)

**What We're NOT Building:**
- ❌ Production-grade data pipelines
- ❌ Full governance framework
- ❌ Complex ML models
- ❌ Complete action library
- ❌ Comprehensive AI patterns

## 1. Executive Summary

### 1.1 Objective
Design and build a demonstration showcasing how Palantir Foundry enables enterprise fraud detection through:
- Real-time fraud detection across multiple data sources
- Network relationship analysis revealing hidden fraud patterns
- Explainable AI decisions meeting regulatory requirements
- Operational case management with audit trails

### 1.2 Use Case: XYZ Bank
XYZ Bank is a multinational retail and commercial bank serving 12M+ customers and processing 25M+ transactions daily.

**Business Problems:**
1. Fragmented risk visibility across disconnected systems
2. Slow investigation cycles (2-4 hours per case)
3. High false positive rates (>60%)
4. Regulatory pressure for explainable AI and audit trails

**Strategic Objective:**
Build a unified Enterprise Fraud & Risk Intelligence Platform that demonstrates how Foundry connects data, enables explainable AI, and supports operational case management.

## 2. User Stories (Simplified)

### 2.1 Data Integration
**As a** Data Engineer  
**I want to** integrate data from 4 disconnected risk systems  
**So that** we have a unified view of fraud signals

**Acceptance Criteria:**
- AC 2.1.1: System ingests transaction data from core banking system
- AC 2.1.2: System ingests customer data from CRM platform
- AC 2.1.3: System ingests device fingerprint data from fraud monitoring system
- AC 2.1.4: System ingests account data from core banking system
- AC 2.1.5* System ingests merchant data (OPTIONAL)

**Success Metric:** Data integration pipeline completes in <2 hours

### 2.2 Ontology Modeling
**As a** Fraud Analyst  
**I want to** view relationships between customers, accounts, transactions, and devices  
**So that** I can identify fraud patterns that span multiple entities

**Acceptance Criteria:**
- AC 2.2.1: Ontology includes Customer object with KYC status and risk rating
- AC 2.2.2: Ontology includes Account object linked to customers
- AC 2.2.3: Ontology includes Transaction object with amount, timestamp, and location
- AC 2.2.4: Ontology includes Device object with fraud flags
- AC 2.2.5: Ontology includes Case object for investigations
- AC 2.2.6* Ontology includes Merchant object (OPTIONAL)
- AC 2.2.7: Relationships are defined: Customer→Account, Account→Transaction, Customer→Device, Transaction→Case

**Success Metric:** Identify shared device fraud pattern (1 device linked to 3+ accounts)

### 2.3 Network Risk Visualization
**As a** Fraud Investigator  
**I want to** visualize network relationships around suspicious transactions  
**So that** I can identify coordinated fraud rings

**Acceptance Criteria:**
- AC 2.3.1: System displays suspicious transaction with all related entities
- AC 2.3.2: System shows devices linked to multiple accounts
- AC 2.3.3: System highlights shared risk factors in network graph
- AC 2.3.4: Graph renders in <3 seconds

**Success Metric:** Detect shared device fraud ring in <5 minutes (vs. hours manually)

### 2.4 Risk Scoring & Explainability
**As a** Compliance Officer  
**I want to** understand why a transaction was flagged as high-risk  
**So that** I can meet regulatory requirements for explainable AI

**Acceptance Criteria:**
- AC 2.4.1: System calculates risk score for each transaction using 3 simple rules
- AC 2.4.2: Rule 1: High-risk geography (40 points for KP, IR, RU, AE)
- AC 2.4.3: Rule 2: Device fraud history (30 points if fraud_flag = True)
- AC 2.4.4: Rule 3: Large transaction (30 points if amount > 10,000)
- AC 2.4.5: System displays risk_score and risk_factors array
- AC 2.4.6: Explanations are human-readable and audit-ready

**Success Metric:** Every flagged transaction has clear, explainable risk factors

### 2.5 Case Management Actions
**As a** Fraud Operations Manager  
**I want to** take operational actions on flagged cases  
**So that** I can prevent fraud losses and maintain audit compliance

**Acceptance Criteria:**
- AC 2.5.1: System provides "Escalate to Case" action from Transaction
- AC 2.5.2: System provides "Freeze Account" action from Account
- AC 2.5.3: Both actions maintain audit trail with timestamps and user attribution
- AC 2.5.4: Case assignment to investigators is tracked

**Success Metric:** Execute action from detection to response in <2 minutes

### 2.6 AI Investigation Assistant
**As a** Fraud Investigator  
**I want to** ask natural language questions about risk patterns  
**So that** I can accelerate investigations with AI-powered insights

**Acceptance Criteria:**
- AC 2.6.1: Investigator can ask "Summarize why this customer is high risk"
- AC 2.6.2: AI generates summary with data citations from ontology
- AC 2.6.3: Investigator can ask "Draft a compliance summary report"
- AC 2.6.4: AI generates structured report grounded in governed data
- AC 2.6.5: All AI responses are traceable to source data

**Success Metric:** AI provides relevant, grounded response in <5 seconds

## 3. Data Requirements

### 3.1 Source Data Files (4 Required + 1 Optional)

**Required:**
- raw_customers.csv (customer_id, full_name, email, risk_rating, kyc_status, country, onboarding_date)
- raw_accounts.csv (account_id, customer_id, account_type, open_date, status)
- raw_transactions.csv (transaction_id, account_id, merchant_id, amount, currency, timestamp, country, channel)
- raw_devices.csv (device_id, customer_id, ip_address, device_type, first_seen, fraud_flag)

**Optional:**
- raw_merchants.csv (merchant_id, merchant_name, mcc, country)

### 3.2 Data Quality Requirements (Minimal)
- AC 3.2.1: All date formats are standardized to ISO 8601
- AC 3.2.2: Boolean fraud flags are normalized (TRUE/FALSE)
- AC 3.2.3: No complex validation rules (demo data is assumed clean)

## 4. Non-Functional Requirements

### 4.1 Performance (Demo-Specific)
- NFR 4.1.1: Risk scoring calculations complete within 2 seconds
- NFR 4.1.2: Network visualizations render within 3 seconds
- NFR 4.1.3: AI assistant responses generate within 5 seconds
- NFR 4.1.4: Actions execute within 2 seconds

### 4.2 Auditability (Demo-Level)
- NFR 4.2.1: All user actions are logged with timestamps
- NFR 4.2.2: All AI decisions are explainable and traceable
- NFR 4.2.3: Risk scoring logic is transparent

### 4.3 Usability
- NFR 4.3.1: Demo narrative flows logically through 6 steps
- NFR 4.3.2: Visualizations are clear and compelling
- NFR 4.3.3: Technical complexity is abstracted for business users
- NFR 4.3.4: Total demo duration: 15-20 minutes

## 5. Success Metrics

### 5.1 Demo Effectiveness
- Clearly demonstrates value of unified fraud intelligence
- Shows potential investigation time reduction (from 2-4 hours to <30 minutes)
- Illustrates explainability for regulatory compliance
- Demonstrates operational actions (not just analytics)

### 5.2 Technical Validation
- All 6 demo steps execute successfully
- Ontology relationships are accurate and meaningful
- Risk scoring produces realistic and explainable results
- AI assistant provides relevant and grounded responses
- No technical failures during demo

### 5.3 Business Impact
- Demo resonates with target audience (banks, fintech, insurance)
- Showcases Palantir Foundry's unique capabilities
- Differentiates from traditional fraud detection tools
- Positions Foundry as operational intelligence platform

### 5.4 Quantifiable Value Proposition (Demo Claims)

| Metric | Current State | Target State | Improvement |
|--------|---------------|--------------|-------------|
| Investigation Time | 2-4 hours/case | <30 minutes | 80% reduction |
| False Positive Rate | 60% | <20% | 67% reduction |
| Fraud Ring Detection | Weeks | Minutes | 10x faster |
| Data Integration | 2-3 days | <2 hours | 90% faster |

**Estimated Annual Impact:**
- Fraud loss prevention: $2M+
- Operational efficiency: 1000+ hours saved
- Compliance risk reduction: Audit-ready explainability

## 6. Out of Scope (Explicitly NOT Building)

The following are explicitly out of scope for this demo:

**Data & Pipelines:**
- ❌ Real-time streaming data ingestion
- ❌ Complex data quality rules
- ❌ Data profiling dashboards
- ❌ External watchlist integration (OFAC, FATF)
- ❌ MCC enrichment pipelines

**Ontology:**
- ❌ Cards object
- ❌ Alert object (using risk_score filter instead)
- ❌ Complex computed properties
- ❌ 8+ relationship types

**Risk Scoring:**
- ❌ 5-factor scoring with behavioral baselines
- ❌ Account average calculations
- ❌ Historical analysis
- ❌ Machine learning models

**Actions:**
- ❌ Request Additional KYC action
- ❌ Close Case action
- ❌ Report to Compliance action
- ❌ Complex workflow orchestration

**AI:**
- ❌ Pattern matching ("What similar cases exist?")
- ❌ Complex similarity search
- ❌ Historical case retrieval
- ❌ Full AI pattern library

**Testing & Operations:**
- ❌ Production-grade scalability testing
- ❌ Performance testing
- ❌ Load testing
- ❌ Security hardening
- ❌ Integration with actual banking systems
- ❌ Real customer PII (all data is synthetic)

## 7. Demo Execution Strategy

### 7.1 Target Audiences

**For Executive Audience (C-Suite, VP-level):**
- Duration: 15-20 minutes
- Focus: Business problem → Ontology graph → Explainable AI + Actions
- Skip: Technical architecture details

**For Technical Audience (Architects, Data Engineers):**
- Duration: 25-30 minutes
- Show: Data integration → Ontology design → Risk scoring → Actions → AIP

**For Mixed Audience (Most Common):**
- Duration: 20 minutes
- Use: "1 High-Impact Fraud Scenario" approach (The Golden Path)

### 7.2 The Golden Path Scenario

**Select 1 transaction that demonstrates:**
- Unusual transaction size (>10,000)
- High-risk geography (KP, IR, RU, or AE)
- Shared device across multiple accounts (fraud_flag = True)
- Clear network relationships

**This single scenario showcases all 6 capabilities in one narrative flow.**

### 7.3 Demo Anti-Patterns to Avoid
- ❌ Showing every feature sequentially
- ❌ Getting lost in technical details
- ❌ Spending >5 minutes on any single section
- ❌ Apologizing for "what's not built yet"

### 7.4 Demo Best Practices
- ✅ Tell a story, not a feature list
- ✅ Focus on business outcomes
- ✅ Keep momentum and energy high
- ✅ Emphasize differentiation from traditional tools

## 8. Build Timeline

**Total Build Time: 8-13 days**

- Phase 1: Data Integration (1-2 days)
- Phase 2: Ontology Configuration (2-3 days)
- Phase 3: Workshop UI Development (2-3 days)
- Phase 4: Actions & Workflows (1-2 days)
- Phase 5: AI Investigation Assistant (1-2 days)
- Phase 6: Demo Preparation & Rehearsal (1 day)

**Principle: Maximum impact with minimum build effort.**

This is a demo, not production. Optimize for clarity and impact, not completeness.
