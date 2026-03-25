# Skills Guide

## What is a Skill?

A **skill** is a set of instructions that tells Cortex Code how to handle a specific type of task.

Think of it like a **recipe card** for the AI. Without a recipe, a cook might improvise -- sometimes that works, sometimes it doesn't. But with a good recipe, the cook follows tested steps and gets a reliable result every time.

Skills work the same way:

- **Without a skill**, Cortex Code uses its general knowledge. It might give you a perfectly fine answer, or it might miss important details specific to your situation.
- **With a skill**, Cortex Code follows a tested, step-by-step workflow designed for that exact type of task.

The bottom line: skills make Cortex Code more **reliable** and **consistent** for the tasks you care about most.

---

## How to Use a Skill

Using a skill is simple. Type a **dollar sign** (`$`) followed by the skill name, then your request.

**Example:**

```
$cortex-code-guide how do sessions work?
```

That's it! The `$` prefix is what activates the skill -- it tells Cortex Code "use this specific set of instructions to answer my question."

### Skills vs. Slash Commands

This is a common point of confusion, so let's clear it up:

- **`$` is for skills** -- sets of instructions that guide how the AI responds
- **`/` is for commands** -- actions that do something immediately (like `/help` or `/clear`)

They look similar but they do very different things. Just remember: **dollar sign for skills, slash for commands**.

---

## Where Skills Come From -- The Four Locations

Skills can live in four different places. When you invoke a skill, Cortex Code checks these locations **in order** and uses the **first match** it finds.

### 1. Project Skills (highest priority)

**Where:** The `.cortex/skills/` folder (or `.claude/skills/`) inside your project

**What they're for:** Instructions that are specific to *this* project and shared with your teammates through git.

**Analogy:** Like a team playbook that sits in the team's shared folder. Everyone on the team uses the same plays.

**Best for:** Team workflows, project-specific deploy steps, coding standards your team follows.

### 2. Global Skills

**Where:** `~/.snowflake/cortex/skills/` on your computer

**What they're for:** Your personal skills that work across all your projects.

**Analogy:** Like your own personal notebook of tips and tricks that you carry from job to job.

**Best for:** Personal productivity shortcuts, your preferred coding patterns, workflows you use everywhere.

### 3. Remote Skills

**Where:** Downloaded from URLs (like GitHub repositories), configured in `skills.json`

**What they're for:** Skills shared across teams or organizations that stay up to date automatically.

**Analogy:** Like subscribing to an expert's newsletter -- you always get the latest version of their advice.

**Best for:** Company-wide standards, community-shared workflows, skills that need frequent updates.

### 4. Bundled Skills (lowest priority)

**Where:** Built right into Cortex Code itself

**What they're for:** General-purpose skills that are always available, no setup needed.

**Analogy:** Like the built-in apps on your phone -- they're there from day one, ready to use.

**Best for:** Common tasks like working with machine learning, managing data governance, building Streamlit apps, and more.

**Examples:** `cortex-code-guide`, `machine-learning`, `skill-development`, `developing-with-streamlit`

### Why the Order Matters

Because Cortex Code checks these locations in order (project, then global, then remote, then bundled), **you can override any skill** by creating one with the same name at a higher-priority location. For instance, if you don't like how a bundled skill works, you can create your own version in your project's `.cortex/skills/` folder and Cortex Code will use yours instead.

---

## Bundled vs. Local vs. Remote -- A Quick Comparison

| Type | Where it lives | Who made it | Example |
|------|---------------|-------------|---------|
| Bundled | Inside Cortex Code itself | The Snowflake team | `cortex-code-guide` |
| Local (project) | `.cortex/skills/` in your project | You or your team | A custom deploy skill |
| Local (global) | `~/.snowflake/cortex/skills/` | You | A personal productivity skill |
| Remote | Downloaded from a URL | Anyone | A shared community skill |

---

## Sub-Skills

Some skills are actually **collections of smaller, specialized skills** working together under one name.

**Analogy:** Think of a hospital. You don't walk in and say "I need the cardiology sub-hospital." You walk into the hospital, describe your symptoms, and the hospital routes you to the right department -- cardiology, radiology, orthopedics, etc.

