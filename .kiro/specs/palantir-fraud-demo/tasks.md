# Enterprise Fraud & Risk Intelligence Platform - Implementation Tasks

## Phase 1: Data Integration & Pipeline Development

### 1. Set Up Foundry Environment
- [ ] 1.1 Create new Foundry project: "XYZ Bank Fraud Intelligence Demo"
- [ ] 1.2 Set up Code Repository for data pipelines
- [ ] 1.3 Configure project permissions and access controls
- [ ] 1.4 Create folder structure: /raw-data, /clean-data, /ontology, /functions, /actions

### 2. Upload Raw Data Sources
- [ ] 2.1 Upload raw_customers.csv to Foundry dataset
- [ ] 2.2 Upload raw_accounts.csv to Foundry dataset
- [ ] 2.3 Upload raw_cards.csv to Foundry dataset
- [ ] 2.4 Upload raw_transactions.csv to Foundry dataset
- [ ] 2.5 Upload raw_devices.csv to Foundry dataset
- [ ] 2.6 Upload raw_merchants.csv to Foundry dataset

### 3. Build Customer Data Pipeline
- [ ] 3.1 Create Python transform for customers dataset
  - [ ] 3.1.1 Standardize onboarding_date to ISO 8601 format
  - [ ] 3.1.2 Normalize country codes to ISO 3166-1 alpha-2
  - [ ] 3.1.3 Validate email format using regex
  - [ ] 3.1.4 Map risk_rating to enum (Low, Medium, High)
  - [ ] 3.1.5 Map kyc_status to enum (Verified, Pending, Failed)
- [ ] 3.2 Run pipeline and validate output dataset
- [ ] 3.3 Profile data quality metrics

### 4. Build Account Data Pipeline
- [ ] 4.1 Create Python transform for accounts dataset
  - [ ] 4.1.1 Standardize open_date to ISO 8601 format
  - [ ] 4.1.2 Map account_type to enum (Checking, Savings, CreditLine)
  - [ ] 4.1.3 Map status to enum (Active, Closed, Frozen)
  - [ ] 4.1.4 Join with customers to validate customer_id foreign key
- [ ] 4.2 Run pipeline and validate output dataset
- [ ] 4.3 Profile data quality metrics

### 5. Build Card Data Pipeline
- [ ] 5.1 Create Python transform for cards dataset
  - [ ] 5.1.1 Standardize last_transaction to ISO 8601 format
  - [ ] 5.1.2 Map card_type to enum (Credit, Debit)
  - [ ] 5.1.3 Map issuer to enum (VISA, MASTERCARD, AMEX)
  - [ ] 5.1.4 Join with accounts to validate account_id foreign key
- [ ] 5.2 Run pipeline and validate output dataset
- [ ] 5.3 Profile data quality metrics

### 6. Build Transaction Data Pipeline
- [ ] 6.1 Create Python transform for transactions dataset
  - [ ] 6.1.1 Standardize timestamp to ISO 8601 format
  - [ ] 6.1.2 Validate amount > 0
  - [ ] 6.1.3 Map currency to ISO 4217 codes
  - [ ] 6.1.4 Map channel to enum (POS, Online, Mobile, ATM)
  - [ ] 6.1.5 Join with accounts and merchants to validate foreign keys
- [ ] 6.2 Run pipeline and validate output dataset
- [ ] 6.3 Profile data quality metrics

### 7. Build Device Data Pipeline
- [ ] 7.1 Create Python transform for devices dataset
  - [ ] 7.1.1 Standardize first_seen to ISO 8601 format
  - [ ] 7.1.2 Normalize fraud_flag to boolean (TRUE/FALSE)
  - [ ] 7.1.3 Validate IP address format
  - [ ] 7.1.4 Map device_type to enum (iOS, Android, Windows, Mac)
  - [ ] 7.1.5 Join with customers to validate customer_id foreign key
- [ ] 7.2 Run pipeline and validate output dataset
- [ ] 7.3 Profile data quality metrics

