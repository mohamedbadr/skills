# skills

A collection of reusable skills for AI assistants. Each skill lives in its own folder with a `SKILL.md` file.

## Available skills

| Skill | Description |
|-------|-------------|
| [feature-plan](feature-plan/SKILL.md) | Interviews you about a code change with open-ended questions, then writes a step-by-step implementation plan a low-cost LLM or junior developer can follow. Ends with a manual verification checklist where every step has an expected output. |

## Installation

### Claude Code

Install for all your projects:

```bash
mkdir -p ~/.claude/skills
cp -r feature-plan ~/.claude/skills/
```

Or for a single project, copy it into `<project>/.claude/skills/`. Claude loads the skill automatically when a request matches its description.

### Claude.ai (web, desktop, mobile)

1. Zip the skill folder, so the archive contains `feature-plan/SKILL.md`.
2. In Claude, open **Settings → Capabilities → Skills** and upload the zip. (Menu names may change, so check the current help docs if you can't find it.)

### Claude API

Skills can be uploaded and referenced through the API. See the Skills documentation at https://docs.claude.com for the current steps.

### Other LLMs (ChatGPT, Gemini, Copilot, Cursor, and similar)

These tools don't read `SKILL.md` files directly, so use the file's content as instructions:

1. Open `SKILL.md` and copy everything below the `---` frontmatter block.
2. Paste it into the tool's custom instructions, for example:
   - **ChatGPT**: a Project's instructions or a Custom GPT's instructions.
   - **Gemini**: a Gem's instructions.
   - **Cursor**: save it as `.cursor/rules/feature-plan.mdc` in your project.
   - **Other tools**: the system prompt or project rules field.
3. Since chat-only tools can't open your repository, paste the relevant code into the conversation so the grilling questions are based on real code.

## Usage

Describe the change you want to make, for example:

> Plan this change: add an Excel export button to the invoice page.

The assistant asks its questions in rounds, then produces the plan and checklist. Review the plan before any code is written.

## Adding a new skill

1. Create a folder named after the skill, with a `SKILL.md` inside.
2. Start `SKILL.md` with frontmatter that has a `name` and a `description`. The description decides when the skill is used, so state what it does and when to use it.
3. Add a row to the table above and a short install note if it needs special steps.
