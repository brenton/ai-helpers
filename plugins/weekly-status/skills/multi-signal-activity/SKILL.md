---
name: "Multi-Signal Jira Activity Collection"
description: "Advanced issue filtering using 5-signal union approach for comprehensive activity analysis"
---

# Multi-Signal Jira Activity Collection Skill

## When to Use This Skill

Use this skill when you need to collect comprehensive Jira activity data that goes beyond simple date-based filtering. This skill implements a proven 5-signal union approach that captures different types of meaningful activity:

- Issues that represent **completed work** (highest priority for leadership reporting)
- Issues with **active collaboration** (team dynamics and communication)
- Issues showing **meaningful progress** (status transitions and updates)
- Issues with **priority escalations** (critical problems requiring attention)
- **New important issues** (emerging problems and initiatives)

This approach ensures no significant activity is missed while providing context about why each issue was selected.

## Implementation Steps

### Step 1: Validate Input Parameters

Parse and validate the following parameters:
- **Projects**: Comma-separated list of Jira project keys (e.g., 'DPTP,TRT,ACM,ART')
- **Days**: Number of days for lookback period (default: 7, max: 90)
- **Activity Mode**: 'focused' (default), 'comments-only', or 'all-activity'

Validate project keys exist in JIRA_ISSUE table and timeframe is reasonable.

### Step 2: Signal 1 — Work Completed (Highest Priority)

Capture issues that represent delivered value:

```sql
-- Signal 1: Issues completed within timeframe (highest priority signal)
SELECT DISTINCT
    ji.ID,
    ji.ISSUEKEY,
    ji.PROJECT_KEY,
    ji.SUMMARY,
    SUBSTR(ji.DESCRIPTION, 1, 2000) AS DESCRIPTION_EXCERPT,
    ji.CREATED,
    ji.UPDATED,
    ji.RESOLUTIONDATE,
    jit.PNAME as ISSUE_TYPE,
    jis.PNAME as ISSUE_STATUS,
    'work_completed' as ACTIVITY_REASON,
    'Issues resolved/closed in timeframe - completed work delivered' as ACTIVITY_EXPLANATION
FROM JIRA_ISSUE ji
LEFT JOIN JIRA_ISSUETYPE jit ON ji.ISSUETYPE = jit.ID
LEFT JOIN JIRA_ISSUESTATUS jis ON ji.ISSUESTATUS = jis.ID
WHERE ji.PROJECT_KEY IN (projects)
  AND ji.RESOLUTIONDATE >= DATEADD(day, -days, CURRENT_DATE())
  AND ji.RESOLUTIONDATE IS NOT NULL
  AND jis.PNAME IN ('Resolved', 'Closed', 'Done', 'Complete', 'Verified', 'Released')
```

### Step 3: Signal 2 — Recent Meaningful Comments

Capture active collaboration and discussion:

```sql
-- Signal 2: Issues with recent meaningful comments (active collaboration)
SELECT DISTINCT
    ji.ID,
    ji.ISSUEKEY,
    ji.PROJECT_KEY,
    ji.SUMMARY,
    SUBSTR(ji.DESCRIPTION, 1, 2000) AS DESCRIPTION_EXCERPT,
    ji.CREATED,
    ji.UPDATED,
    ji.RESOLUTIONDATE,
    jit.PNAME as ISSUE_TYPE,
    jis.PNAME as ISSUE_STATUS,
    'recent_comment' as ACTIVITY_REASON,
    'Issues with active discussion/collaboration' as ACTIVITY_EXPLANATION
FROM JIRA_ISSUE ji
LEFT JOIN JIRA_ISSUETYPE jit ON ji.ISSUETYPE = jit.ID
LEFT JOIN JIRA_ISSUESTATUS jis ON ji.ISSUESTATUS = jis.ID
INNER JOIN JIRA_COMMENT jc ON jc.ISSUEID = ji.ID
WHERE ji.PROJECT_KEY IN (projects)
  AND jc.CREATED >= DATEADD(day, -days, CURRENT_DATE())
  AND LENGTH(TRIM(jc.BODY)) > 20  -- Filter out trivial comments
  AND jc.BODY NOT LIKE '%moved to%'  -- Filter out automated comments
  AND jc.BODY NOT LIKE '%changed the status%'
  AND jis.PNAME NOT IN ('New', 'Open', 'Backlog', 'To Do')  -- Only actual work in progress
```

### Step 4: Signal 3 — Status Transitions

Capture meaningful progress indicators:

