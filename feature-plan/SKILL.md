---
name: feature-plan
description: Grill the developer with open-ended clarifying questions about a code change, then produce a detailed, step-by-step implementation plan that a low-cost LLM or a junior human developer can execute without guessing, ending with a manual verification checklist where every step has an expected output. Use this whenever the user is about to implement a new feature, make a code change, fix a bug with unclear scope, or says things like "plan this", "grill me", "what am I missing", "write the plan", "make a plan a cheap model can follow", or "give me a checklist to test it". Trigger even when the user only describes a change in one or two sentences, because the grilling step is what catches the missing details.
---

# Feature Plan

This skill turns a vague code change into a plan that someone who has never seen the codebase can execute and verify. It works in three phases and does not write code until the plan is approved.

The reason for the structure: most implementation failures come from unstated assumptions (which file, which version, which environment, what "done" means). Phase 1 surfaces those assumptions. Phase 2 writes them down as explicit steps. Phase 3 gives the person a way to prove each step worked, so they never have to trust the output on faith.

The agent always has access to the code that will be changed. Read it before asking questions, so that questions are about real files, real patterns, and real behavior rather than guesses.

## Phase 1: Grill the developer

Goal: find every detail the plan would otherwise have to guess. The grilling is a free-form interview about this change. Ask whatever the situation needs; do not work through a fixed checklist. The areas below are only examples of things that often matter, not a scope limit.

Before asking anything:
- Read the user's description, any ticket, pasted code, and attached files.
- Read the relevant code in the repository, including nearby modules and existing patterns, so you know what is already true.
- Note what is already known. Do not ask for information that is already in front of you or in the code.

How to ask:
- Ask in rounds. Each round covers every question whose prerequisites are already answered. Never ask a question that depends on an answer you have not heard yet.
- Keep rounds focused. A round can be a few questions or many, depending on what is unclear.
- Prefer concrete questions, and where you can, give a recommended answer based on the code, so the developer can reply "yes" or "go with your recommendation".
- Ask about the things that most often break: existing behavior that will change, data shape and validity, failure and edge cases, where the code belongs, how it is deployed, and how it will be proven to work. Follow the thread wherever the answers lead.
- Push back on shallow or vague answers. If an answer leaves a gap that would change the code, ask again.
- Accept "I don't know" when it is honest, and record it as an open question or an assumption.

Rules for this phase:
- Stop when no unstated assumptions remain that would change the plan. Count rounds, not questions. Long interviews usually mean the scope is too large; say so and suggest splitting the change.
- If the user answers "use your judgment" to something, record the assumption you chose under **Assumptions**, so it is visible and can be overruled.
- If the user says "stop asking" or "just write it", write the plan now and list every unresolved question under **Open questions** at the top, so the executor knows what was guessed.
- If a question can only be answered by trying something (for example, how a screen should feel), say so, and propose a small throwaway prototype rather than continuing to ask.
- Do not start implementing during grilling.

## Phase 2: Write the implementation plan

Use this exact structure. Every step must be specific enough that the executor never has to decide what "update the service" means.

```
# Plan: <feature name>

## Summary
Two or three sentences: what changes, why, and what is out of scope.

## Assumptions
- Each assumption made during grilling, with the reason.

## Open questions
- Anything unresolved. Remove this section only if empty.

## Affected files
| File | Action (create / modify / delete) | Purpose |
|------|-----------------------------------|---------|

## Steps
### Step 1: <short action title>
- **File**: exact path
- **Change**: exact description. Include the signature, the field, the query, the config key, or the markup to add. Show a short code snippet when the change is not obvious from words.
- **Why**: one sentence, so the executor can judge edge cases.
- **Expected result**: what the file or system looks like after this step.

### Step 2: ...
(repeat for every step, in the order they must run)

## Dependencies and ordering
Which steps must finish before others, and any command to run between steps (build, migration, restore).

## Verification checklist
(see Phase 3)
```

Guidelines for writing steps:
- One step equals one reviewable change. If a step has "and" in the title twice, split it.
- Name exact paths, exact identifiers, and exact config keys. Never write "the appropriate file" or "the relevant service".
- Put commands in code blocks, marked with the working directory they run in.
- Order steps so the project still builds after each one where possible.
- Include the migration, configuration, and infrastructure steps explicitly, even when they feel minor. Those are the steps most often forgotten.
- Keep the plan in Markdown. Do not hide decisions in prose paragraphs; a step list is easier for a cheap model or a tired developer to follow.

## Phase 3: Manual verification checklist

Every plan ends with a checklist that a person can work through by hand. The checklist exists so that the developer confirms results with their own eyes, not by assuming the code is right.

Rules:
- Each check has a **Do** (the exact action to take) and an **Expect** (the exact observable output that means pass).
- Expected outputs must be observable: a specific value, a status code, a row count, a log line, a rendered label, a file that exists with specific content. "It works" is not an expected output.
- Include checks at three levels, in this order:
  1. **Build and static**: the project compiles with zero new warnings, and linters or Terraform `plan` show only the intended changes.
  2. **Unit or component**: the named tests pass, and any new test is listed by name.
  3. **End to end**: run the feature in the target environment and check the user-visible result, including at least one failure case that came up during grilling.
- Add a regression check for any existing behavior that the grilling identified as at risk of changing.
- Finish with a sign-off line: "All checks above passed. Date: ____ Tester: ____".

Format each check like this:

```
- [ ] **V1 — Project builds**
  - Do: `dotnet build src/Wafi.sln` from the repository root.
  - Expect: "Build succeeded" with 0 errors and no new warnings compared to the main branch.

- [ ] **V2 — Empty input is rejected**
  - Do: Submit the form with the name field empty.
  - Expect: The message "Name is required" appears under the field and no record is created in the table.
```

## Delivering the plan

- Save the plan as a Markdown file when the user asks for a file, or present it inline when they want to read it in the conversation. If the plan is long and will be shared or executed later, offer to save it as a document.
- End with one short line asking whether to adjust anything before the plan is used. Do not start implementing unless the user explicitly asks you to.

## Example opener

User: "I need a button on the invoice page that exports the list to Excel."

Good first response: read the invoice page and any existing export code, then ask the questions that the code and the request leave open, in rounds. Those might include which columns to export, whether the export respects the current filter, which Excel library the project already uses, whether it is core or client-specific, and what should happen with very large lists. Ask follow-ups as the answers reveal new gaps. Only after the grilling is complete, write the plan.