### 8. Build Merchant Data Pipeline
- [ ] 8.1 Create Python transform for merchants dataset
  - [ ] 8.1.1 Validate MCC codes are 4-digit strings
  - [ ] 8.1.2 Normalize country codes to ISO 3166-1 alpha-2
  - [ ] 8.1.3 Add MCC description lookup (e.g., 4829 = "Money Transfer")
- [ ] 8.2 Run pipeline and validate output dataset
- [ ] 8.3 Profile data quality metrics

### 9. Create External Risk Data
- [ ] 9.1 Create sanctions_list dataset (OFAC watchlist simulation)
- [ ] 9.2 Create high_risk_countries dataset (FATF list: KP, RU, etc.)
- [ ] 9.3 Create high_risk_mcc_codes dataset (money transfer, gambling, etc.)

## Phase 2: Ontology Configuration

### 10. Define Customer Object Type
- [ ] 10.1 Create Customer object in Ontology Manager
- [ ] 10.2 Configure primary key: customer_id
- [ ] 10.3 Add properties: full_name, email, phone, risk_rating, kyc_status, country, onboarding_date
- [ ] 10.4 Link to customers_clean dataset
- [ ] 10.5 Add computed property: is_pep (check against PEP list)
- [ ] 10.6 Add computed property: is_sanctioned (check against sanctions list)
- [ ] 10.7 Add computed property: total_accounts (count linked accounts)
- [ ] 10.8 Add computed property: total_devices (count linked devices)

### 11. Define Account Object Type
- [ ] 11.1 Create Account object in Ontology Manager
- [ ] 11.2 Configure primary key: account_id
- [ ] 11.3 Add properties: account_type, open_date, status
- [ ] 11.4 Link to accounts_clean dataset
- [ ] 11.5 Add computed property: transaction_count (count linked transactions)
- [ ] 11.6 Add computed property: high_risk_transaction_count (count where risk_score > 70)

### 12. Define Transaction Object Type
- [ ] 12.1 Create Transaction object in Ontology Manager
- [ ] 12.2 Configure primary key: transaction_id
- [ ] 12.3 Add properties: amount, currency, timestamp, country, channel
- [ ] 12.4 Link to transactions_clean dataset
- [ ] 12.5 Add computed property: is_high_risk_country (check against high_risk_countries)
- [ ] 12.6 Add computed property: is_cross_border (compare transaction.country vs customer.country)

### 13. Define Device Object Type
- [ ] 13.1 Create Device object in Ontology Manager
- [ ] 13.2 Configure primary key: device_id
- [ ] 13.3 Add properties: ip_address, device_type, first_seen, fraud_flag
- [ ] 13.4 Link to devices_clean dataset
- [ ] 13.5 Add computed property: account_count (count distinct linked accounts)
- [ ] 13.6 Add computed property: is_shared_device (account_count > 3)

### 14. Define Merchant Object Type
- [ ] 14.1 Create Merchant object in Ontology Manager
- [ ] 14.2 Configure primary key: merchant_id
- [ ] 14.3 Add properties: merchant_name, mcc, country
- [ ] 14.4 Link to merchants_clean dataset
- [ ] 14.5 Add computed property: mcc_description (lookup from reference data)
- [ ] 14.6 Add computed property: is_high_risk_mcc (check against high_risk_mcc_codes)
- [ ] 14.7 Add computed property: is_high_risk_country (check against high_risk_countries)
- [ ] 14.8 Add computed property: transaction_count (count linked transactions)

### 15. Define Alert Object Type
- [ ] 15.1 Create Alert object in Ontology Manager
- [ ] 15.2 Configure primary key: alert_id (auto-generated)
- [ ] 15.3 Add properties: alert_type, severity, created_date, status, explanation
- [ ] 15.4 Create backing dataset for alerts