Sub-skills work the same way:

- The **parent skill** is the front door. You invoke it with your question.
- The parent skill figures out what you need and **routes you to the right sub-skill** automatically.
- You **don't need to know which sub-skill to call** -- just talk to the parent.

### Examples

- **`machine-learning`** -- You say "help me train a model" and it routes to the training sub-skill. Say "deploy my model" and it routes to the deployment sub-skill.
- **`data-governance`** -- Ask about "who has access to this table" and it routes to the access history sub-skill. Ask about "mask this column" and it routes to the masking policy sub-skill. Ask about "find PII in my data" and it routes to the data classification sub-skill.

The takeaway: just invoke the parent skill and describe what you need. It handles the rest.

---

## The Skill File Structure

Every skill lives in its own folder and follows a simple structure:

```
my-skill/
├── SKILL.md               # Required: the main instructions
└── references/             # Optional: extra knowledge files
    ├── GUIDE.md
    └── FAQ.md
```

- **`SKILL.md`** is the only file you absolutely need. It contains the instructions Cortex Code follows.
- **`references/`** is an optional folder where you can put additional documentation, guides, FAQs, or any other information the skill might need to consult while working.

Think of `SKILL.md` as the main recipe, and the `references/` folder as the cookbook it can flip through for extra details.

---

## The SKILL.md File

The `SKILL.md` file has two parts:

### Part 1: Frontmatter (the metadata)

At the very top of the file, between two lines of three dashes (`---`), you put metadata about the skill:

```yaml
---
name: my-skill
description: "What it does. Use when: situations. Triggers: keywords."
---
```

This section tells Cortex Code *about* the skill -- its name, what it does, and when to use it.

### Part 2: Body (the actual instructions)

Everything after the frontmatter is the body. This is written in Markdown and contains the step-by-step instructions that Cortex Code follows when the skill is activated.

You can think of the frontmatter as the **label on the outside of the recipe card** (title, category, when to use it), and the body as the **actual recipe inside**.

### Frontmatter Fields

| Field | Required? | What it does |
|-------|-----------|-------------|
| `name` | Yes | A short identifier for the skill. Use dashes instead of spaces (e.g., `my-cool-skill`). |
| `description` | Yes | Explains what the skill does and when to use it. Include trigger words so Cortex Code knows when to suggest it. |
| `tools` | No | Lists which tools the skill is allowed to use. If omitted, the skill can use all available tools. |

### Writing a Good Description

The `description` field is important because it helps Cortex Code decide when to activate or suggest your skill. A good description includes:

- **What the skill does** -- a plain summary
- **When to use it** -- the situations it's designed for
- **Trigger words** -- specific keywords that should activate this skill

**Example:**
```yaml
description: "Deploys the application to staging or production. Use when: deploying, releasing, pushing to prod. Triggers: deploy, release, ship, push to staging."
```

---

## Managing Skills

Cortex Code has a built-in skill manager. Type:

```
/skill
```

This opens an interactive view that shows you **all your skills** -- where they come from (bundled, project, global, or remote), what they do, and lets you:

- **View** existing skills and their details
- **Add** new skills from various sources
- **Create** brand new skills from scratch
- **Remove** skills you no longer need

It's the easiest way to see everything at a glance and manage your skill collection.

---

## Tips for Creating Great Skills

Here are some practical tips if you're building your own skills:

- **Keep it under 500 lines.** Shorter skills perform better. If your skill is getting long, consider breaking it into a parent skill with sub-skills.

- **Include clear stopping points.** Build in places where the skill should pause and ask the user a question before continuing. This keeps the user in control and prevents the AI from going off in the wrong direction.

- **Put trigger keywords in the description.** This is how Cortex Code knows when to suggest your skill. Be generous with keywords -- include synonyms, abbreviations, and related terms.

- **Use the `references/` folder for detailed docs.** Keep your `SKILL.md` focused on instructions. Put detailed reference material, FAQs, and guides in the `references/` folder where the skill can look them up as needed.

- **Test your skill.** After creating it, type `$your-skill-name` and try a few different requests to make sure it handles them well. Try edge cases too -- what happens if someone asks something slightly outside the skill's scope?