```sql
-- Signal 3: Issues with meaningful status transitions (progress indicators)  
-- Use field history instead of updated field to track actual status changes
SELECT DISTINCT
    ji.ID,
    ji.ISSUEKEY,
    ji.PROJECT_KEY,
    ji.SUMMARY,
    SUBSTR(ji.DESCRIPTION, 1, 2000) AS DESCRIPTION_EXCERPT,
    ji.CREATED,
    ji.UPDATED,
    ji.RESOLUTIONDATE,
    jit.PNAME as ISSUE_TYPE,
    jis.PNAME as ISSUE_STATUS,
    'status_transition' as ACTIVITY_REASON,
    'Issues with actual status changes - not just updated timestamps' as ACTIVITY_EXPLANATION
FROM JIRA_ISSUE ji
LEFT JOIN JIRA_ISSUETYPE jit ON ji.ISSUETYPE = jit.ID
LEFT JOIN JIRA_ISSUESTATUS jis ON ji.ISSUESTATUS = jis.ID
INNER JOIN JIRA_CHANGEGROUP jcg ON jcg.ISSUEID = ji.ID
INNER JOIN JIRA_CHANGEITEM jci ON jci.GROUPID = jcg.ID
WHERE ji.PROJECT_KEY IN (projects)
  AND jcg.CREATED >= DATEADD(day, -days, CURRENT_DATE())  -- Actual field changes, not just updates
  AND jci.FIELD = 'status'  -- Only status changes
  AND ji.RESOLUTIONDATE IS NULL  -- Exclude completed work (covered by signal 1)
  AND jis.PNAME NOT IN ('New', 'Open', 'Backlog', 'To Do')  -- Must have progressed beyond initial states
```

### Step 5: Signal 4 — High Priority Updates

Capture escalations and critical issues:

```sql
-- Signal 4: High-priority issues with recent updates (escalations)
SELECT DISTINCT
    ji.ID,
    ji.ISSUEKEY,
    ji.PROJECT_KEY,
    ji.SUMMARY,
    SUBSTR(ji.DESCRIPTION, 1, 2000) AS DESCRIPTION_EXCERPT,
    ji.CREATED,
    ji.UPDATED,
    ji.RESOLUTIONDATE,
    jit.PNAME as ISSUE_TYPE,
    jis.PNAME as ISSUE_STATUS,
    'high_priority_update' as ACTIVITY_REASON,
    'High-priority issues requiring leadership attention' as ACTIVITY_EXPLANATION
FROM JIRA_ISSUE ji
LEFT JOIN JIRA_ISSUETYPE jit ON ji.ISSUETYPE = jit.ID
LEFT JOIN JIRA_ISSUESTATUS jis ON ji.ISSUESTATUS = jis.ID
WHERE ji.PROJECT_KEY IN (projects)
  AND (
    ji.RESOLUTIONDATE >= DATEADD(day, -days, CURRENT_DATE())  -- Recently resolved high-priority work
    OR (jc.CREATED >= DATEADD(day, -days, CURRENT_DATE()) AND LENGTH(TRIM(jc.BODY)) > 20)  -- Recent meaningful comments
  )
  AND jis.PNAME NOT IN ('New', 'Open', 'Backlog', 'To Do')  -- Only actual work in progress
  AND (
    ji.PRIORITY IN (1, 2)  -- Critical, High priority
    OR jit.PNAME IN ('Bug', 'Vulnerability', 'Incident')
    OR ji.SUMMARY ILIKE '%critical%'
    OR ji.SUMMARY ILIKE '%urgent%'
    OR ji.SUMMARY ILIKE '%blocker%'
  )
```

### Step 6: Signal 5 — Newly Created Important Issues

Capture emerging problems and new initiatives:

```sql
-- Signal 5: Recently created important issues (emerging problems/initiatives)
SELECT DISTINCT
    ji.ID,
    ji.ISSUEKEY,
    ji.PROJECT_KEY,
    ji.SUMMARY,
    SUBSTR(ji.DESCRIPTION, 1, 2000) AS DESCRIPTION_EXCERPT,
    ji.CREATED,
    ji.UPDATED,
    ji.RESOLUTIONDATE,
    jit.PNAME as ISSUE_TYPE,
    jis.PNAME as ISSUE_STATUS,
    'newly_created' as ACTIVITY_REASON,
    'Important new issues that emerged during the period' as ACTIVITY_EXPLANATION
FROM JIRA_ISSUE ji
LEFT JOIN JIRA_ISSUETYPE jit ON ji.ISSUETYPE = jit.ID
LEFT JOIN JIRA_ISSUESTATUS jis ON ji.ISSUESTATUS = jis.ID
WHERE ji.PROJECT_KEY IN (projects)
  AND ji.CREATED >= DATEADD(day, -days, CURRENT_DATE())
  AND (
    ji.PRIORITY IN (1, 2, 3)  -- Critical, High, Medium priority
    OR jit.PNAME IN ('Epic', 'Story', 'Bug', 'Vulnerability')
    OR LENGTH(ji.SUMMARY) > 30  -- Non-trivial issues
  )
```

