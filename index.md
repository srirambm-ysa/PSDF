---
layout: default
title: Probabilistic-Safe Development Framework (PSDF)
---

# Probabilistic-Safe Development Framework (PSDF)

Version: 1.0
Date: May 2026
Status: Draft
Audience: Offshore Dev Teams, Team Leads, MSME Clients

## Preamble

This framework addresses a fundamental mismatch in modern software development: AI code generation tools are probabilistic — they produce varied, sometimes incorrect output — while software systems require deterministic, verifiable behaviour.

PSDF provides a methodology to safely integrate AI into a standard SDLC by wrapping probabilistic AI output inside deterministic verification layers. It draws on Pair Programming, Test-Driven Development (TDD), and the BA-QA model — not as alternatives, but as complementary layers of a single system.

### Core Principle

Use AI to generate. Use humans and tests to verify. Use documents to persist.

AI is treated as a capable but unreliable junior contributor — never as a source of truth.

The Five Structural Problems PSDF Addresses

#	Problem	Nature
| #   | Problem | Nature |
| --- | --- | --- |
| 1   | AI has no memory between sessions | Architectural — cannot be fixed |
| 2   | Full context must be re-supplied each session | Architectural — can be mitigated |
| 3   | AI hallucinates — confidently produces wrong output | Structural — can be contained |
| 4   | Documentation rot silently corrupts AI context | Process — can be managed |
| 5   | Developers expect deterministic tools | Cultural — must be addressed |



# SECTION 1: INTERNAL TEAM PLAYBOOK

Practical, how-to guide for developers and team leads

1.1 The PSDF Mental Model

Before writing a single line of AI-assisted code, every team member must internalise one shift:

```
OLD: AI is a tool. I give input, I get correct output.
NEW: AI is a junior dev. I give context, I review output, I own the result.
```

You are not a prompt engineer. You are a senior developer supervising an AI contributor.

## 1.2 The Four Non-Negotiables

These are team rules, not suggestions. Violations are treated as process failures, same as a broken build.

1.  **Never merge AI-generated code you cannot explain line by line.**
2.  **Every AI session starts with a context file. No context file = no AI session.**
3.  **Tests are written before AI generates code. Always.**
4.  **Acceptance criteria from BA is the only source of truth. Not AI output.**

## 1.3 Daily AI-Assisted Development Loop

```
START OF TASK
│
├── 1. Pull latest AGENTS.md + relevant context file
├── 2. Read BA acceptance criteria for the task
├── 3. Write failing test(s) that represent acceptance criteria
│
├── AI GENERATION STEP
│   ├── Feed: context file + failing test + specific instruction
│   ├── AI generates code
│   └── Run tests immediately
│
├── REVIEW STEP (Pair or Self)
│   ├── Can you explain every line?
│   ├── Does it match acceptance criteria — not just pass tests?
│   └── Any hardcoded values, missing edge cases, invented methods?
│
├── COMMIT
│   ├── Tests pass in CI
│   ├── PR description includes: what AI generated, what you changed, why
│   └── Update context file if architecture changed
│
END OF TASK
```

## 1.4 Context File Standard

Every repository must contain these files, kept current by the team:

### 'AGENTS.md' — Top-level AI context

```markdown
# Project: [Name]
# Stack: [Languages, frameworks, versions]
# Architecture: [Brief — monolith/microservice, DB type, auth method]
# Key conventions: [Naming, folder structure, patterns used]
# What AI must never do: [e.g., never use raw SQL, never skip validation]
# Last updated: [Date] by [Name]
```

### `CONTEXT/[feature-name].md` — Per-feature context

```markdown
# Feature: [Name]
# Purpose: [One sentence]
# Key files: [List]
# Dependencies: [External services, APIs]
# Known AI failure points: [What hallucinated last time]
# Last updated: [Date] by [Name]
```
### Context file update rule:

> If you changed architecture, added a dependency, or discovered an AI failure pattern — update the context file **in the same commit**. Not later. Same commit.


## 1.5 Prompt Library

Store prompts that reliably produce good output. Treat them like code.

```
/prompts
  /codegen
    unit-test-from-acceptance-criteria.md
    refactor-function.md
    generate-api-endpoint.md
  /review
    review-ai-output.md
    find-hallucinations.md
  /debug
    explain-failing-test.md
```

**Prompt template format:**

```
CONTEXT: {paste AGENTS.md summary}
TASK: {one specific task only}
CONSTRAINTS: {language version, patterns to follow, what to avoid}
OUTPUT FORMAT: {function only / full file / diff}
VERIFY BY: {what the output must satisfy}
```

## 1.6 The Pair Programming Rule for AI Teams

AI changes the pair programming model:

```
TRADITIONAL:    Driver writes code ←→ Navigator reviews
AI-ASSISTED:    Driver prompts AI + reviews output ←→ Navigator verifies against acceptance criteria
```
Recommended pairings:

- **Senior + Junior:** Junior drives AI, Senior reviews. Builds junior skill while maintaining quality.
- **Dev + BA:** Dev generates, BA validates against criteria in real time for complex features.