### 16. Define Case Object Type
- [ ] 16.1 Create Case object in Ontology Manager
- [ ] 16.2 Configure primary key: case_id (auto-generated)
- [ ] 16.3 Add properties: case_type, status, priority, assigned_to, created_date, resolved_date, resolution_notes
- [ ] 16.4 Create backing dataset for cases

### 17. Configure Ontology Relationships
- [ ] 17.1 Create "has account" link: Customer → Account (one-to-many)
- [ ] 17.2 Create "executed transaction" link: Account → Transaction (one-to-many)
- [ ] 17.3 Create "paid to merchant" link: Transaction → Merchant (many-to-one)
- [ ] 17.4 Create "uses device" link: Customer → Device (many-to-many)
- [ ] 17.5 Create "triggered alert" link: Transaction → Alert (one-to-many)
- [ ] 17.6 Create "escalated to case" link: Alert → Case (many-to-one)
- [ ] 17.7 Create "has card" link: Account → Card (one-to-many)

## Phase 3: Risk Scoring & Explainability

### 18. Develop Risk Scoring Function
- [ ] 18.1 Create Foundry Function: calculate_risk_score
- [ ] 18.2 Implement Factor 1: Unusual transaction size (0-30 points)
  - [ ] 18.2.1 Calculate account average transaction amount
  - [ ] 18.2.2 Flag if current transaction > 3x average
- [ ] 18.3 Implement Factor 2: High-risk geography (0-25 points)
  - [ ] 18.3.1 Check if transaction.country in high_risk_countries
- [ ] 18.4 Implement Factor 3: Device fraud history (0-25 points)
  - [ ] 18.4.1 Check if linked device has fraud_flag = True
- [ ] 18.5 Implement Factor 4: Cross-border activity (0-10 points)
  - [ ] 18.5.1 Compare transaction.country vs customer.country
- [ ] 18.6 Implement Factor 5: High-risk merchant (0-10 points)
  - [ ] 18.6.1 Check if merchant.is_high_risk_mcc = True
- [ ] 18.7 Return risk_score, fraud_probability, and risk_factors array
- [ ] 18.8 Attach function to Transaction object

### 19. Create Explainability Framework
- [ ] 19.1 Add risk_score property to Transaction object (function-backed)
- [ ] 19.2 Add fraud_probability property to Transaction object (function-backed)
- [ ] 19.3 Add risk_factors property to Transaction object (function-backed)
- [ ] 19.4 Test function on sample transactions
- [ ] 19.5 Validate explanations are human-readable

### 20. Generate Sample Alerts
- [ ] 20.1 Create pipeline to auto-generate alerts for transactions with risk_score > 70
- [ ] 20.2 Populate alert_type based on primary risk factor
- [ ] 20.3 Set severity based on risk_score thresholds (70-79=Medium, 80-89=High, 90+=Critical)
- [ ] 20.4 Generate explanation text from risk_factors
- [ ] 20.5 Run pipeline to create initial alert dataset

## Phase 4: Workshop UI Development

### 21. Build Investigation Dashboard
- [ ] 21.1 Create new Workshop application: "Fraud Investigation Console"
- [ ] 21.2 Add Object Set: High-Risk Transactions (filter: risk_score > 70)
- [ ] 21.3 Add Object Table widget showing transaction details
- [ ] 21.4 Add filters: date range, risk score, country, channel
- [ ] 21.5 Add sorting by risk_score descending
- [ ] 21.6 Configure click-through to transaction detail view

### 22. Build Transaction Detail View
- [ ] 22.1 Create detail page for Transaction object
- [ ] 22.2 Add property panel: amount, timestamp, country, channel
- [ ] 22.3 Add risk score visualization (gauge or progress bar)
- [ ] 22.4 Add risk factors explanation panel
- [ ] 22.5 Add related entities section: Customer, Account, Merchant, Device
- [ ] 22.6 Add action buttons: Create Alert, Escalate to Case

