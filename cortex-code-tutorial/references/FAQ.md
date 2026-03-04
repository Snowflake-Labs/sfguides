# Frequently Asked Questions

A plain-language guide to the most common questions about Cortex Code.

---

## Getting Started

### What is Cortex Code?

Cortex Code is an AI-powered assistant that runs right inside your terminal. You type messages in everyday language, and it helps you with coding, data analysis, file management, and much more. Think of it as a knowledgeable coworker sitting next to you -- one who never gets tired of answering questions.

### How do I install Cortex Code?

Head over to the official installation guide for step-by-step instructions:
<https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli>

The guide walks you through everything you need to get up and running.

### How do I start it?

Open your terminal and type:

```
cortex
```

Then press **Enter**. That's it -- Cortex Code will start up and wait for your first message.

### How do I exit?

You have a few options:

- Type `/quit` or `/q` and press Enter.
- Press **Ctrl+C** to exit immediately.

### Is Cortex Code free?

Cortex Code requires a Snowflake account. If you're not sure whether you have access, check with your Snowflake administrator -- they can let you know what's available to you.

---

## Using Cortex Code

### What can I ask it to do?

Pretty much anything you'd ask a helpful coworker! Here are some examples:

- **Write code** -- "Write a Python function that reads a CSV file."
- **Explain code** -- "What does this function do?" (attach the file with `@`)
- **Fix bugs** -- "This script is throwing an error, can you help?"
- **Search files** -- "Find all files that mention 'database connection'."
- **Run commands** -- "Run the tests and tell me what failed."
- **Analyze data** -- "Query this Snowflake table and summarize the results."
- **Write documentation** -- "Write a docstring for this function."

If you're not sure whether it can help, just ask! The worst that happens is it tells you it can't.

### Can it see my files?

Yes. Cortex Code can read files in the folder where you started it (and all subfolders). It's like opening a project folder -- everything inside is visible.

You can also explicitly show it a specific file by typing `@filename`. For example:

```
@src/main.py What does this file do?
```

### Can it change my files?

Yes, but don't worry -- in the default mode, it **asks your permission first** before making any changes. You're always in control. You'll see a preview of what it wants to change, and you can approve or reject each one.

### What if it makes a mistake?

No problem! You have several safety nets:

- **Tell it directly** -- Just say "that's wrong, try again" or "undo that."
- **Use `/rewind`** -- This rolls back recent messages, like an undo button for the conversation.
- **Use git** -- If it changed files and you want to revert, standard `git checkout` or `git restore` commands work as usual.

Mistakes are normal and easy to fix. Don't be afraid to experiment.

### How do I give it more context?

The more context you provide, the better the help you'll get. Here are your options:

- **`@file`** -- Attach a file so it can read the contents. Example: `@config.yaml Can you explain this?`
- **`!command`** -- Run a shell command and show the output. Example: `!cat error.log What's going wrong here?`
- **Plain language** -- Just explain what you're working on, what you've tried, and what you expect. The AI is good at understanding natural descriptions.

Think of it like talking to a coworker -- the more background you give, the more useful the answer.

---

## Slash Commands

### What's the difference between `/` and `$`?

Good question -- they look similar but do different things:

- **`/` (slash commands)** control Cortex Code itself. They're like buttons on a remote control -- `/quit` exits, `/rewind` undoes things, `/compact` cleans up the conversation.
- **`$` (skills)** activate specialized instruction sets. They're like calling in a specialist -- `$cortex-agent` brings in expertise about building Cortex Agents, for example.

A simple way to remember: `/` is for **actions**, `$` is for **expertise**.

### How do I see all available commands?

Type:

```
/commands
```

This shows you every slash command you can use, with a short description of each one.

### Can I create my own slash commands?

Yes! It's simpler than you might think:

1. Create a markdown file (`.md`) with the instructions you want.
2. Put it in one of these folders:
   - **Project-level** (just for this project): `.cortex/commands/`
   - **Global** (available everywhere): `~/.snowflake/cortex/commands/`

The filename becomes the command name. So a file called `deploy.md` becomes the `/deploy` command.

---

## Skills

### What is a skill?

A skill is a set of instructions that tells the AI how to handle a specific type of task. Think of it like a **recipe card** -- it gives the AI step-by-step guidance for a particular job, so it doesn't have to figure everything out from scratch.

For example, there's a skill for building Streamlit apps, one for working with dbt projects, one for managing data governance, and many more.

### How do I use a skill?

Type `$` followed by the skill name, then your request. For example:

```
$cortex-code-guide how do sessions work?
```

The AI will load that skill's instructions and use them to give you a better answer.

### How do I see what skills are available?

Type:

```
/skill
```

