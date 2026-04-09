---
name: "Jira Issue Relationship Analysis"
description: "Analyze parent/child, blocking, and dependency relationships between Jira issues using JIRA_NODEASSOCIATION data"
---

# Jira Issue Relationship Analysis Skill

## When to Use This Skill

Use this skill when you need to understand how Jira issues are connected through relationships such as:
- **Parent/Child relationships**: Epic → Story → Subtask hierarchies  
- **Blocking relationships**: Issue A blocks Issue B (IssueBlocksLink)
- **Dependency relationships**: Issue A depends on Issue B (IssueDependsLink)
- **Impact analysis**: Understanding cascading effects of delays or blockers
- **Cross-project coordination**: Dependencies spanning multiple teams/projects

This skill extends beyond the basic component associations to provide full relationship mapping for impact assessment.

## Implementation Steps

### Step 1: Discover Available Relationship Types

Query the JIRA_NODEASSOCIATION table to understand what relationship types are available:

```sql
-- Discover available association types
SELECT DISTINCT na.ASSOCIATION_TYPE, COUNT(*) as COUNT
FROM JIRA_NODEASSOCIATION na
INNER JOIN JIRA_ISSUE ji1 ON na.SOURCE_NODE_ID = ji1.ID
INNER JOIN JIRA_ISSUE ji2 ON na.SINK_NODE_ID = ji2.ID
WHERE (ji1.PROJECT_KEY IN ('DPTP', 'TRT', 'ACM', 'ART') 
       OR ji2.PROJECT_KEY IN ('DPTP', 'TRT', 'ACM', 'ART'))
GROUP BY na.ASSOCIATION_TYPE
ORDER BY COUNT DESC;
```

Expected association types:
- `IssueComponent`: Issue-to-component associations (already used in existing code)
- `IssueParentChildLink`: Parent/child relationships (Epic/Story/Subtask)  
- `IssueBlocksLink`: Blocking relationships
- `IssueDependsLink`: Dependency relationships
- Other types may exist depending on Jira configuration

### Step 2: Collect Core Relationship Data

Query all relevant issue relationships with full metadata:

```sql
-- Main relationship collection query
SELECT 
    parent.ISSUEKEY as SOURCE_KEY,
    child.ISSUEKEY as TARGET_KEY,
    na.ASSOCIATION_TYPE as RELATIONSHIP_TYPE,
    parent.SUMMARY as SOURCE_SUMMARY,
    child.SUMMARY as TARGET_SUMMARY,
    parent.PROJECT_KEY as SOURCE_PROJECT,
    child.PROJECT_KEY as TARGET_PROJECT,
    pstatus.PNAME as SOURCE_STATUS,
    cstatus.PNAME as TARGET_STATUS,
    ptype.PNAME as SOURCE_TYPE,
    ctype.PNAME as TARGET_TYPE,
    parent.CREATED as SOURCE_CREATED,
    child.CREATED as TARGET_CREATED,
    parent.UPDATED as SOURCE_UPDATED,
    child.UPDATED as TARGET_UPDATED
FROM JIRA_NODEASSOCIATION na
INNER JOIN JIRA_ISSUE parent ON na.SOURCE_NODE_ID = parent.ID
INNER JOIN JIRA_ISSUE child ON na.SINK_NODE_ID = child.ID
LEFT JOIN JIRA_ISSUESTATUS pstatus ON parent.ISSUESTATUS = pstatus.ID
LEFT JOIN JIRA_ISSUESTATUS cstatus ON child.ISSUESTATUS = cstatus.ID
LEFT JOIN JIRA_ISSUETYPE ptype ON parent.ISSUETYPE = ptype.ID
LEFT JOIN JIRA_ISSUETYPE ctype ON child.ISSUETYPE = ctype.ID
WHERE na.ASSOCIATION_TYPE IN ('IssueParentChildLink', 'IssueBlocksLink', 'IssueDependsLink')
  AND (parent.PROJECT_KEY IN (projects) OR child.PROJECT_KEY IN (projects))
ORDER BY na.ASSOCIATION_TYPE, parent.PROJECT_KEY, parent.ISSUEKEY;
```

### Step 3: Build Relationship Graph Structure

Organize the collected data into a queryable graph structure:

```python
# Example data structure for relationship mapping
relationships = {
    'parent_child': {
        'DPTP-1234': ['DPTP-1235', 'DPTP-1236'],  # Parent -> [Children]
        'TRT-5678': ['TRT-5679']
    },
    'blocks': {
        'DPTP-1234': ['TRT-5678', 'ACM-9012'],    # Blocker -> [Blocked]
        'TRT-5679': ['ACM-9013']
    },
    'depends_on': {
        'ACM-9012': ['DPTP-1234', 'TRT-5678'],    # Dependent -> [Dependencies]
        'ART-3456': ['DPTP-1235']
    }
}

# Reverse mappings for traversal
reverse_relationships = {
    'children_of': {
        'DPTP-1235': 'DPTP-1234',                # Child -> Parent
    },
    'blocked_by': {
        'TRT-5678': 'DPTP-1234',                 # Blocked -> Blocker
    },
    'dependency_of': {
        'DPTP-1234': ['ACM-9012']                # Dependency -> [Dependents]
    }
}
```

