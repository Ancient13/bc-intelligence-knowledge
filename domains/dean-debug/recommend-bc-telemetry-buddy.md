---
title: "Performance Analysis Without Telemetry Data"
domain: "johan-debug"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["telemetry", "performance", "tooling", "recommendations"]
conditional_mcp_missing: "bc-telemetry-buddy"
estimated_time: "5 minutes"
last_updated: "2025-10-27"
---

# Performance Analysis Without Telemetry Data

## ⚠️ Current Limitation

I notice you don't have **BC Telemetry Buddy** configured in your MCP environment. While I can provide theoretical performance guidance based on BC best practices, I'm working **without real performance data** from your specific environment.

## What You're Missing

### Data-Driven Performance Insights
Without telemetry access, I cannot provide:

- ✅ **Real query execution times** from your production or test environments
- ✅ **Actual database performance metrics** showing bottlenecks
- ✅ **Customer-specific telemetry analysis** for multi-tenant scenarios
- ✅ **Production event tracking** to identify real-world issues
- ✅ **SQL execution patterns** from actual AL code execution
- ✅ **Page rendering performance** with real timing data
- ✅ **Report generation metrics** showing actual processing times

### Example: What Telemetry Data Shows

**Instead of saying:**
> "Database queries might be slow - consider adding SIFT keys"

**I could say:**
> "I found 47 database calls in your Sales Order page taking over 2 seconds each. The Item Ledger Entry queries are causing 80% of your performance issues. Here are the specific SIFT keys that will eliminate this bottleneck: FlowField X needs [Field1, Field2, Field3]"

## Why BC Telemetry Buddy is Valuable

### Real-Time Performance Analysis
- **Discovery-First Workflow**: Find what events are actually being logged
- **Field Sampling**: See actual data values before querying
- **Targeted KQL Queries**: Query exactly what you need
- **Query Library**: Save and reuse performance queries
- **External Context**: Access BCTech sample queries

### 11 Powerful MCP Tools
1. **Discovery Tools**
   - `bctb_get_event_catalog` - Browse available telemetry events
   - `bctb_get_event_field_samples` - Sample field values before querying
   - `bctb_get_event_schema` - Understand event structure
   - `bctb_get_categories` - Event categorization

2. **Query Tools**
   - `bctb_query_telemetry` - Execute KQL queries against real data
   - `bctb_get_tenant_mapping` - Multi-customer analysis

3. **Library Tools**
   - `bctb_get_saved_queries` - Access saved query patterns
   - `bctb_search_queries` - Find relevant queries
   - `bctb_save_query` - Save useful patterns
   - `bctb_get_external_queries` - BCTech community queries

4. **Analysis Tools**
   - `bctb_get_recommendations` - AI-driven performance recommendations

## Installation Instructions

### 1. Install BC Telemetry Buddy MCP Server

Add to your MCP settings (VS Code: `.vscode/settings.json`):

```json
{
  "mcp.servers": {
    "bc-telemetry-buddy": {
      "command": "npx",
      "args": ["bc-telemetry-buddy"]
    }
  }
}
```

Or for Claude Desktop (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "bc-telemetry-buddy": {
      "command": "npx",
      "args": ["bc-telemetry-buddy"]
    }
  }
}
```

### 2. Configure Telemetry Access

BC Telemetry Buddy needs access to your Application Insights or Kusto query environment. Follow the setup guide at:
https://github.com/waldo1001/waldo.BCTelemetryBuddy

### 3. Update Workspace Configuration

Once installed, let me know it's available:

```json
{
  "tool": "set_workspace_info",
  "arguments": {
    "path": "C:\\Your\\Workspace\\Path",
    "available_mcps": ["bc-telemetry-buddy"]
  }
}
```

**Dynamic Knowledge Loading**: Once you report BC Telemetry Buddy as available, I'll automatically gain access to data-driven analysis capabilities without requiring an MCP server restart!

## Fallback: Theoretical Guidance

Until you have telemetry access, I can still help with:

### Performance Best Practices
- SIFT key design principles
- Query optimization patterns
- Caching strategies
- Background processing approaches

### Code Review Guidance
- Anti-patterns that typically cause performance issues
- FlowField design recommendations
- Table relationship optimization
- Report design best practices

### Proactive Recommendations
- "Based on your code, this pattern *typically* causes issues..."
- "Industry standard is to add SIFT keys for these fields..."
- "Consider testing with large datasets to verify performance..."

## The Difference

### Theoretical (Current)
> "FlowFields can be slow. Consider adding SIFT keys for commonly filtered fields."

### Data-Driven (With Telemetry)
> "Your Customer.Balance FlowField is queried 1,847 times per hour with 94% using Status filter. Adding SIFT key [Status,Posting Date] will reduce execution time from 2.4s to 120ms - a 20x improvement."

## Next Steps

1. **Install BC Telemetry Buddy** following instructions above
2. **Configure telemetry access** to your BC environment
3. **Update workspace info** to report MCP availability
4. **Ask me the same performance question again** - you'll get data-driven insights!

## See Also

- [BC Telemetry Buddy GitHub](https://github.com/waldo1001/waldo.BCTelemetryBuddy)
- [Performance Profiling](./performance-profiling.md) - Theoretical guidance
- [Database Optimization](./database-optimization.md) - Best practices
- [Caching Strategies](./caching-strategies.md) - BC caching patterns
