# Introduction to Agent Skills

Course notes and progress for the Anthropic Academy "Introduction to Agent Skills" course.

## Exercises

---

## Lesson 1 — What are skills?

**Source:** https://anthropic.skilljar.com/introduction-to-agent-skills/434525

### What you'll learn

*Estimated time: 15 minutes*

By the end of this lesson you'll be able to:

- Define what Claude Code skills are and how they work
- Explain where skills live (personal vs. project directories)
- Distinguish between skills, CLAUDE.md, and slash commands
- Identify scenarios where skills are the right customization tool

---

### Video Summary *(3 minutes)*

This video introduces skills — reusable markdown files that teach Claude Code how to handle specific tasks automatically. Instead of repeating instructions every time you ask Claude to review a PR or write a commit message, you write a skill once and Claude applies it whenever the task comes up. The video covers what skills are, where they live, and how they compare to other Claude Code customization options.

**Key takeaways:**

- Skills are folders of instructions that Claude Code can discover and use to handle tasks more accurately. Each skill lives in a `SKILL.md` file with a name and description in its frontmatter.
- Claude uses the description to match skills to requests. When you ask Claude to do something, it compares your request against available skill descriptions and activates the ones that match.
- Personal skills go in `~/.claude/skills` and follow you across all projects. Project skills go in `.claude/skills` inside a repository and are shared with anyone who clones it.
- Skills load on demand — unlike CLAUDE.md (which loads into every conversation) or slash commands (which require explicit invocation), skills activate automatically when Claude recognizes the situation.
- If you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written.

---

### What Skills Are

Every time you explain your team's coding standards to Claude, you're repeating yourself. Every PR review, you re-describe how you want feedback structured. Every commit message, you remind Claude of your preferred format. Skills fix this.

A skill is a markdown file that teaches Claude how to do something once. Claude then applies that knowledge automatically whenever it's relevant.

Skills are folders of instructions and resources that Claude Code can discover and use to handle tasks more accurately. Each skill lives in a `SKILL.md` file with a name and description in its frontmatter.

The description is how Claude decides whether to use the skill. When you ask Claude to review a PR, it matches your request against available skill descriptions and finds the relevant one. Claude reads your request, compares it to all available skill descriptions, and activates the ones that match.

Here's what a skill's frontmatter looks like:

```
---
name: pr-review
description: Reviews pull requests for code quality. Use when reviewing PRs or checking code changes.
---
```

Below the frontmatter, you write the actual instructions — your review checklist, formatting preferences, or whatever Claude needs to know for that task.

---

### Where Skills Live

You can store skills in different places depending on who needs them:

- **Personal skills** go in `~/.claude/skills` (your home directory). These follow you across all your projects — your commit message style, your documentation format, how you like code explained.
- **Project skills** go in `.claude/skills` inside the root directory of your repository. Anyone who clones the repo gets these skills automatically. This is where team standards live, like your company's brand guidelines, preferred fonts, and colors for web design.

On Windows, personal skills live in `C:/Users/<your-user>/.claude/skills`.

Project skills get committed to version control alongside your code, so the whole team shares them.

---

### Skills vs. CLAUDE.md vs. Slash Commands

Claude Code has several ways to customize behavior. Skills are unique because they're automatic and task-specific. Here's how they compare:

- **CLAUDE.md** files load into every conversation. If you want Claude to always use TypeScript's strict mode, that goes in CLAUDE.md.
- **Skills** load on demand when they match your request. Claude only loads the name and description initially, so they don't fill up your entire context window. Your PR review checklist doesn't need to be in context when you're debugging — it loads when you actually ask for a review.
- **Slash commands** require you to explicitly type them. Skills don't. Claude applies them when it recognizes the situation.

When Claude matches a skill to your request, you'll see it load in the terminal.

---

### When to Use Skills

Skills work best for specialized knowledge that applies to specific tasks:

- Code review standards your team follows
- Commit message formats you prefer
- Brand guidelines for your organization
- Documentation templates for specific types of docs
- Debugging checklists for particular frameworks

The rule of thumb is simple: if you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written.

---

### Lesson Reflection

- Think about your most recent interactions with Claude Code. Which instructions did you find yourself repeating? How might a skill have saved you time?
- Consider your team's workflow. Which standards or processes would benefit most from being encoded as skills?

### What's Next

In the next lesson, you'll create your first skill from scratch and learn how Claude Code discovers, matches, and loads skills behind the scenes.