This opens the skill manager, where you can browse all available skills and see what each one does.

### What's a bundled skill?

A bundled skill is one that comes **pre-installed** with Cortex Code. You don't need to download, create, or configure anything -- it's ready to use right away. Most of the skills you'll use are bundled.

### What's a sub-skill?

Some skills are like departments in a company -- they contain smaller, specialized skills inside them. When you use the parent skill, it automatically figures out which sub-skill you need based on your question.

For example, the `data-governance` skill contains sub-skills for access history, masking policies, and data classification. You just ask your question, and it routes you to the right specialist.

### Can I create my own skill?

Absolutely! Here's the basic idea:

1. Create a folder for your skill.
2. Inside it, create a file called `SKILL.md` with the instructions.
3. Put the folder in one of these locations:
   - **Project-level**: `.cortex/skills/your-skill-name/`
   - **Global**: `~/.snowflake/cortex/skills/your-skill-name/`

The folder name becomes the skill name, and the `SKILL.md` file contains the instructions the AI will follow.

---

## Sessions

### Can I come back to a conversation later?

Yes! Your conversations are saved automatically. To pick up where you left off:

- **Resume your last session**: `cortex --resume last`
- **Browse and choose a session**: `cortex resume`

It's like bookmarking a chat -- everything is right where you left it.

### How do I name a session?

Type:

```
/rename my-session-name
```

Give it a meaningful name so you can find it later. For example: `/rename fixing-login-bug` or `/rename data-migration-project`.

### What does `/fork` do?

Fork creates a **copy** of your conversation from a point you choose. It's like "Save As" for conversations.

This is useful when you want to try a different approach without losing your current progress. You keep the original conversation and get a new branch to experiment with.

### What does `/compact` do?

Compact **summarizes** your conversation and clears the detailed history. Think of it like cleaning off your desk -- the important notes are kept, but the clutter is removed.

Use it when:
- Your conversation has been going on for a while.
- The AI seems to be slowing down.
- You want a fresh start without losing all context.

---

## Modes

### What mode should I use?

Start with **Confirm Actions** -- it's the default and the safest option. In this mode, the AI asks for your permission before making any changes to your files or running commands. It's like having training wheels on.

Once you're comfortable and doing simpler tasks, you can switch to **Bypass mode**, where the AI acts more independently without asking for confirmation each time.

### What's plan mode for?

Plan mode lets the AI **look around and make suggestions** without actually changing anything. It's like asking an architect to sketch ideas before you start building.

It's great for:
- Exploring a codebase you're not familiar with.
- Understanding how something works before changing it.
- Getting the AI's recommendation on how to approach a task.

### How do I know what mode I'm in?

Type:

```
/status
```

This shows you your current mode, along with other useful information about your session.

---

## Troubleshooting

### Cortex Code won't start.

Try these steps in order:

1. **Check if it's installed**: Run `cortex --help`. If you see a help message, it's installed. If you get "command not found," follow the [installation guide](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli).
2. **Run diagnostics**: Once inside Cortex Code, type `/doctor`. This checks for common problems and suggests fixes.
3. **Check your connection**: Make sure you have a valid Snowflake connection configured.

### It's running slowly.

Long conversations can slow things down, just like a really long email chain gets hard to follow. Try:

```
/compact
```

This summarizes the conversation and gives the AI a fresh, shorter context to work with. It usually speeds things right up.

### It's not reading my files.

A few things to check:

1. **Are you in the right folder?** Cortex Code can only see files in the directory where you started it. Use `!ls` to see what's visible.
2. **Try attaching the file directly**: Use `@filename` to explicitly point it to the file you want.
3. **Check the file path**: Make sure the filename is spelled correctly, including any folder paths.

### A skill isn't working.

Here's a quick checklist:

1. **Is the skill available?** Type `/skill` to see all installed skills.
2. **Are you using the right prefix?** Skills use `$`, not `/`. So it's `$skill-name`, not `/skill-name`.
3. **Is the name correct?** Skill names use hyphens, not spaces. For example: `$cortex-code-guide`, not `$cortex code guide`.

---

## Tips and Tricks

Here are some handy shortcuts and features that can make your life easier:

| What you want | What to do |
|---|---|
| Quick help | Type `?` for instant help |
| Multi-line messages | Press **Ctrl+J** to add a new line (Enter submits your message) |
| Search prompt history | Press **Ctrl+R** to search through previous things you've typed |
| Cycle display modes | Press **Ctrl+O** to switch between different display layouts |
| Customize colors | Type `/theme` to pick a color scheme you like |
| Diagnose problems | Type `/doctor` to run a health check |

---

*Still have questions? Just ask Cortex Code directly -- type your question in plain language and it will do its best to help!*
