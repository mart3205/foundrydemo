# Palantir Foundry Fraud Demo - Specification Package

## Overview

This specification package contains two versions of the demo design:

1. **Production-Grade Version** (Original) - Comprehensive enterprise solution
2. **Demo-Optimized Version** (Simplified) - Maximum impact, minimum build effort

## Which Version Should You Use?

### Use the **SIMPLIFIED VERSION** if:
- ✅ You need to build a demo quickly (8-13 days)
- ✅ Your goal is to demonstrate Foundry's unique value, not build production software
- ✅ You want to focus on ONE compelling fraud story (The Golden Path)
- ✅ You have limited resources or tight timeline
- ✅ You're doing a proof-of-concept or sales demo

**Files:**
- `requirements-simplified.md`
- `design-simplified.md`
- `tasks-simplified.md`

**Build Time:** 8-13 days
**Tasks:** 22 main tasks
**Objects:** 5 (Customer, Account, Transaction, Device, Case)
**Actions:** 2 (Escalate to Case, Freeze Account)
**AI Queries:** 2 (Summarize risk, Draft report)

### Use the **PRODUCTION-GRADE VERSION** if:
- ✅ You're building a real pilot or MVP for a client
- ✅ You need to demonstrate full enterprise architecture thinking
- ✅ You have 4-6 weeks for implementation
- ✅ You want to show comprehensive governance, lineage, and audit capabilities
- ✅ You're positioning for a large enterprise deal

**Files:**
- `requirements.md`
- `design.md`
- `tasks.md`

**Build Time:** 4-6 weeks
**Tasks:** 52 main tasks
**Objects:** 7 (Customer, Account, Transaction, Device, Merchant, Alert, Case)
**Actions:** 5 (Freeze Account, Escalate, Request KYC, Close Case, Report to Compliance)
**AI Queries:** 3 (Summarize risk, Find similar cases, Draft report)

## Key Differences

| Aspect | Simplified (Demo) | Production-Grade |
|--------|-------------------|------------------|
| **Build Time** | 8-13 days | 4-6 weeks |
| **Total Tasks** | 22 | 52 |
| **Object Types** | 5 | 7 |
| **Relationships** | 4 | 8 |
| **Actions** | 2 | 5 |
| **AI Queries** | 2 | 3 |
| **Risk Scoring** | 3 simple rules | 5-factor algorithm |
| **Data Pipelines** | 4 minimal | 7 comprehensive |
| **External Data** | None | OFAC, FATF watchlists |
| **Data Quality** | Basic | Full framework |
| **Fraud Patterns** | 1 (shared device) | 4 patterns |
| **Demo Duration** | 15-20 minutes | 30-40 minutes |

## Recommendation

**For most use cases, start with the SIMPLIFIED VERSION.**

Why?
- You can build it in 2 weeks instead of 6 weeks
- It demonstrates the same core value proposition
- You avoid over-engineering for a demo
- You can always expand later if needed

**The Golden Path:**
A suspicious transaction from a high-risk country → linked to a shared device → escalated to investigation → explained by AI.

This single scenario demonstrates:
- ✅ Connected intelligence (ontology graph)
- ✅ Explainable scoring (risk factors)
- ✅ Operational action (escalate case, freeze account)
- ✅ AI grounded in ontology (summarize risk, draft report)

That's all you need for a compelling demo.

## What's Removed in Simplified Version?

To achieve 70% reduction in build effort, we removed:

**Objects:**
- ❌ Cards object
- ❌ Alert object (using risk_score filter instead)

**Actions:**
- ❌ Request Additional KYC
- ❌ Close Case
- ❌ Report to Compliance

**Data & Pipelines:**
- ❌ External watchlist integration (OFAC, FATF)
- ❌ MCC enrichment
- ❌ Complex data quality rules
- ❌ Data profiling dashboards

**Risk Scoring:**
- ❌ 5-factor scoring with behavioral baselines
- ❌ Account average calculations
- ❌ Historical analysis

**AI:**
- ❌ Pattern matching ("What similar cases exist?")
- ❌ Complex similarity search

**Testing:**
- ❌ Performance testing
- ❌ Load testing
- ❌ Security hardening

## What's Kept in Simplified Version?

Everything you need for a compelling demo:

**Core Ontology:**
- ✅ Customer, Account, Transaction, Device, Case (+ Merchant optional)
- ✅ 4 key relationships

**Risk Scoring:**
- ✅ 3 simple, explainable rules (high-risk country, device fraud, large amount)
- ✅ Clear risk_score and risk_factors output

**Network Analysis:**
- ✅ Shared device fraud pattern (1 device → multiple accounts)
- ✅ Graph visualization with clear relationships

**Operational Actions:**
- ✅ Escalate to Case (from Transaction)
- ✅ Freeze Account (from Account)
- ✅ Audit trail

**AI Assistant:**
- ✅ "Summarize why this customer is high risk"
- ✅ "Draft a compliance summary report"
- ✅ Grounded in ontology data

**Demo Flow:**
- ✅ All 6 steps (disconnected data → ontology → network → risk scoring → actions → AI)
- ✅ 15-20 minute narrative
- ✅ Maximum impact

## Getting Started

### If Using Simplified Version:
1. Read `requirements-simplified.md` to understand scope
2. Review `design-simplified.md` for technical approach
3. Execute tasks from `tasks-simplified.md` (22 tasks, 8-13 days)
4. Focus on "The Golden Path" scenario

### If Using Production-Grade Version:
1. Read `requirements.md` to understand full scope
2. Review `design.md` for comprehensive architecture
3. Execute tasks from `tasks.md` (52 tasks, 4-6 weeks)
4. Plan for phased delivery

## Strategic Advice

**From the feedback you received:**

> "The risk is not technical. The risk is over-complexity in live demo."
> "You have 50+ tasks. For live demo, you must focus on 1 high-impact fraud scenario."
> "Remember: Clarity > Completeness"

**The simplified version addresses this by:**
- Reducing tasks from 52 to 22 (58% reduction)
- Reducing build time from 4-6 weeks to 8-13 days (70% reduction)
- Focusing on ONE compelling fraud story (The Golden Path)
- Keeping only what's essential for demo impact

**You can always expand later if the demo leads to a pilot or production engagement.**

## Questions?

If you're unsure which version to use, ask yourself:

1. **What's my timeline?** 
   - <2 weeks → Simplified
   - 4-6 weeks → Production-grade

2. **What's my goal?**
   - Demonstrate value → Simplified
   - Build pilot/MVP → Production-grade

3. **What's my audience?**
   - Executives/Mixed → Simplified (15-20 min demo)
   - Technical deep-dive → Production-grade (30-40 min demo)

4. **What's my risk tolerance?**
   - Need to deliver quickly → Simplified
   - Can afford longer timeline → Production-grade

**When in doubt, start with Simplified. You can always expand later.**

## Next Steps

1. Choose your version (Simplified recommended for most cases)
2. Review the requirements document
3. Review the design document
4. Start executing tasks from the tasks document
5. Focus on building "The Golden Path" scenario
6. Rehearse your demo narrative
7. Deliver with confidence

**Remember: This is a demo, not production. Optimize for impact, not completeness.**

Good luck! 🚀
