# CLAUDE.md Update Suggestion Command

**Your Role**: Analyze conversation history and suggest new rules or patterns to add to CLAUDE.md.

**Important**: Output "suggestions for content to add to CLAUDE.md," not summaries or explanations of the conversation.

## Usage
```bash
/suggest-claude-md
```

## Analysis Criteria
Detect content that matches any of the following three trigger conditions.

### 1. Project-Specific Rules
**Detection Patterns**:
- "Use X instead of Y"
- "In this project, we do it like..."
- Instructions to modify standard generated code to match project-specific conventions

**Examples**:
- Use the project's custom Config module instead of directly referencing environment variables
- In this project, we have a rule against passing params directly in Controllers
- Use the existing finder methods instead of the ORM's standard methods

### 2. Repeated Similar Correction Instructions
**Detection Patterns**:
- The same type of correction instruction appears 2+ times
- Similar code modifications occur across multiple files
- The same advice is given multiple times

**Examples**:
- Instructions to add the same setup process across multiple tests
- The same pattern of modifications across multiple controllers
- Instructions to use the same finder method across multiple models

### 3. Patterns That Should Be Consistent Across Related Areas
**Detection Patterns**:
- "Make the implementation consistent between these areas"
- "Unify this between the Web and API sides"
- "Keep this consistent between the standard and WebView versions"
- Instructions to maintain uniformity across multiple related areas

**Examples**:
- "If the Web side uses `/brands/:brand_id/series`, the API side should use `/api/brands/:brand_id/series`"
- "When you update `/xxx`, also update `/webview/xxx` the same way"
- "Implement this image upload process with the same pattern in other models"

## Output Format
**Required**: Use the following format. Other formats (summaries, reports, explanations, etc.) are prohibited.

When a trigger is matched:
```markdown
I've analyzed the conversation history. Would you like to add the following to CLAUDE.md?

If you'd like to proceed, please instruct me with something like "Add this content to CLAUDE.md."

[Specific suggested content]

Reason: [Project-specific rule / Repeated similar correction instructions (N times) / Pattern that should be consistent across related areas]
```

When no triggers are matched:
```markdown
I've analyzed the conversation history. No new content to add to CLAUDE.md was found.
```

**Prohibited**:
- Don't write in completion report format like "Fixed..." or "Implemented..."
- Don't output conversation summaries or explanations
- Don't omit the opening "I've analyzed the conversation history."

## Suggestion Criteria
### Do Suggest (○)
- Universal rules that should be shared across the entire project
- Content with guaranteed technical accuracy
- Content that can be clearly defined as a rule
- New patterns that emerged from the current code changes

### Don't Suggest (×)
- Temporary decisions or case-specific responses
- Content already documented in CLAUDE.md
- Personal preferences or temporary experiments
- Unclear or ambiguous instructions

## Post-Execution Actions
1. Review the suggested content
2. Instruct Claude to add approved content to CLAUDE.md
3. Include it in the same PR as the code changes
4. Have team members review it during PR review