### Step 7: Union All Signals and Deduplicate

Combine all signals into a unified result set:

```sql
-- Final union of all signals with deduplication
WITH all_signals AS (
  -- Union all 5 signal queries here
  SELECT * FROM signal_1_work_completed
  UNION
  SELECT * FROM signal_2_recent_comments  
  UNION
  SELECT * FROM signal_3_status_transitions
  UNION
  SELECT * FROM signal_4_high_priority
  UNION
  SELECT * FROM signal_5_newly_created
)
SELECT *
FROM all_signals
ORDER BY 
  -- Prioritize completed work, then by project and update date
  CASE ACTIVITY_REASON 
    WHEN 'work_completed' THEN 1
    WHEN 'high_priority_update' THEN 2
    WHEN 'status_transition' THEN 3
    WHEN 'recent_comment' THEN 4
    WHEN 'newly_created' THEN 5
  END,
  PROJECT_KEY,
  UPDATED DESC;
```

## Output Format

### Activity Summary
```json
{
  "collection_summary": {
    "projects": ["DPTP", "TRT", "ACM", "ART"],
    "timeframe_days": 7,
    "total_issues": 156,
    "signals": {
      "work_completed": 45,
      "recent_comment": 38,
      "status_transition": 29,
      "high_priority_update": 23,
      "newly_created": 21
    }
  },
  "issues": [
    {
      "issuekey": "DPTP-1234",
      "project_key": "DPTP",
      "summary": "Implement hermetic build improvements",
      "activity_reason": "work_completed",
      "activity_explanation": "Issues resolved/closed in timeframe - completed work delivered",
      "issue_type": "Story",
      "issue_status": "Resolved",
      "created": "2024-03-15T10:30:00Z",
      "updated": "2024-04-08T14:20:00Z",
      "resolutiondate": "2024-04-08T14:20:00Z"
    }
  ]
}
```

### Activity Reason Distribution
```markdown
## Activity Collection Summary

**Total Issues Collected**: 156 across DPTP, TRT, ACM, ART
**Timeframe**: Last 7 days

**Signal Distribution**:
- 🏁 **Work Completed**: 45 issues (29%) - Delivered value
- 💬 **Recent Comments**: 38 issues (24%) - Active collaboration  
- ⚡ **Status Transitions**: 29 issues (19%) - Meaningful progress
- 🚨 **High Priority**: 23 issues (15%) - Critical attention needed
- 🆕 **Newly Created**: 21 issues (13%) - Emerging work

**Key Insights**:
- High completion rate indicates strong delivery momentum
- Significant comment activity suggests good team collaboration
- Priority escalations require leadership attention
```

## Error Handling

**No Issues Found**:
- Check if projects have activity in the specified timeframe
- Suggest expanding timeframe or project scope
- Verify project keys are correct and accessible

**Query Timeout**:
- Reduce timeframe or number of projects
- Implement pagination for large result sets
- Suggest using activity-mode 'comments-only' for faster execution

**Missing Lookup Tables**:
- Gracefully handle missing JIRA_ISSUETYPE or JIRA_ISSUESTATUS data
- Use numeric IDs if PNAME lookups fail
- Continue collection with available data

## Technical Details

### Performance Optimization
- Each signal query is optimized with appropriate indexes
- Union approach allows parallel execution of signal collection
- DISTINCT operations prevent duplicate issues across signals
- Timeframe filtering early in WHERE clauses for efficiency

### Customization Options
- **Activity Mode 'comments-only'**: Execute only Signal 2 for faster results
- **Activity Mode 'all-activity'**: Include additional signals for comprehensive collection
- **Project Filtering**: Can be applied at query level or post-collection
- **Priority Filtering**: Adjustable based on organization priority schemes

### Integration Notes
- Output format compatible with existing risk-analyzer and team-dynamics skills
- ACTIVITY_REASON field enables targeted analysis based on why issues were collected
- Temporal accuracy maintained for precise leadership reporting
- Evidence linking prepared through Jira URL patterns