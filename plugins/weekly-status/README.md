# Weekly Status Plugin

Interactive weekly status report generation with relationship analysis and iterative improvement capabilities. Generate executive-focused leadership reports from Jira data and iteratively refine them through conversation.

## Features

- **Draft Generation**: Create structured weekly status reports from Jira activity data
- **Interactive Improvement**: Iteratively refine reports with citations, explanations, and context
- **Relationship Analysis**: Leverage Jira parent/child, blocks, and depends-on relationships for impact assessment
- **Evidence Linking**: Automatic Jira URL generation and query creation for verification
- **Multi-Signal Activity Filtering**: Advanced filtering based on completion, comments, status transitions, priority changes, and new issues

## Prerequisites

- Snowflake MCP server configured for JIRA_DB.CLOUD_MARTS access
- Anthropic API access for AI-powered analysis and report generation

## Commands

### `/weekly-status:draft`
Generate an initial weekly status report from Jira activity data. Analyzes issues across specified projects using multi-signal filtering and creates a structured leadership report.

### `/weekly-status:improve`
Iteratively improve sections of the draft report based on user feedback. Supports citation enhancement, explanation expansion, and impact analysis using relationship data.

### `/weekly-status:analyze-relationships`
Analyze Jira issue relationships (parent/child, blocking dependencies) to understand impact chains and risk propagation across projects.

## Installation

```bash
/plugin install weekly-status@ai-helpers
```

## For Plugin Developers

The plugin uses a modular skill-based architecture with reusable components for data collection, analysis, and report generation. See individual command and skill documentation for implementation details.