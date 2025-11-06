---
title: "BC Telemetry Buddy MCP Integration"
domain: "johan-debug"
bc_versions: "14+"
difficulty: "intermediate"
tags: ["telemetry", "performance", "mcp-integration", "kql", "troubleshooting", "monitoring"]
related_topics:
  - "../performance/performance-profiling.md"
  - "../performance/database-optimization.md"
applies_to:
  - "Business Central Server"
  - "Application Insights"
  - "Production Monitoring"
last_updated: "2025-10-27"
conditional_mcp: "bc-telemetry-buddy"
---

# BC Telemetry Buddy MCP Integration

## Overview

When **BC Telemetry Buddy MCP** (`bc-telemetry-buddy`) is available in your environment, you gain access to **actual telemetry data** from Application Insights. This transforms performance troubleshooting from theoretical guidance to **data-driven analysis with real metrics**.

**Game-Changer**: Instead of saying "check for slow queries," Johan can now say "I found 47 database calls taking over 2 seconds - here are the specific targets and operations."

## When BC Telemetry Buddy is Available

### **Detection**

Check if BC Telemetry Buddy is in the available MCPs list:
```
get_workspace_info()
// Returns: { available_mcps: ["bc-telemetry-buddy", ...] }
```

### **What BC Telemetry Buddy Provides**

**11 MCP Tools** for comprehensive telemetry analysis:

**Discovery Tools** (Find What to Query):
- `bctb_get_event_catalog` - List available BC events with descriptions, frequency
- `bctb_get_event_field_samples` - Analyze customDimensions structure for events
- `bctb_get_event_schema` - Get detailed schema for specific event IDs
- `bctb_get_categories` - List event categories (Performance, Error, Security, etc.)
- `bctb_get_tenant_mapping` - Map company names to Azure tenant IDs

**Query Execution**:
- `bctb_query_telemetry` - Execute KQL queries against Application Insights

**Query Library**:
- `bctb_get_saved_queries` - List saved queries with tag filtering
- `bctb_search_queries` - Search queries by keywords
- `bctb_save_query` - Save successful queries for reuse

**Context**:
- `bctb_get_external_queries` - Fetch KQL examples from configured sources

**Analysis**:
- `bctb_get_recommendations` - Get actionable insights from query results

## Data-Driven Performance Analysis

### **The Discovery-First Workflow** ⭐

BC Telemetry Buddy v1.0.0+ uses a systematic discovery approach:

**Step 1: Discover Available Events**
```markdown
**Johan:** "Let me check what performance events are available in your telemetry..."

[Use bctb_get_event_catalog with category="performance"]

**Result:**
- RT0005: Long Running SQL Queries (847 occurrences/day)
- RT0006: Long Running AL Operations (234 occurrences/day)
- RT0012: Database Lock Timeouts (12 occurrences/day)
- RT0020: Report Generation Performance (156 occurrences/day)

"I see you have active performance issues - 847 slow SQL queries per day!"
```

**Step 2: Analyze Event Structure**
```markdown
**Johan:** "Let me analyze the structure of the Long Running SQL Query events..."

[Use bctb_get_event_field_samples with eventId="RT0005"]

**Result - Custom Dimensions Available:**
- executionTimeInMs (number, 100% prevalence) - samples: [2543, 3821, 15234]
- sqlStatement (string, 95% prevalence) - truncated SQL
- alObjectType (string, 100%) - "Table", "Page", "Report"
- alObjectId (string, 100%) - object IDs
- companyName (string, 98%)
- aadTenantId (string, 100%)

"Perfect! I can now query by execution time, object type, and SQL statement."
```

**Step 3: Execute Targeted Query**
```markdown
**Johan:** "Let me find the slowest queries from the last 24 hours..."

[Use bctb_query_telemetry with discovered fields]

**KQL Query Generated:**
```kql
traces
| where timestamp > ago(24h)
| where customDimensions.eventId == "RT0005"
| extend executionTime = todouble(customDimensions.executionTimeInMs)
| extend sqlStatement = tostring(customDimensions.sqlStatement)
| extend objectType = tostring(customDimensions.alObjectType)
| extend objectId = tostring(customDimensions.alObjectId)
| where executionTime > 2000
| summarize 
    count(), 
    avg(executionTime),
    max(executionTime)
  by sqlStatement, objectType, objectId
| order by avg_executionTime desc
| take 20
```

**Results:**
| SQL Statement | Object Type | Object ID | Count | Avg Time | Max Time |
|---------------|-------------|-----------|-------|----------|----------|
| SELECT * FROM "Customer" WHERE... | Table | 18 | 47 | 3245ms | 5821ms |
| SELECT "Item"."No_" FROM "Item"... | Page | 31 | 23 | 2876ms | 4234ms |
```

**Step 4: Provide Data-Driven Recommendations**
```markdown
**Johan:** "I found **47 slow queries** to the Customer table averaging 3.2 seconds!

**Root Cause:** Full table scan without proper indexing

**Recommendation:**
1. Add index on Customer table for the filter field
2. Use SETCURRENTKEY before filters
3. Consider SETRANGE instead of SETFILTER for better SIFT utilization

**Code Fix:**
```al
// ❌ Before (causing full scan)
Customer.SETFILTER("Credit Limit", '>%1', MinCredit);