---

## Lesson 2 — Creating your first skill

**Source:** https://anthropic.skilljar.com/introduction-to-agent-skills/434527

### What you'll learn

*Estimated time: 20 minutes*

By the end of this lesson you'll be able to:

- Create a skill from scratch with proper frontmatter structure
- Test and verify that a skill loads correctly in Claude Code
- Explain how Claude Code matches incoming requests to available skills
- Describe the skill priority hierarchy (Enterprise, Personal, Project, Plugins)

---

### Video Summary *(4 minutes)*

This video walks through building a skill from scratch — a personal PR description skill that works across all your projects. You'll see exactly how to structure the `SKILL.md` file, test it, and understand how Claude Code discovers and matches skills to your requests. The video also covers the priority hierarchy that determines which skill wins when names conflict.

**Key takeaways:**

- A skill is a directory containing a `SKILL.md` file with metadata (name, description) in frontmatter and instructions below.
- Claude loads only skill names and descriptions at startup, then matches incoming requests against those descriptions using semantic matching.
- You get a confirmation prompt before Claude loads the full skill content into context.
- Priority for name conflicts: **Enterprise → Personal → Project → Plugins**.
- To update a skill, edit its `SKILL.md`. To remove one, delete its directory. Always restart Claude Code for changes to take effect.

---

### Creating a Skill

We'll build a personal skill that teaches Claude how to write PR descriptions in a consistent format. Since it's a personal skill, it lives in your home directory and works across all your projects.

First, create a directory for your skill inside the skills folder. The directory name should match your skill name:

```bash
mkdir -p ~/.claude/skills/pr-description
```

Then create a `SKILL.md` file inside that directory. The file has two parts separated by frontmatter dashes:

```
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR, writing a PR, or when the user asks to summarize changes for a pull request.
---

When writing a PR description:

1. Run `git diff main...HEAD` to see all changes on this branch
2. Write a description following this format:

## What
One sentence explaining what this PR does.

## Why
Brief context on why this change is needed

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
```

The **name** identifies your skill. The **description** tells Claude when to use it — this is the matching criteria. Everything after the second set of dashes is the instructions Claude follows when the skill is activated.

---

### Testing Your Skill

Claude Code loads skills at startup, so restart your session after creating one. You can verify it's available by checking the available skills list. You should see your skill listed.

To test it, make some changes on a branch and say something like "write a PR description for my changes." Claude will indicate it's using the PR description skill, check your diff, and write a description following your template — same format every time.

---

### How Skill Matching Works

When Claude Code starts, it scans four locations for skills but only loads the name and description — not the full content. This is an important detail.

When you send a request, Claude compares your message against the descriptions of all available skills. For example, "explain what this function does" would match a skill described as "explain code with visual diagrams" because the intent overlaps.

Once a match is found, Claude asks you to confirm loading the skill. This confirmation step keeps you aware of what context Claude is pulling in. After you confirm, Claude reads the complete `SKILL.md` file and follows its instructions.

---

### Skill Priority

If you clone a repository that has a skill with the same name as one of your personal skills, which one wins? There's a clear priority order:

1. **Enterprise** — managed settings, highest priority
2. **Personal** — your home directory (`~/.claude/skills`)
3. **Project** — the `.claude/skills` directory inside a repository
4. **Plugins** — installed plugins, lowest priority

This lets organizations enforce standards through enterprise skills while still allowing individual customization. If your company has an enterprise "code-review" skill and you create a personal "code-review" skill with the same name, the enterprise version takes precedence.

To avoid conflicts, use descriptive names. Instead of just "review," use something like "frontend-review" or "backend-review."

---

### Updating and Removing Skills

To update a skill, edit its `SKILL.md` file. To remove one, delete its directory. Restart Claude Code after any changes for them to take effect.

---

### Lesson Reflection

- What's one task in your daily workflow that you could turn into a skill right now? What would the description look like?
- How might the priority hierarchy affect your team's skill management strategy? Would you rely more on personal or project-level skills?

### What's Next

In the next lesson, you'll learn about advanced configuration options including metadata fields, tool restrictions with `allowed-tools`, and how to structure larger skills using progressive disclosure and multi-file organization.

---

## Lesson 3 — Configuration and multi-file skills

**Source:** https://anthropic.skilljar.com/introduction-to-agent-skills/434526

### What you'll learn

*Estimated time: 20 minutes*