### 23. Build Network Graph Visualization
- [ ] 23.1 Add Graph widget to transaction detail view
- [ ] 23.2 Configure starting node: selected Transaction
- [ ] 23.3 Add expansion rules:
  - [ ] 23.3.1 Transaction → Account → Customer
  - [ ] 23.3.2 Customer → Devices
  - [ ] 23.3.3 Customer → Other Accounts
  - [ ] 23.3.4 Other Accounts → Other Transactions
  - [ ] 23.3.5 Transaction → Merchant
- [ ] 23.4 Configure node styling: color by risk level, size by transaction amount
- [ ] 23.5 Configure edge styling: thickness by transaction frequency
- [ ] 23.6 Add legend explaining node colors and sizes

### 24. Build Case Management Interface
- [ ] 24.1 Create Case List view (Object Set: all Cases)
- [ ] 24.2 Add filters: status, priority, assigned_to
- [ ] 24.3 Create Case Detail view
- [ ] 24.4 Add linked alerts panel
- [ ] 24.5 Add investigation timeline (audit log)
- [ ] 24.6 Add notes section for investigator comments
- [ ] 24.7 Add action buttons: Close Case, Report to Compliance

### 25. Build Customer Risk Profile View
- [ ] 25.1 Create detail page for Customer object
- [ ] 25.2 Add KYC status and risk rating prominently
- [ ] 25.3 Add sanctions/PEP flags if applicable
- [ ] 25.4 Add linked accounts summary
- [ ] 25.5 Add linked devices summary
- [ ] 25.6 Add transaction history table
- [ ] 25.7 Add alerts history table

## Phase 5: Actions & Workflows

### 26. Implement "Freeze Account" Action
- [ ] 26.1 Create Action in Ontology Manager
- [ ] 26.2 Configure trigger: Account object
- [ ] 26.3 Add parameter: reason (string, required)
- [ ] 26.4 Implement logic: Update Account.status = "Frozen"
- [ ] 26.5 Add audit logging: user, timestamp, reason
- [ ] 26.6 Test action on sample account

### 27. Implement "Escalate to Investigation" Action
- [ ] 27.1 Create Action in Ontology Manager
- [ ] 27.2 Configure trigger: Alert object
- [ ] 27.3 Add parameters: priority (enum), assigned_to (string)
- [ ] 27.4 Implement logic: Create new Case object
- [ ] 27.5 Link Alert to newly created Case
- [ ] 27.6 Add audit logging
- [ ] 27.7 Test action on sample alert

### 28. Implement "Request Additional KYC" Action
- [ ] 28.1 Create Action in Ontology Manager
- [ ] 28.2 Configure trigger: Customer object
- [ ] 28.3 Add parameter: kyc_documents_required (multi-select)
- [ ] 28.4 Implement logic: Update Customer.kyc_status = "Pending"
- [ ] 28.5 Generate KYC request form (PDF or structured data)
- [ ] 28.6 Add audit logging
- [ ] 28.7 Test action on sample customer

### 29. Implement "Close Case" Action
- [ ] 29.1 Create Action in Ontology Manager
- [ ] 29.2 Configure trigger: Case object
- [ ] 29.3 Add parameters: outcome (enum: Fraud Confirmed, False Positive, Insufficient Evidence), resolution_notes (string)
- [ ] 29.4 Implement logic: Update Case.status = "Closed", set resolved_date
- [ ] 29.5 Add audit logging
- [ ] 29.6 Test action on sample case

### 30. Implement "Report to Compliance" Action
- [ ] 30.1 Create Action in Ontology Manager
- [ ] 30.2 Configure trigger: Case object
- [ ] 30.3 Implement logic: Generate compliance report
- [ ] 30.4 Include in report: case details, linked alerts, transactions, customers, risk factors, investigation timeline
- [ ] 30.5 Add data lineage section showing source systems
- [ ] 30.6 Export as PDF
- [ ] 30.7 Add audit logging
- [ ] 30.8 Test action on sample case

## Phase 6: AI Investigation Assistant (AIP)