Pair review is **mandatory for any AI-generated code touching core business logic, auth, payments, or data handling.**

## 1.7 Handling Hallucinations

When AI produces wrong output, do not just fix it silently.

1.  Note the failure in the relevant `CONTEXT/[feature].md` under "Known AI failure points"
2.  Log it in the sprint retrospective
3.  If it's a recurring hallucination on a specific library/pattern — add a constraint to the prompt template

Over time, your prompt library becomes a **hallucination suppression system** built from real failures.

# SECTION 2: FORMAL METHODOLOGY DOCUMENT

Structured reference for team leads, architects, and process owners

## 2.1 Framework Overview

PSDF integrates three established software methodologies into a unified AI-safe SDLC:

| Methodology | Role in PSDF | LLM Problem It Addresses |
| --- | --- | --- |
| BA-QA Model | Requirements persistence + validation closure | Context loss, doc rot |
| Test-Driven Development | Deterministic verification of probabilistic output | Hallucination, variance |
| Pair Programming | Human judgment layer over AI generation | Over-trust, missed edge cases |


## 2.2 PSDF Phases

### Phase 1 — Requirements (BA-Led)

**Owner:** Business Analyst  
**AI Role:** Assist in edge case generation, ambiguity clarification  
**Output:** Acceptance Criteria Document (ACD)

The ACD serves a dual purpose in PSDF:

1.  Traditional: Defines what "done" means
2.  PSDF-specific: Becomes the **persistent context seed** for all AI sessions in that feature

ACD format requirement:

```
Feature: [Name]
User Story: As a [role], I want [action] so that [outcome]
Acceptance Criteria:
  - GIVEN [state] WHEN [action] THEN [outcome]
  - (repeat for each scenario)
Edge Cases: [List]
Out of Scope: [Explicit exclusions]
AI Context Notes: [Any domain knowledge AI will need]
```


### Phase 2 — Design (Dev + BA)

**Owner:** Senior Developer  
**AI Role:** Architecture suggestions, boilerplate generation, option evaluation  
**Output:** `AGENTS.md` updated, `CONTEXT/[feature].md` created, task breakdown

Design decisions made with AI assistance must be:

- Understood and endorsed by a senior developer
- Documented in `AGENTS.md` or feature context file
- Not accepted solely because AI suggested them



### Phase 3 — Development (TDD + AI Generation Loop)

**Owner:** Developer (with Pair)  
**AI Role:** Code generation within test boundaries  
**Output:** Passing code + passing tests + updated context files

The TDD-AI loop is the core engine of PSDF:

```
1. Developer reads ACD
2. Developer writes failing test expressing one acceptance criterion
3. Developer feeds: context file + failing test → AI
4. AI generates implementation
5. Tests run — pass or fail is the only judge
6. Developer reviews generated code for understanding and correctness
7. Pair reviews against ACD (not just test pass)
8. Commit with PR description noting AI involvement
9. Repeat from step 2 for next criterion
```

**CI Pipeline requirements:**

- All tests must pass before merge — no exceptions
- PR template must include AI usage declaration
- Context file diff checked — if architecture changed without context update, PR is blocked


### Phase 4 — QA (BA-QA Validation)

**Owner:** QA (guided by BA)  
**AI Role:** Generate additional regression test cases, assist with test data  
**Output:** Validation sign-off against ACD

QA in PSDF validates against the **original ACD**, not just test coverage metrics. This closes the loop between what was asked (BA) and what was built (Dev + AI).

QA checklist addition for AI-assisted projects:

- [ ] Does output match ACD acceptance criteria exactly?
- [ ] Are there any behaviours present that are NOT in the ACD? (AI often adds unrequested logic)
- [ ] Were any known hallucination patterns from context files tested explicitly?

### Phase 5 — Retrospective (Team)

**Owner:** Team Lead  
**AI Role:** None — this is a human reflection step  
**Output:** Updated prompt library, updated context files, updated hallucination log

Retrospective additions for PSDF:

1.  What did AI generate correctly this sprint?
2.  What did AI hallucinate or get wrong?
3.  Which prompts worked reliably? Add to prompt library.
4.  Which context files need updating?
5.  Is the team's mental model shifting from deterministic to probabilistic?

## 2.3 Roles and Responsibilities

| Role | Traditional Responsibility | PSDF Addition |
| --- | --- | --- |
| Business Analyst | Requirements | Owns ACD as AI context source. Updates it. |
| Senior Developer | Architecture + review | Reviews AI output. Owns AGENTS.md. |
| Developer | Implementation | Writes tests before prompting AI. Documents AI usage in PRs. |
| QA  | Testing | Validates against ACD, not just test pass. Tests known hallucination patterns. |
| Team Lead | Delivery | Owns prompt library. Runs PSDF retrospective. |

## 2.4 Metrics

Traditional metrics remain. PSDF adds:

