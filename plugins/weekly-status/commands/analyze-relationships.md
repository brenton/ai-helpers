---
description: "Analyze Jira issue relationships and dependencies to understand impact chains and risk propagation"
argument-hint: "PROJECTS [--issue KEY] [--depth N]"
---

## Name
weekly-status:analyze-relationships

## Synopsis
```
/weekly-status:analyze-relationships PROJECTS [--issue KEY] [--depth N] [--types TYPES]
```

## Description
Analyzes Jira issue relationships (parent/child, blocking dependencies, depends-on links) to understand impact chains and risk propagation across projects. Provides detailed mapping of how issues connect and affect each other, crucial for understanding the broader implications of status updates and blockers.

This command helps leadership understand dependencies that might not be obvious from individual issue analysis, revealing potential cascading risks and coordination requirements.

## Implementation

### Step 1 — Parse Arguments and Validate
Parse command arguments:
- `PROJECTS`: Comma-separated Jira project keys for analysis scope
- `--issue KEY`: Optional specific issue to analyze (shows its relationship tree)
- `--depth N`: Relationship traversal depth (default: 2, max: 5)
- `--types TYPES`: Relationship types to include (default: all, options: parent-child,blocks,depends)

Validate project keys and issue existence if specified.

### Step 2 — Verify Snowflake Connectivity
Use `setup-snowflake` skill to establish connection and verify access to:
- JIRA_ISSUE table for issue metadata
- JIRA_NODEASSOCIATION table for relationship data
- Required lookup tables (JIRA_ISSUETYPE, JIRA_ISSUESTATUS)

### Step 3 — Collect Relationship Data
Use `relationship-analysis` skill to query JIRA_NODEASSOCIATION:

```sql
-- Collect all relationship types
SELECT 
    parent.ISSUEKEY as SOURCE_KEY,
    child.ISSUEKEY as TARGET_KEY,
    na.ASSOCIATION_TYPE as RELATIONSHIP_TYPE,
    parent.SUMMARY as SOURCE_SUMMARY,
    child.SUMMARY as TARGET_SUMMARY,
    parent.ISSUESTATUS as SOURCE_STATUS,
    child.ISSUESTATUS as TARGET_STATUS,
    parent.PROJECT_KEY as SOURCE_PROJECT,
    child.PROJECT_KEY as TARGET_PROJECT
FROM JIRA_NODEASSOCIATION na
INNER JOIN JIRA_ISSUE parent ON na.SOURCE_NODE_ID = parent.ID
INNER JOIN JIRA_ISSUE child ON na.SINK_NODE_ID = child.ID
WHERE na.ASSOCIATION_TYPE IN ('IssueParentChildLink', 'IssueBlocksLink', 'IssueDependsLink')
  AND (parent.PROJECT_KEY IN (projects) OR child.PROJECT_KEY IN (projects))
```

### Step 4 — Build Relationship Graph
Process collected data to create relationship mappings:
- **Parent/Child Hierarchies**: Epic → Story → Subtask relationships
- **Blocking Chains**: Issue A blocks Issue B blocks Issue C
- **Dependency Networks**: Issue depends on multiple other issues
- **Cross-Project Links**: Dependencies spanning project boundaries

### Step 5 — Impact Analysis
For each relationship type, analyze implications:

**Parent/Child Analysis**:
- Identify orphaned children (missing parents)
- Find blocked hierarchies (parent blocked affects all children)
- Calculate completion percentages for epics/stories

**Blocking Analysis**:
- Find critical blockers (issues blocking multiple others)
- Identify blocking chains and bottlenecks
- Highlight cross-team blocking relationships

**Dependency Analysis**:
- Map dependency networks and potential risks
- Identify circular dependencies
- Find issues with unresolved dependencies

### Step 6 — Generate Analysis Report
Create comprehensive relationship analysis including:

**Executive Summary**:
- Total relationship counts by type
- Critical blockers requiring attention
- Cross-project coordination needs

**Relationship Maps**:
- Visual representation of key relationship chains
- Issue hierarchies with status indicators
- Dependency networks with risk highlights

**Impact Assessment**:
- Issues at risk due to dependencies
- Potential cascading delays
- Coordination requirements between teams

**Action Items**:
- Blockers requiring immediate attention
- Dependencies to monitor
- Cross-team communication needs

### Step 7 — Specific Issue Analysis (if --issue provided)
For targeted issue analysis:
- Show complete relationship tree (parents, children, blockers, dependencies)
- Calculate potential impact radius
- Identify all affected projects and teams
- Provide risk assessment based on relationships

## Examples

**Analyze relationships across key projects:**
```bash
/weekly-status:analyze-relationships DPTP,TRT,ACM --depth 3
```

**Deep-dive specific issue impact:**
```bash
/weekly-status:analyze-relationships TRT --issue TRT-4521 --depth 2
```

**Focus on blocking relationships only:**
```bash
/weekly-status:analyze-relationships DPTP,ART --types blocks --depth 3
```

**Cross-project dependency analysis:**
```bash
/weekly-status:analyze-relationships DPTP,TRT,ACM,ART,SHIPSTRAT --types depends --depth 2
```

## Return Value
Creates relationship analysis report including:
- **Relationship Summary**: Counts and types of discovered relationships
- **Critical Blockers**: Issues blocking multiple others or cross-project work
- **Dependency Networks**: Complex dependency chains and potential risks
- **Risk Assessment**: Impact analysis based on relationship data
- **Coordination Needs**: Cross-team dependencies requiring attention

Output files:
- `.work/weekly-status/relationship_analysis.md`: Human-readable analysis report
- `.work/weekly-status/relationship_graph.json`: Structured relationship data for further analysis

## Arguments
- `PROJECTS`: Comma-separated Jira project keys (required)
- `--issue KEY`: Specific issue key to analyze in detail (optional)
- `--depth N`: Relationship traversal depth, 1-5 (default: 2)
- `--types TYPES`: Relationship types - parent-child,blocks,depends (default: all)
- `--output FILE`: Output file path (default: .work/weekly-status/relationship_analysis.md)

## Error Handling
- No relationships found: Suggest expanding project scope or checking data availability
- Circular dependencies detected: Highlight and recommend resolution
- Missing relationship data: Graceful degradation with available data
- Deep traversal timeout: Reduce depth and provide partial results

## Skills Used
- `setup-snowflake`: Snowflake MCP connection and authentication
- `relationship-analysis`: JIRA_NODEASSOCIATION query and graph building

## Integration with Other Commands
- **draft**: Relationship data enhances impact analysis in reports
- **improve**: Relationship context supports targeted enhancement requests
- **Standalone**: Provides dedicated relationship analysis for complex dependency mapping