### 31. Set Up AIP Integration
- [ ] 31.1 Enable AIP in Foundry project
- [ ] 31.2 Configure ontology context for AIP
- [ ] 31.3 Grant AIP access to all object types
- [ ] 31.4 Configure data grounding settings

### 32. Configure Risk Summarization Query
- [ ] 32.1 Create AIP query template: "Summarize why this customer is high risk"
- [ ] 32.2 Configure context: Customer object + linked Devices, Transactions, Alerts
- [ ] 32.3 Define output format: bullet points with data citations
- [ ] 32.4 Test query on high-risk customer sample
- [ ] 32.5 Validate response accuracy and grounding

### 33. Configure Pattern Matching Query
- [ ] 33.1 Create AIP query template: "What similar cases exist?"
- [ ] 33.2 Configure context: Current Case + historical Cases
- [ ] 33.3 Define similarity criteria: customer risk profile, transaction patterns, alert types
- [ ] 33.4 Configure output: top 5 similar cases with comparison
- [ ] 33.5 Test query on sample case
- [ ] 33.6 Validate response relevance

### 34. Configure Report Generation Query
- [ ] 34.1 Create AIP query template: "Draft a compliance summary report"
- [ ] 34.2 Configure context: Case object + all linked entities
- [ ] 34.3 Define report structure: executive summary, key facts, risk factors, investigation timeline, recommendation
- [ ] 34.4 Test query on sample case
- [ ] 34.5 Validate report completeness and accuracy

### 35. Build AIP Chat Interface in Workshop
- [ ] 35.1 Add AIP Chat widget to Case Detail view
- [ ] 35.2 Configure pre-defined query buttons for common questions
- [ ] 35.3 Enable free-form text input for custom queries
- [ ] 35.4 Display responses with data citations and links
- [ ] 35.5 Test end-to-end chat experience

## Phase 7: Demo Preparation & Testing

### 36. Prepare Demo Scenario
- [ ] 36.1 Identify 3-5 high-risk transactions for demo walkthrough
- [ ] 36.2 Ensure demo transactions have clear fraud patterns (shared devices, high-risk countries, etc.)
- [ ] 36.3 Pre-generate alerts and cases for demo transactions
- [ ] 36.4 Prepare demo script with narrative for each step

### 37. Test Demo Flow - Step 1: Disconnected Data
- [ ] 37.1 Prepare slide or view showing 6 separate CSV files
- [ ] 37.2 Practice narrative: "Data is siloed, fraud signals are invisible"
- [ ] 37.3 Time this section (target: 2-3 minutes)

### 38. Test Demo Flow - Step 2: Build Ontology
- [ ] 38.1 Navigate to Ontology Manager
- [ ] 38.2 Show 7 object types and 8 relationships
- [ ] 38.3 Practice narrative: "This is a living network of risk relationships"
- [ ] 38.4 Time this section (target: 3-4 minutes)

### 39. Test Demo Flow - Step 3: Network Visualization
- [ ] 39.1 Open Workshop to high-risk transaction
- [ ] 39.2 Expand graph to show Customer → Devices → Other Accounts → Merchant
- [ ] 39.3 Highlight fraud pattern (e.g., shared device across multiple accounts)
- [ ] 39.4 Practice narrative: "Fraud is rarely isolated. It is relational."
- [ ] 39.5 Time this section (target: 4-5 minutes)

### 40. Test Demo Flow - Step 4: Risk Scoring
- [ ] 40.1 Click on flagged transaction
- [ ] 40.2 Show risk score breakdown (e.g., 85/100)
- [ ] 40.3 Display risk factors: unusual size, high-risk geography, device fraud history
- [ ] 40.4 Practice narrative: "We are not just predicting fraud — we are explaining it."
- [ ] 40.5 Time this section (target: 3-4 minutes)