// ✅ After (uses index)
Customer.SETCURRENTKEY("Credit Limit");
Customer.SETRANGE("Credit Limit", MinCredit, 999999);
```

**Impact:** Should reduce query time from 3.2s to <100ms based on similar optimizations.

Want me to save this query for future monitoring?
```
```

### **Customer-Specific Analysis**

```markdown
**User:** "Show me errors for customer Contoso in the last week"

**Johan:** "Let me map Contoso to their tenant ID and analyze..."

[Use bctb_get_tenant_mapping]
**Result:** Contoso = aadTenantId: abc123-...

[Use bctb_query_telemetry with tenant filter]

**KQL:**
```kql
traces
| where timestamp > ago(7d)
| where customDimensions.eventId startswith "RT00" // Error events
| extend tenantId = tostring(customDimensions.aadTenantId)
| where tenantId == "abc123-..."
| summarize count() by 
    tostring(customDimensions.eventId),
    tostring(customDimensions.alObjectType),
    tostring(customDimensions.message)
| order by count_ desc
```

**Findings:**
"Contoso is experiencing 23 RT0010 errors (permission denied) when accessing Sales Order pages. This is a security configuration issue, not a code bug."
```

## Performance Troubleshooting Workflows

### **Workflow 1: "My Page is Slow"**

**Without Telemetry Buddy:**
```markdown
**Johan:** "Possible causes of slow pages:
- Database queries without proper keys
- Excessive triggers firing
- Heavy calculations in OnAfterGetRecord
- Too many FlowFields

Check your code for these patterns..."
```

**With Telemetry Buddy:** 
```markdown
**Johan:** "Let me check the actual telemetry for this page..."

[Use bctb_get_event_catalog to find page events]
[Use bctb_query_telemetry for specific page ID]

**Real Data:**
"I found the issue! Page 50100 (Customer Priority List) is loading:
- 3,247ms average load time
- 89 database calls per page load
- FlowField 'Total Sales' calculating for every row

**Root Cause:** FlowField in repeater calculated OnAfterGetRecord for 200 rows

**Fix:** Set AutoCalcField = false, calculate only on drill-down

**Before/After:** Page load will drop from 3.2s to <200ms based on eliminating unnecessary FlowField calculations."
```

### **Workflow 2: "Find Performance Opportunities"**

**Proactive Analysis:**
```markdown
**Johan:** "Let me scan your telemetry for performance refactoring opportunities..."

[Use bctb_get_event_catalog with category="performance"]
[Use bctb_query_telemetry to find patterns]

**Findings:**
1. **Database Locks** - 12 lock timeout events/day on Item table
   - Cause: Long-running batch job holding locks
   - Fix: Commit more frequently, use smaller batches

2. **Report Timeouts** - Sales Analysis report times out 45% of the time
   - Cause: Querying 5 years of data without date filter
   - Fix: Add mandatory date filter, default to current year

3. **API Performance** - Web service calls averaging 1.2s response time
   - Cause: No query result caching
   - Fix: Implement caching with 5-minute TTL

**Impact:** These three fixes will improve user experience significantly with minimal code changes.
```

### **Workflow 3: "Production Issue Investigation"**

**Real-Time Investigation:**
```markdown
**User:** "Users are reporting errors in Production"

**Johan:** "Let me check the latest error telemetry..."

[Use bctb_get_event_catalog with status="error", last 1 hour]
[Use bctb_query_telemetry for error details]

**Live Data:**
"I see 47 RT0010 errors in the last hour - all pointing to Codeunit 80 (Sales-Post).

Error Message: 'Dimension Value Code must have a value in Dimension Set Entry'

**Pattern:** Only happens for Customer 'C-10000', only on Sales Orders > $50,000

**Root Cause:** Missing dimension setup for high-value sales validation

**Immediate Fix:** Add dimension code for high-value validation rule
**Long-term Fix:** Add validation to prevent orders without required dimensions

This is not a code bug - it's a data configuration issue. Let me show you how to fix it..."
```

## Query Library and Context Building

### **Saving Queries**

```markdown
**Johan:** "This query is useful for ongoing monitoring. Let me save it..."

[Use bctb_save_query]

**Saved:** `.vscode/bctb/queries/Performance/slow-database-operations.kql`

"I've saved this query. Now Telemetry Buddy will use it as context for future similar questions, and you can run it manually via CodeLens in VS Code."
```

### **Discovering Existing Patterns**

```markdown
**User:** "How do I query for failed web service calls?"

**Johan:** "Let me check if we have saved queries for this pattern..."

[Use bctb_search_queries with keywords="web service, failed, api"]

**Found:** `api-errors.kql` and `web-service-performance.kql`

"Perfect! We have existing queries for this. Here's the pattern we've used before..."

