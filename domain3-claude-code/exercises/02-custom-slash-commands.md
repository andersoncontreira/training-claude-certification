# Exercise 02: Custom Slash Commands

**Difficulty:** Intermediate
**Domain:** Claude Code
**Goal:** Create project-specific slash commands that automate common development workflows.

---

## Context

Custom slash commands are Markdown files in `.claude/commands/`. When invoked, Claude reads the file and executes its instructions. They can accept arguments via `$ARGUMENTS`.

---

## Task

Create 3 custom slash commands for a Python project:

1. `/review` — reviews the current git diff for bugs and style issues
2. `/test-plan <file>` — generates a test plan for a given file
3. `/standup` — generates a daily standup summary from recent git activity

---

## Part 1: /review

Create `.claude/commands/review.md`:

```markdown
TODO: Write a prompt that tells Claude to:
1. Run `git diff` to see what changed
2. Review the changes for: bugs, edge cases, and code style issues
3. Output findings in a structured format: Summary / Issues / Suggestions
```

Test it by making a small change to any file, then running `/review`.

---

## Part 2: /test-plan

Create `.claude/commands/test-plan.md`:

```markdown
TODO: Write a prompt that uses $ARGUMENTS as the file path and tells Claude to:
1. Read the specified file
2. Identify all public functions/methods
3. Generate a test plan with: happy path, edge cases, and error cases for each function
4. Format as a checklist
```

Test with: `/test-plan main.py`

---

## Part 3: /standup

Create `.claude/commands/standup.md`:

```markdown
TODO: Write a prompt that tells Claude to:
1. Run `git log --oneline --since="yesterday" --author="$(git config user.name)"`
2. Summarize the commits into standup format:
   - What I did yesterday
   - What I'm doing today (infer from the most recent commits)
   - Blockers: none (unless something in the logs suggests otherwise)
```

---

## Part 4: Test and Refine

Run each command and evaluate:

| Command | Does it work? | What needs improving? |
|---------|--------------|----------------------|
| `/review` | | |
| `/test-plan main.py` | | |
| `/standup` | | |

Iterate on the prompts in the `.md` files until the output quality meets your standards.

---

## Part 5: Add a Global Command

Global commands live in `~/.claude/commands/` and are available in every project.

Create `~/.claude/commands/explain-error.md`:

```markdown
TODO: Write a prompt that takes $ARGUMENTS (an error message or stack trace)
and explains: what the error means, what likely caused it, and how to fix it.
Format: plain language, no jargon, actionable steps.
```

Test with: `/explain-error TypeError: 'NoneType' object is not subscriptable`

---

## Key Design Principles for Slash Commands

1. **Be specific about output format** — "output as a numbered list" is better than "list the issues"
2. **Reference concrete tools** — "run `git diff`" is clearer than "look at the changes"
3. **Use `$ARGUMENTS` for flexibility** — commands that accept input are more reusable
4. **Keep commands focused** — one command, one purpose

---

## Extension Challenges

1. Create a `/changelog` command that generates a changelog entry from commits since the last git tag.
2. Create a `/security-check` command that reviews the current diff for common security issues (SQL injection, hardcoded secrets, etc).
3. Create a `/refactor <function>` command that reads a function by name and suggests refactoring improvements.

---

## Key Concepts Practiced

- `.claude/commands/` file structure
- `$ARGUMENTS` placeholder
- Global (`~/.claude/commands/`) vs. project-level commands
- Iterative prompt refinement