### 41. Test Demo Flow - Step 5: Case Management
- [ ] 41.1 Click "Escalate to Investigation" action
- [ ] 41.2 Create new case, assign to investigator
- [ ] 41.3 Show audit trail
- [ ] 41.4 Execute "Freeze Account" action
- [ ] 41.5 Practice narrative: "Foundry is not just analytics — it is operational risk management."
- [ ] 41.6 Time this section (target: 4-5 minutes)

### 42. Test Demo Flow - Step 6: AI Assistant
- [ ] 42.1 Open AIP chat in Case Detail view
- [ ] 42.2 Ask: "Summarize why this customer is high risk"
- [ ] 42.3 Show AI response with data citations
- [ ] 42.4 Ask: "What similar cases exist?"
- [ ] 42.5 Show similar case matches
- [ ] 42.6 Ask: "Draft a compliance summary report"
- [ ] 42.7 Show generated report
- [ ] 42.8 Practice narrative: "AI is grounded in governed, auditable enterprise data."
- [ ] 42.9 Time this section (target: 5-6 minutes)

### 43. End-to-End Demo Rehearsal
- [ ] 43.1 Run full demo from Step 1 to Step 6
- [ ] 43.2 Time total duration (target: 20-30 minutes)
- [ ] 43.3 Identify any technical issues or slow sections
- [ ] 43.4 Refine narrative and transitions
- [ ] 43.5 Prepare for common audience questions

### 44. Prepare Demo Backup Plan
- [ ] 44.1 Take screenshots of each demo step
- [ ] 44.2 Record video walkthrough as backup
- [ ] 44.3 Prepare static slides if live demo fails
- [ ] 44.4 Test demo on different network/browser

### 45. Create Demo Documentation
- [ ] 45.1 Write demo script with talking points
- [ ] 45.2 Create FAQ document for common questions
- [ ] 45.3 Prepare technical architecture diagram
- [ ] 45.4 Create one-pager summarizing demo value proposition

## Phase 8: Final Validation & Delivery

### 46. Validate Data Quality
- [ ] 46.1 Run data profiling on all clean datasets
- [ ] 46.2 Verify no null values in critical fields
- [ ] 46.3 Confirm all foreign key relationships are valid
- [ ] 46.4 Check for duplicate records

### 47. Validate Ontology Configuration
- [ ] 47.1 Test all computed properties return expected values
- [ ] 47.2 Verify all relationships are bidirectional
- [ ] 47.3 Confirm link cardinality constraints
- [ ] 47.4 Test object search and filtering

### 48. Validate Risk Scoring
- [ ] 48.1 Test risk scoring function on 10+ sample transactions
- [ ] 48.2 Verify risk factors are accurate and explainable
- [ ] 48.3 Confirm fraud probability aligns with risk score
- [ ] 48.4 Test edge cases (e.g., transaction with no risk factors)

### 49. Validate Actions
- [ ] 49.1 Test each action executes successfully
- [ ] 49.2 Verify audit logs capture all required information
- [ ] 49.3 Confirm actions update object properties correctly
- [ ] 49.4 Test action permissions and access controls

### 50. Validate AIP Responses
- [ ] 50.1 Test all pre-configured query templates
- [ ] 50.2 Verify AI responses are grounded in ontology data
- [ ] 50.3 Confirm data citations link to correct objects
- [ ] 50.4 Test free-form queries for accuracy

### 51. Performance Testing
- [ ] 51.1 Measure Workshop page load times (target: <3 seconds)
- [ ] 51.2 Measure graph rendering time (target: <3 seconds)
- [ ] 51.3 Measure risk scoring calculation time (target: <2 seconds)
- [ ] 51.4 Measure AIP response time (target: <5 seconds)

### 52. Final Demo Delivery
- [ ] 52.1 Conduct final rehearsal with stakeholders
- [ ] 52.2 Gather feedback and make final adjustments
- [ ] 52.3 Prepare demo environment for presentation
- [ ] 52.4 Deliver demo to target audience
- [ ] 52.5 Collect feedback and lessons learned
