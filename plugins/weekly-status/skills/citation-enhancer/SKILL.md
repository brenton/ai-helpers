---
name: "Citation and Evidence Enhancer"
description: "Add Jira links, evidence queries, and verification sources to weekly status report content"
---

# Citation and Evidence Enhancer Skill

## When to Use This Skill

Use this skill when you need to enhance weekly status report content with proper citations and evidence sources. This skill is commonly invoked during the improve command when users request:

- "Citation needed" - Add Jira links and evidence queries
- "Add sources" - Include verification links for claims
- "Evidence linking" - Connect statements to supporting Jira data
- "Verification support" - Enable leaders to independently verify claims

This skill is critical for maintaining credibility and enabling leadership to drill down into details when needed.

## Implementation Steps

### Step 1: Parse Target Content and Claims

Analyze the provided text to identify different types of claims that require evidence:

**Quantitative Claims**:
- "150 tickets resolved" → Needs JQL query showing count
- "96.2% first-time pass rate" → Needs metric source and timeframe
- "40% reduction in blocker resolution" → Needs before/after comparison

**Specific Issues/Outcomes**:
- "TRT-4521 blocking deployment" → Needs direct Jira link
- "ACM v4.18 release delayed" → Needs issue link and timeline evidence

**Process/Team Claims**:
- "Daily standups implemented" → Needs meeting evidence or process documentation
- "Cross-team coordination improved" → Needs collaboration metrics or examples

**Timeline Claims**:
- "Completed last week" → Needs resolution date verification
- "Expected by Friday" → Needs due date or target date reference

### Step 2: Generate Jira Issue Links

For any mentioned Jira issue keys, create proper markdown links:

**Pattern Detection**:
```regex
[A-Z]{2,10}-\d{1,6}
```

**Link Generation**:
```markdown
[DPTP-1234](https://issues.redhat.com/browse/DPTP-1234)
[TRT-5678](https://issues.redhat.com/browse/TRT-5678)
```

**Bulk Issue References**:
For lists of issues, create expandable sections:
```markdown
Resolved 45 CI stability tickets ([DPTP-1001](https://issues.redhat.com/browse/DPTP-1001), [DPTP-1002](https://issues.redhat.com/browse/DPTP-1002), [see all 45 issues](https://issues.redhat.com/issues/?jql=project%20%3D%20DPTP%20AND%20labels%20%3D%20ci-stability%20AND%20resolved%20%3E%3D%20-7d))
```

### Step 3: Create Evidence Queries

Generate JQL queries that support quantitative claims using **enumerated issue lists** instead of relative dates to ensure permanent link validity:

**CRITICAL**: Use issue enumeration, not relative date queries (`-7d`) which break over time.

**Count Verification Queries**:
```jql
# For "15 monitoring plugin tickets resolved"
issueKey IN (ART-14661, ART-14662, ART-14650, ART-14651, ART-14654, ART-14655, ART-14656, ART-14657, ART-14658, ART-14659, ART-14660, ART-14663, ART-14664, ART-14616, ART-14861)

# For "Critical bugs fixed this week"  
issueKey IN (ART-14864, DPTP-4752, TRT-2625) AND priority = Critical

# For "Cross-project dependencies resolved"
issueKey IN (DPTP-1234, TRT-5678, ACM-9012) AND issueFunction in linkedIssues("EPIC-123")
```

**Best Practices**:
- Query actual data sources to get concrete issue keys
- Build static lists from time-bounded analysis results
- Include issue count in link text for verification
- Preserve evidence chain with permanent identifiers

**Status/Progress Verification**:
```jql
# For "Epic 85% complete"
parent = "DPTP-1000" AND status IN (Resolved, Closed, Done)

# For "Blocker resolution improved"
priority = Blocker AND updated >= -7d AND project IN (DPTP, TRT)
```

**Timeline Verification**:
```jql
# For "Issues created this week"
project = DPTP AND created >= -7d

# For "Overdue items"
project IN (DPTP, TRT) AND due < now() AND status NOT IN (Resolved, Closed)
```

### Step 4: Format Evidence Links

Create clickable links that open directly in Jira:

**Enumerated Issue Evidence**:
```markdown
**Evidence**: [15 monitoring plugins resolved](https://issues.redhat.com/issues/?jql=issueKey%20IN%20(ART-14661%2C%20ART-14662%2C%20ART-14650%2C%20ART-14651%2C%20ART-14654%2C%20ART-14655%2C%20ART-14656%2C%20ART-14657%2C%20ART-14658%2C%20ART-14659%2C%20ART-14660%2C%20ART-14663%2C%20ART-14664%2C%20ART-14616%2C%20ART-14861))
```

**Complex Verification**:
```markdown
**Verification Queries**:
- [Critical bugs fixed](https://issues.redhat.com/issues/?jql=project%20IN%20(DPTP%2C%20TRT)%20AND%20type%20%3D%20Bug%20AND%20priority%20%3D%20Critical%20AND%20resolved%20%3E%3D%20-7d) (7 total)
- [Cross-project dependencies](https://issues.redhat.com/issues/?jql=project%20IN%20(DPTP%2C%20TRT)%20AND%20linkedIssues(%22DPTP-1234%22)) (12 linked)
```