| Metric | What It Measures |
| --- | --- |
| AI Acceptance Rate | % of AI-generated code merged without modification |
| Hallucination Rate | Logged AI errors per sprint |
| Context File Freshness | Days since last context file update |
| Prompt Library Growth | New validated prompts added per sprint |
| ACD Coverage | % of acceptance criteria with corresponding tests |

## 2.5 Tool Stack (Frugal-First)
| Layer | Recommended Tool | Cost |
| --- | --- | --- |
| Local AI model | Ollama + DeepSeek Coder | Free |
| IDE AI integration | Continue.dev | Free |
| RAG over codebase | Continue.dev (built-in) | Free |
| Prompt library | Markdown files in repo | Free |
| Context files | AGENTS.md standard | Free |
| Fallback (complex reasoning) | Claude API | Pay per use |
| CI enforcement | GitHub Actions / GitLab CI | Free tier |

# SECTION 3: CLIENT-FACING FRAMEWORK

### For explaining and selling PSDF to clients

## Why Your Software Project Needs a Different Approach with AI

Most software teams today are using AI tools — ChatGPT, GitHub Copilot, and others — to write code faster. This is good. But there is a problem that most teams and most clients do not talk about openly:

**AI tools are not reliable in the way that traditional software tools are.**

A compiler either works or it doesn't. An AI tool might give you the right answer, a slightly wrong answer, or a confidently wrong answer — and it looks the same each time.

Without a framework to manage this, AI-assisted development creates hidden risks:

- Code that looks correct but has subtle bugs
- Features built against misunderstood requirements
- Knowledge that disappears when a developer leaves or a session ends
- Inconsistent behaviour that is hard to debug or reproduce

## What PSDF Does for Your Project

The Probabilistic-Safe Development Framework wraps AI assistance inside a set of verification layers so that the speed benefits of AI are captured without the reliability risks.

In plain terms:

**AI writes faster. Your team verifies smarter. You get both speed and confidence.**

## How It Works — In Plain Language

```
1. YOUR REQUIREMENTS ARE LOCKED FIRST
   A Business Analyst documents exactly what the software must do
   before any code is written. This document is the source of truth
   — not the AI, not the developer's memory.

2. TESTS ARE WRITTEN BEFORE CODE
   Developers write automated tests that define correct behaviour
   before asking AI to generate code. If AI gets it wrong,
   the test fails immediately. The bug never reaches you.

3. AI GENERATES, HUMANS VERIFY
   AI writes the code. A second developer reviews it.
   QA validates it against your original requirements.
   Three checkpoints before anything reaches production.

4. KNOWLEDGE IS STORED IN THE SYSTEM, NOT IN PEOPLE'S HEADS
   Every project decision, every AI failure, every working
   pattern is documented in the codebase itself.
   When a developer leaves or a session ends, nothing is lost.
```
   
## What This Means for You as a Client

| Your Concern | How PSDF Addresses It |
| --- | --- |
| "Will AI introduce bugs?" | Tests catch AI errors before merge. QA validates against your requirements. |
| "Will the team understand the code they ship?" | No AI code is merged unless the developer can explain it. |
| "What if requirements get lost or misunderstood?" | BA owns a living requirements document that feeds every AI session. |
| "How do I know AI was used responsibly?" | Every PR declares AI involvement. Metrics tracked per sprint. |
| "What if a developer leaves mid-project?" | Context files in the codebase mean any developer can resume with full AI context. |


## What PSDF Does Not Promise

We believe in being direct with clients.

PSDF does not eliminate AI errors — it contains them. AI will still occasionally produce wrong output. The framework ensures those errors are caught before they reach production, not that they never occur.

PSDF does not make AI development faster from day one. The first 4–6 weeks involve building context files, prompt libraries, and team habits. Velocity increases significantly after this ramp period.

## Summary

> PSDF treats AI as a powerful but probabilistic tool — and builds the human and process infrastructure to make it safe, auditable, and productive in a real software delivery context.

It is not a new methodology invented for AI. It is Test-Driven Development, Pair Programming, and the BA-QA model — proven approaches — assembled specifically to address what makes AI different from every other development tool your team has used before.


# APPENDIX

## A. Glossary

| Term | Definition |
| --- | --- |
| Probabilistic output | Output that varies across runs even with the same input |
| Hallucination | AI producing confident but factually incorrect output |
| Context file | Structured document fed to AI at session start to supply persistent memory |
| ACD | Acceptance Criteria Document — BA-owned source of truth |
| RAG | Retrieval Augmented Generation — retrieving relevant context chunks rather than uploading entire codebase |
| Prompt library | Version-controlled collection of validated prompts |
| PSDF loop | The TDD-AI generation cycle at the core of Phase 3 |


## B. Document Control
| Field | Value |
| --- | --- |
| Framework name | Probabilistic-Safe Development Framework |
| Version | 1.0 |
| Created | May 2026 |
| Review cycle | Every 6 months or after major AI tooling changes |
| Owner | \[Balaraman Sriram, Your Strategic Advisor] |

End of Document