By the end of this lesson you'll be able to:

- Configure advanced skill metadata fields including `allowed-tools` and `model`
- Write effective skill descriptions that reliably trigger on the right requests
- Use `allowed-tools` to restrict what Claude can do when a skill is active
- Organize complex skills using progressive disclosure and multi-file structures

---

### Video Summary *(4 minutes)*

This video covers the advanced techniques that make skills more powerful: the full set of metadata fields, how to write descriptions that trigger reliably, restricting tool access for security-sensitive workflows, and organizing larger skills across multiple files for complex use cases.

**Key takeaways:**

- `name` and `description` are required — `allowed-tools` and `model` are optional but powerful additions.
- A good description answers two questions: What does the skill do? When should Claude use it?
- `allowed-tools` restricts which tools Claude can use when the skill is active — useful for read-only or security-sensitive workflows.
- Progressive disclosure: keep `SKILL.md` under 500 lines and link to supporting files (references, scripts, assets) that Claude reads only when needed.
- Scripts execute without loading their contents into context — only the output consumes tokens, keeping context efficient.

---

### Skill Metadata Fields

The agent skills open standard supports several fields in the `SKILL.md` frontmatter. Two are required, and the rest are optional:

- **`name`** *(required)* — Identifies your skill. Use lowercase letters, numbers, and hyphens only. Maximum 64 characters. Should match your directory name.
- **`description`** *(required)* — Tells Claude when to use the skill. Maximum 1,024 characters. This is the most important field because Claude uses it for matching.
- **`allowed-tools`** *(optional)* — Restricts which tools Claude can use when the skill is active.
- **`model`** *(optional)* — Specifies which Claude model to use for the skill.

---

### Writing Effective Descriptions

Be explicit with your instructions. If someone told you "your job is to help with docs," you wouldn't know what to do — and Claude thinks the same way.

A good description answers two questions:

1. What does the skill do?
2. When should Claude use it?

If your skill isn't triggering when you expect it to, try adding more keywords that match how you actually phrase your requests. The description is what Claude uses to decide whether a skill is relevant, so the language matters.

---

### Restricting Tools with `allowed-tools`

Sometimes you want a skill that can only read files, not modify them. This is useful for security-sensitive workflows, read-only tasks, or any situation where you want guardrails.

In this example, the `allowed-tools` field is set to `Read, Grep, Glob, Bash`. When this skill is active, Claude can only use those tools without asking permission — no editing, no writing.

```
---
name: codebase-onboarding
description: Helps new developers understand how the system works.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

If you omit `allowed-tools` entirely, the skill doesn't restrict anything. Claude uses its normal permission model.

---

### Progressive Disclosure

Skills share Claude's context window with your conversation. When Claude activates a skill, it loads the contents of that `SKILL.md` into context. But sometimes you need references, examples, or utility scripts that the skill depends on.

Cramming everything into one 2,000-line file has two problems: it takes up a lot of context window space, and it's harder to maintain.

Progressive disclosure solves this. Keep essential instructions in `SKILL.md` and put detailed reference material in separate files that Claude reads only when needed.

The open standard suggests organizing your skill directory with:

- `scripts/` — Executable code
- `references/` — Additional documentation
- `assets/` — Images, templates, or other data files

Then in `SKILL.md`, link to the supporting files with clear instructions about when to load them. Claude reads a reference file (e.g. `architecture-guide.md`) only when the request actually requires it — if someone is asking where to add a component, that file never loads. It's like having a table of contents in the context window rather than the entire document.

A good rule of thumb: keep `SKILL.md` under 500 lines. If you're exceeding that, consider whether the content should be split into separate reference files.

---

### Using Scripts Efficiently

Scripts in your skill directory can run without loading their contents into context. The script executes and only the output consumes tokens. The key instruction to include in your `SKILL.md` is to tell Claude to run the script, not read it.

This is particularly useful for:

- Environment validation
- Data transformations that need to be consistent
- Operations that are more reliable as tested code than generated code

---

### Lesson Reflection

- Think about a skill you'd like to build that involves multiple files. How would you structure the `SKILL.md` versus supporting reference files?
- Are there workflows in your team where restricting tool access with `allowed-tools` would add an important safety layer?

### What's Next

In the next lesson, we'll compare skills to the other ways you can customize Claude Code — CLAUDE.md, subagents, hooks, and MCP servers — so you can choose the right tool for each situation.