[Show and adapt existing query]
```

## External Context Integration

### **BCTech Samples**

BC Telemetry Buddy can pull KQL examples from external sources:

```markdown
**Johan:** "Let me check Microsoft's BCTech repository for telemetry samples..."

[Use bctb_get_external_queries]

**Found:**
- Long Running SQL Queries (from BCTech samples)
- Database Lock Analysis (from BCTech)  
- Report Performance Analysis (from community)

"I'll adapt Microsoft's official sample for your specific event structure..."
```

## Recommendations and Actionable Insights

### **Data-Driven Recommendations**

```markdown
[After executing query, use bctb_get_recommendations]

**Telemetry Buddy Recommendations:**
1. **Index Optimization** - 3 tables showing full scan patterns
2. **Batch Size Reduction** - Posting routines holding locks > 30 seconds
3. **Caching Opportunity** - Repeated identical queries (847 times/day)
4. **FlowField Optimization** - 12 FlowFields calculated unnecessarily

**Priority:** Fix database locks first (immediate user impact), then optimize FlowFields (biggest performance gain).
```

## When BC Telemetry Buddy is NOT Available

### **Fallback Strategy**

If `bc-telemetry-buddy` is NOT in `available_mcps`:

✅ **Johan provides**:
- Theoretical performance guidance
- Code review for common anti-patterns
- Best practices for optimization
- Profiler usage instructions

✅ **Johan's Fallback Response**:
```markdown
"For performance analysis, I can review your code for common performance issues:
- SETCURRENTKEY usage
- FlowField optimization
- Database query patterns
- Trigger efficiency

However, for **data-driven analysis** of actual production performance, I recommend installing BC Telemetry Buddy MCP. It provides:
- Real telemetry data from Application Insights
- KQL query generation with discovery tools
- Performance metrics and patterns
- Customer-specific issue tracking

[Installation: https://alguidelines.dev/docs/agentic-coding/communityresources/tools/waldo-bctelemetrybuddy/]

Without telemetry, I can only provide theoretical guidance. With Telemetry Buddy, I can show you **actual slow queries, real error rates, and measured performance metrics**."
```

### **Recommendation Pattern**

When user mentions production issues:
```markdown
**Johan:** "For production issue investigation, BC Telemetry Buddy MCP would be incredibly valuable!

**What you'd get:**
- Real error data from Application Insights
- Actual query execution times
- Customer-specific issue patterns
- Historical performance trends
- Saved query library for monitoring

**Setup is simple:**
1. Configure Application Insights access
2. Install BC Telemetry Buddy extension
3. Add to available_mcps when calling set_workspace_info

Once installed, I can investigate issues with **real data** instead of guessing."
```

## Best Practices for Telemetry-Driven Analysis

1. **Discovery First**: Always start with event catalog before querying
2. **Understand Schema**: Check field samples to know what's queryable
3. **Save Useful Queries**: Build a library for recurring analysis
4. **Customer Context**: Use tenant mapping for multi-tenant environments
5. **Data-Driven Recommendations**: Base fixes on actual metrics, not assumptions
6. **Monitor Trends**: Save and re-run queries to track improvements

## Integration with Other Specialists

### **Handoff to Alex Architect**

```markdown
**Johan:** "This performance issue requires architectural changes..."

**Findings from Telemetry:**
- 847 slow queries/day to Sales Header table
- Pattern: Filtering on non-indexed fields
- Impact: 3.2s average, 5.8s max

**Handoff to Alex:**
"@alex-architect - We have a systemic performance issue requiring table redesign. Telemetry shows heavy filtering on non-indexed 'Priority Code' field. Recommend adding table extension with optimized indexes. Here's the telemetry data..."
```

### **Handoff to Quinn (Testing)**

```markdown
**Johan:** "Let me check error patterns to help Quinn design test cases..."

[Query telemetry for error event types and patterns]

**Findings:**
- Dimension validation errors: 45% of all errors
- Permission errors: 30% of errors
- Batch posting errors: 15% of errors

**Handoff to Quinn:**
"@quinn-tester - Here are the top error patterns from production telemetry. These should be your priority test scenarios. I've saved the queries for ongoing monitoring."
```

## Advanced: Custom Event Analysis

### **Partner Extension Events**

```markdown
**Johan:** "Let me check if your custom extension is logging telemetry..."

[Use bctb_get_event_catalog with includeCustomEvents=true]

**Found Custom Events:**
- CUST0001: Sales Priority Calculated (custom telemetry)
- CUST0002: Priority Threshold Exceeded (custom event)

"Great! Your extension is logging custom telemetry. Let me analyze the priority calculation performance..."
```

## See Also

- [Performance Profiling](../performance/performance-profiling.md) - Manual profiling techniques
- [Database Optimization](../performance/database-optimization.md) - Query optimization patterns
- [BC Telemetry Buddy Documentation](https://github.com/waldo1001/waldo.BCTelemetryBuddy/blob/main/docs/UserGuide.md)
- [BCTech Telemetry Samples](https://github.com/microsoft/BCTech/tree/master/samples/AppInsights)