**Aggregate Evidence**:
```markdown
**Supporting Data** ([view query](https://issues.redhat.com/issues/?jql=project%20%3D%20DPTP%20AND%20labels%20%3D%20performance%20AND%20resolved%20%3E%3D%20-30d)):
- Performance improvements: 23 tickets
- Load testing enhancements: 8 tickets  
- Monitoring upgrades: 12 tickets
```

### Step 5: URL Encoding for JQL

Properly encode JQL queries for URL compatibility:

**Common Encodings**:
- Space: `%20`
- Equals: `%3D`
- Comma: `%2C`
- Quotes: `%22`
- Greater than/equals: `%3E%3D`
- Parentheses: `%28` and `%29`

**Example Transformation**:
```text
Original JQL: project = DPTP AND resolved >= -7d
Encoded URL:  project%20%3D%20DPTP%20AND%20resolved%20%3E%3D%20-7d
Full URL:     https://issues.redhat.com/issues/?jql=project%20%3D%20DPTP%20AND%20resolved%20%3E%3D%20-7d
```

### Step 6: Context-Aware Enhancement

Tailor evidence based on the type of content and audience:

**Executive Summary Evidence**:
- Focus on business impact metrics
- Include customer-facing issue counts
- Link to high-level trend queries

**Technical Section Evidence**:
- Provide detailed issue breakdowns
- Include component-specific queries
- Link to implementation details

**Risk Section Evidence**:
- Show blocker chains and dependencies
- Include escalation timelines
- Link to critical path analyses

## Output Format

### Enhanced Text with Inline Citations

**Before**:
```markdown
DPTP team resolved 45 CI stability issues, improving first-time pass rate to 96.2%.
```

**After**:
```markdown
DPTP team resolved [45 CI stability issues](https://issues.redhat.com/issues/?jql=project%20%3D%20DPTP%20AND%20labels%20%3D%20ci-stability%20AND%20resolved%20%3E%3D%20-7d), improving first-time pass rate to 96.2% ([trend analysis](link-to-metrics-dashboard)).
```

### Evidence Blocks for Complex Claims

**Before**:
```markdown
Cross-project coordination significantly improved with resolution of multiple blocking dependencies.
```

**After**:
```markdown
Cross-project coordination significantly improved with resolution of multiple blocking dependencies:

**Evidence**:
- [DPTP→TRT blockers resolved](https://issues.redhat.com/issues/?jql=project%20%3D%20TRT%20AND%20issueFunction%20in%20linkedIssuesOf(%22project%20%3D%20DPTP%22%2C%20%22blocks%22)%20AND%20resolved%20%3E%3D%20-7d): 8 issues
- [ACM dependency completion](https://issues.redhat.com/issues/?jql=project%20%3D%20ACM%20AND%20issueFunction%20in%20linkedIssuesOf(%22project%20%3D%20TRT%22%2C%20%22depends%22)%20AND%20resolved%20%3E%3D%20-7d): 5 issues
- [Cross-team communication threads](https://issues.redhat.com/issues/?jql=project%20IN%20(DPTP%2C%20TRT%2C%20ACM)%20AND%20comment%20~%20%22coordination%22%20AND%20updated%20%3E%3D%20-7d): 12 issues
```

### Verification Footer

Add verification footer to major sections:

```markdown
---
**Verification**: All quantitative claims above are supported by [Jira queries](https://issues.redhat.com/issues/?jql=project%20IN%20(DPTP%2C%20TRT%2C%20ACM)%20AND%20updated%20%3E%3D%20-7d) and can be independently verified through the provided links.
```

## Error Handling

**Invalid Issue Keys**:
- Validate issue key format before creating links
- Check issue existence if Jira API access available
- Use formatted text instead of links for non-existent issues

**Complex JQL Generation Failures**:
- Fall back to simpler project-based queries
- Provide manual JQL construction guidance
- Include general search links as alternatives

**URL Encoding Issues**:
- Test encoded URLs before including in output
- Provide unencoded JQL as fallback in code blocks
- Include instructions for manual query construction

## Technical Details

### JQL Query Patterns

**Time-Based Queries**:
- Last 7 days: `>= -7d`
- This week: `>= startOfWeek()`
- Last month: `>= -30d`
- Current release cycle: `>= "2024-03-01"`

**Relationship Queries**:
- Linked issues: `issueFunction in linkedIssuesOf("DPTP-1234")`
- Parent/child: `parent = "EPIC-123"`
- Blocking: `issueFunction in linkedIssuesOf("project = DPTP", "blocks")`

**Complex Filters**:
- Multiple projects: `project IN (DPTP, TRT, ACM)`
- Issue types: `type IN (Bug, Story, Task)`
- Priorities: `priority IN (Critical, High)`
- Custom fields: `"Team[Select List (multiple choices)]" ~ "Platform"`

### Integration Notes
- Compatible with existing weekly_status report structure
- Preserves original content while adding evidence
- Supports iterative enhancement through multiple passes
- Maintains readability while adding verification capability