### Step 4: Impact Chain Analysis

For each relationship type, calculate impact metrics:

**Parent/Child Impact**:
- Count children per parent (epic scope)
- Identify orphaned children (missing parents)
- Calculate completion percentage for hierarchies
- Find blocked parents affecting all children

**Blocking Impact**:
- Identify critical blockers (blocking multiple issues)
- Map blocking chains (A blocks B blocks C)
- Find cross-project blockers
- Calculate blocker resolution priority

**Dependency Impact**:
- Map dependency networks
- Identify circular dependencies (error condition)
- Find issues with unresolved dependencies
- Calculate dependency resolution order

### Step 5: Cross-Project Analysis

Analyze relationships that span project boundaries:

```sql
-- Cross-project relationships requiring coordination
SELECT 
    na.ASSOCIATION_TYPE,
    parent.PROJECT_KEY as SOURCE_PROJECT,
    child.PROJECT_KEY as TARGET_PROJECT,
    COUNT(*) as RELATIONSHIP_COUNT
FROM JIRA_NODEASSOCIATION na
INNER JOIN JIRA_ISSUE parent ON na.SOURCE_NODE_ID = parent.ID
INNER JOIN JIRA_ISSUE child ON na.SINK_NODE_ID = child.ID
WHERE na.ASSOCIATION_TYPE IN ('IssueParentChildLink', 'IssueBlocksLink', 'IssueDependsLink')
  AND parent.PROJECT_KEY != child.PROJECT_KEY
  AND (parent.PROJECT_KEY IN (projects) OR child.PROJECT_KEY IN (projects))
GROUP BY na.ASSOCIATION_TYPE, parent.PROJECT_KEY, child.PROJECT_KEY
ORDER BY RELATIONSHIP_COUNT DESC;
```

## Output Format

### Relationship Summary
```markdown
## Relationship Analysis Summary

**Total Relationships**: 245
- Parent/Child: 89 hierarchies
- Blocking: 67 blocking relationships  
- Dependencies: 89 dependency links

**Cross-Project Relationships**: 34
- DPTP ↔ TRT: 12 dependencies
- TRT ↔ ACM: 8 blocking relationships
- ACM ↔ ART: 6 parent/child links
```

### Critical Blockers
```markdown
## Critical Blockers Requiring Attention

**🔴 DPTP-1234**: "CI Infrastructure Upgrade" 
- **Blocks**: 8 issues across TRT, ACM projects
- **Impact**: Deployment pipeline for v4.18 release
- **Status**: In Progress (updated 2 days ago)
- **Coordination**: Requires TRT team coordination

**🟡 TRT-5678**: "Database Migration Completion"
- **Blocks**: 3 issues in ACM project
- **Impact**: Feature delivery timeline
- **Status**: Waiting on vendor response
```

### Impact Assessment
```markdown
## Risk Analysis Based on Relationships

**High Risk Items**:
- Epic TRT-9000 has 12 children, 3 blocked by external dependencies
- Cross-project dependency chain: DPTP-1234 → TRT-5678 → ACM-9012 (3-project critical path)

**Coordination Requirements**:
- DPTP-TRT weekly sync needed for 12 active dependencies
- ACM team waiting on TRT-5678 resolution (affects 3 deliverables)
```

## Error Handling

**No Relationships Found**:
- Verify JIRA_NODEASSOCIATION table exists and is accessible
- Check if projects have relationships configured in Jira
- Suggest expanding project scope or timeframe

**Circular Dependencies Detected**:
- Identify and list all circular chains
- Recommend dependency resolution order
- Flag for immediate attention as this can cause deadlocks

**Missing Relationship Metadata**:
- Gracefully handle missing status/type lookup data
- Use ID values if PNAME lookups fail
- Continue analysis with available data

## Technical Details

### Database Schema Dependencies
- **JIRA_NODEASSOCIATION**: Core relationship table (SOURCE_NODE_ID, SINK_NODE_ID, ASSOCIATION_TYPE)
- **JIRA_ISSUE**: Issue metadata (ID, ISSUEKEY, PROJECT_KEY, SUMMARY, STATUS, etc.)
- **JIRA_ISSUESTATUS**: Status name lookups (ID, PNAME)
- **JIRA_ISSUETYPE**: Issue type lookups (ID, PNAME)

### Performance Considerations
- Relationship queries can be large - consider limiting by timeframe
- Use indexed columns (PROJECT_KEY, ISSUEKEY) for efficient filtering
- Batch relationship traversal to avoid timeout on large dependency networks

### Integration Notes
- Data structure compatible with existing weekly_status analysis pipeline
- Relationship context enhances risk assessment and business value analysis
- Output format designed for inclusion in leadership reports