# Weekly Status Plugin

Evidence-based weekly status report generation with Jira activity analysis and natural text highlighting improvements. Generate executive-focused leadership reports from real work activity.

## Features

- **Draft Generation**: Create structured weekly status reports from evidence-based Jira activity data
- **Natural Improvement Workflow**: Highlight text and request changes directly - no commands needed
- **Relationship Analysis**: Leverage Jira parent/child, blocks, and depends-on relationships for impact assessment
- **Evidence Linking**: Permanent issue enumeration links that work regardless of when viewed
- **Multi-Signal Activity Filtering**: Advanced filtering based on completion, comments, actual status changes, priority escalations, and meaningful new issues

## Prerequisites

- Snowflake MCP server configured for JIRA_DB.CLOUD_MARTS access
- Anthropic API access for AI-powered analysis and report generation

## Commands

### `/weekly-status:draft`
Generate an evidence-based weekly status report from Jira activity data. Analyzes issues across specified projects using multi-signal filtering and creates a structured leadership report with permanent evidence links.

**Improvement workflow**: After generation, simply highlight any text and request changes:
- "add evidence links"
- "verify this claim"
- "remove duplicate content" 
- "explain technical terms for leadership"

No special commands needed - natural conversation works best.

## Installation

```bash
/plugin install weekly-status@ai-helpers
```

## For Plugin Developers

The plugin uses a modular skill-based architecture with reusable components for data collection, analysis, and report generation. See individual command and skill documentation for implementation details.