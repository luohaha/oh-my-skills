---
name: pr-description
description: Generate comprehensive pull request descriptions that help reviewers understand code changes. Use when the user asks to "generate PR description", "create PR summary", "write PR intro", or needs help documenting their code changes for review. Analyzes git diffs or specified files to create structured descriptions covering why changes were made and what was implemented.
---

# PR Description Generator

Generate clear, comprehensive pull request descriptions that help reviewers quickly understand your changes.

## Overview

This skill analyzes code changes and generates structured PR descriptions with two key sections:
- **Why I'm Doing This**: Business context, technical motivation, and background
- **What I'm Doing**: Implementation details, technical decisions, risks, and review focus areas

## Workflow

### Step 1: Gather Code Changes

Choose the appropriate method based on user input:

**Method A: Analyze Current Branch (Default)**
- Run `git status` to check current branch
- Run `git diff $(git merge-base HEAD main)...HEAD` to get all changes since branch diverged
- Run `git log $(git merge-base HEAD main)..HEAD --oneline` to see commit history

**Method B: User-Specified Files**
- If user mentions specific files, read those files
- Compare with git diff for those specific paths if needed

### Step 2: Understand the Changes

Analyze the code changes to understand:
1. **Purpose**: What problem is being solved? What feature is being added?
2. **Scope**: Which files/modules are affected? How significant are the changes?
3. **Type**: Is this a feature, bug fix, refactoring, optimization, or something else?
4. **Impact**: What are the functional changes? Any performance implications?
5. **Risks**: Breaking changes, edge cases, deployment considerations?

**Analysis Tips:**
- Look at file names and paths to understand affected modules
- Read commit messages for context
- Identify patterns: new files (feature), modified logic (bug fix), restructured code (refactoring)
- Check for configuration changes, database migrations, API changes
- Note any TODO comments or follow-up work mentioned

### Step 3: Generate the PR Description

Create a structured Markdown description with these sections:

#### Why I'm Doing This
Provide context that helps reviewers understand the motivation:
- Business problem or user need being addressed
- Technical motivation (performance, security, maintainability, tech debt)
- Reference to related issues, tickets, or discussions
- Background information for those unfamiliar with the context

**Keep it concise but informative** - 2-4 sentences or bullet points typically suffice.

#### What I'm Doing
Describe the implementation in a way that aids review:

**High-level summary** (required):
- Brief overview of the changes (1-2 sentences)
- Key components or files modified

**Technical details** (include when relevant):
- Important technical decisions and trade-offs
- New features or capabilities added
- Performance optimizations or improvements
- Architecture or design pattern changes
- Dependencies added or updated

**Testing and validation** (include when relevant):
- Test coverage added
- How changes were validated
- Edge cases handled

**Risks and considerations** (include when present):
- Breaking changes or deprecations
- Known limitations or edge cases
- Follow-up work needed
- Deployment considerations
- Areas requiring special review attention

**Format Guidelines:**
- Use bullet points for readability
- Group related changes together
- Highlight important information with **bold**
- Use code formatting for technical terms: `function_name`, `file.ts`
- Keep it scannable - reviewers should grasp key points quickly

### Step 4: Review and Refine

Before presenting the PR description:
- Ensure "Why" section provides sufficient context
- Verify "What" section accurately reflects the code changes
- Check that risks and important considerations are called out
- Confirm the description helps reviewers know what to focus on

## Output Format

Present the generated PR description in a Markdown code block so the user can easily copy it:

```markdown
## Why I'm Doing This

[Context and motivation]

## What I'm Doing

[Implementation details]

**Technical Details:**
- [Key technical points]

**Risks:**
- [Any risks or considerations]
```

## Best Practices

1. **Be thorough but concise**: Include all relevant information without overwhelming reviewers
2. **Think like a reviewer**: What would help someone understand and review these changes?
3. **Call out the important stuff**: Highlight breaking changes, risks, and areas needing careful review
4. **Provide context**: Don't assume reviewers have full background knowledge
5. **Be honest about limitations**: Note known issues, follow-up work, or trade-offs made

## Examples and Reference

For detailed examples of well-written PR descriptions across different scenarios (features, bug fixes, optimizations, refactoring), see [pr_examples.md](references/pr_examples.md).

Load this reference when you need inspiration or want to understand best practices for specific types of changes.
