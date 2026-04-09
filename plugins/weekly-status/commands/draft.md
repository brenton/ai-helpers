---
description: "Generate an initial weekly status report from Jira activity data with relationship analysis"
argument-hint: "PROJECTS --days N [--focus AREA]"
---

## Name
weekly-status:draft

## Synopsis
```
/weekly-status:draft PROJECTS --days N [--focus AREA] [--output FILE]
```

## Description
Generates a comprehensive weekly status report by analyzing Jira activity across specified projects. Uses multi-signal filtering to identify relevant issues and creates an executive-focused leadership report with business impact analysis, risk assessment, and evidence linking.

The command leverages Jira relationship data (parent/child, blocks, depends-on) to provide context about dependencies and impact chains. Reports focus exclusively on demonstrable work activity with concrete evidence.

**Improvement Workflow**: After generation, highlight any text in the report and request changes directly (e.g., "add evidence", "verify this claim", "remove duplicate"). No special commands needed - natural text highlighting provides superior interactive refinement.

## Implementation

### Step 1 — Parse Arguments
Parse and validate command arguments:
- `PROJECTS`: Comma-separated list of Jira project keys (e.g., "DPTP,TRT,ACM,ART")
- `--days N`: Number of days to analyze (default: 7)
- `--focus AREA`: Optional focus area (strategic, team_dynamics, risks)
- `--output FILE`: Optional output file (default: .work/weekly-status/draft_report.md)

Validate that projects are valid Jira project keys and days is a positive integer.

### Step 2 — Verify Snowflake Connectivity
Use the `setup-snowflake` skill to:
- Verify MCP connection to Snowflake JIRA_DB.CLOUD_MARTS
- Authenticate with browser SSO if needed
- Set session context (role JIRA_CLOUDMARTS_GROUP, database JIRA_DB, schema CLOUD_MARTS)
- Confirm access to required tables

### Step 3 — Multi-Signal Activity Collection
Use the `multi-signal-activity` skill to collect relevant issues using 5-signal union approach:

1. **Work Completed**: Issues resolved/closed in timeframe (highest priority)
2. **Recent Comment**: Issues with active collaboration/discussion
3. **Status Transition**: Issues showing meaningful progress
4. **High Priority Update**: Issues with priority escalations or critical updates
5. **Newly Created**: Important new issues that emerged

Each issue includes an ACTIVITY_REASON field explaining why it was selected.

### Step 4 — Relationship Analysis
Use the `relationship-analysis` skill to:
- Query JIRA_NODEASSOCIATION table for issue relationships
- Collect parent/child relationships (IssueParentChildLink)
- Collect blocking dependencies (IssueBlocksLink)
- Collect depends-on relationships (IssueDependsLink)
- Build relationship graph for impact analysis

### Step 5 — Business Impact and Risk Analysis
Apply analysis skills in parallel:
- Use `risk-analyzer` skill for business impact categorization and risk assessment
- Use `team-dynamics` skill for collaboration pattern analysis
- Identify cross-project dependencies and escalation indicators
- Generate evidence queries and Jira URL links

### Step 6 — Report Generation
Launch `report-generator` agent to create structured markdown report:
- Executive Summary (1-2 critical business-impacting updates)
- Quality (cross-org initiatives with measurable progress)
- Risks/Issues (🔴 Critical, 🟡 At Risk, 🔵 Informational)
- Key Decisions (strategic/organizational decisions)
- Weekly Updates (most impactful items per project with Jira links)

**CRITICAL REPORTING STANDARD**: Only include items with concrete evidence of activity:
- ✅ Issues with recent comments, commits, status changes, or deliverables
- ✅ Completed work with resolution evidence
- ✅ Active work in progress (In Progress, Review, Testing, etc.)
- ❌ Dormant issues without recent activity evidence
- ❌ Aspirational or "hope to finish" work
- ❌ Issues in planning status (To Do, New, Backlog) - no actual work started
- ❌ Issues based solely on "updated" timestamp (automated updates, bot activity)

**EVIDENCE LINKING STANDARD**: Use enumerated issue lists for permanent link validity:
- ✅ Static issue keys: `issueKey IN (ART-1234, DPTP-5678)`
- ❌ Relative dates: `resolved >= -7d` (breaks over time)
- ✅ Preserve evidence chain with permanent identifiers

Leadership assumes awareness of backlog items - only report demonstrable progress or completion.

### Step 7 — Output and Context Preservation
- Write draft report to specified output file
- Save analysis data to `.work/weekly-status/draft_analysis.json` for iterative improvement
- Include token cost summary
- Display report location and next steps for iterative improvement

## Examples

**Basic weekly report:**
```bash
/weekly-status:draft DPTP,TRT,ACM --days 7
```

**Monthly strategic review:**
```bash
/weekly-status:draft DPTP,TRT,ACM,ART --days 30 --focus strategic --output monthly_review.md
```

**Team health focus:**
```bash
/weekly-status:draft DPTP --days 14 --focus team_dynamics
```

**Custom timeframe with risk focus:**
```bash
/weekly-status:draft TRT,SHIPSTRAT --days 21 --focus risks --output risk_assessment.md
```

## Return Value
Creates two files:
- **Draft Report**: Markdown file with structured leadership report
- **Analysis Context**: JSON file with detailed analysis data for iterative improvement

Prints summary including:
- Number of issues analyzed by activity reason
- Relationship counts (parent/child, blocking, depends-on)
- Token costs for AI analysis
- File locations for next steps

## Arguments
- `PROJECTS`: Comma-separated Jira project keys (required)
- `--days N`: Days to analyze, positive integer (default: 7)
- `--focus AREA`: Analysis focus - strategic, team_dynamics, risks (optional)
- `--output FILE`: Output file path (default: .work/weekly-status/draft_report.md)

## Error Handling
- Invalid project keys: Display available projects from schema discovery
- Snowflake connection failure: Guide user to MCP setup documentation
- No issues found: Suggest expanding timeframe or checking project activity
- AI analysis failure: Fall back to structured template report with raw data

## Skills Used
- `setup-snowflake`: Snowflake MCP connection verification
- `multi-signal-activity`: Advanced issue filtering and collection
- `relationship-analysis`: Jira issue relationship mapping
- `citation-enhancer`: Evidence linking with permanent issue enumeration

## Agents Used
- `report-generator`: AI-powered leadership report creation

## Interactive Improvements
Use natural text highlighting + conversation for refinements:
- Highlight text → "add evidence links"
- Highlight text → "verify this claim"  
- Highlight text → "remove duplicate content"
- Highlight text → "explain technical terms for leadership"