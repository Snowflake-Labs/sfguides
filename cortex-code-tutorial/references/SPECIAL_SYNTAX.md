# Special Syntax Reference

Cortex Code has five special characters that act as shortcuts. Think of them like keyboard shortcuts in a word processor -- small symbols that save you a lot of time. Each one starts with a single character and tells Cortex Code to do something specific.

---

## The `@` Key -- Attach Files

**What it does:** Attaches a file to your message so the AI can read it.

Think of it like attaching a file to an email. Instead of copying and pasting the contents of a file into your message, you just point to it with `@` and Cortex Code pulls it in for you.

**How to use it:**

```
@path/to/file
```

After you type `@`, suggestions will pop up to help you find the file you want -- just like autocomplete on your phone.

**Selecting specific lines:**

You don't always need the whole file. You can zoom in on just the part you care about:

- `@file.py$10-50` -- show only lines 10 through 50
- `@file.py$10` -- show just line 10
- `@file.py$10-` -- show from line 10 to the end of the file

**Paths with spaces:**

If your file or folder name has spaces in it, wrap the path in curly braces:

```
@{my folder/my file.txt}
```

**Examples:**

- `@README.md` -- "Here's my README, can you summarize it?"
- `@src/app.py$1-20` -- "Look at the first 20 lines of this file"
- `Explain @config.json` -- Include a file as part of a larger question

---

## The `!` Key -- Run Shell Commands

**What it does:** Runs a terminal command and shows the output to the AI.

Think of it like telling your assistant "go run this and look at what comes back." The command runs on your computer, and the results get fed to the AI as context so it can help you with what it sees.

**How to use it:**

```
!command
```

**Examples:**

- `!ls` -- list the files in your current folder
- `!git status` -- check which files have changed recently
- `!python --version` -- find out which Python version is installed
- `!cat package.json` -- show the contents of a file

**Good to know:** The output goes to the AI as context, not just to your screen. So you can follow up with questions like "what do you see?" or "anything look wrong?"

---

## The `$` Key -- Activate Skills

**What it does:** Loads a skill -- a set of specialized instructions -- that makes the AI an expert in a particular area.

Think of it like picking a specialized tool from a toolbox. A wrench is great for bolts, a screwdriver is great for screws. Skills work the same way: each one gives the AI deeper knowledge about a specific topic.

**How to use it:**

```
$skill-name your request
```

**Examples:**

- `$cortex-code-guide how do I fork a session?`
- `$machine-learning help me train a model`
- `$skill-development create a new skill`

**Finding available skills:** Type `/skill` to see the full list of skills you can use.

**Important:** `$` is for skills, `/` is for slash commands. They look similar but do different things! Think of `$` as "activate an expert" and `/` as "run a menu action."

---

## The `#` Key -- Reference Snowflake Tables

**What it does:** Pulls in metadata about a Snowflake table -- things like column names, data types, and a few sample rows.

Think of it like showing someone a spreadsheet so they understand the shape of your data before you ask them a question about it. You don't send the whole table -- just enough for the AI to understand what it's working with.

**How to use it:**

```
#DATABASE.SCHEMA.TABLE
```

This requires a Snowflake connection to be active. After you type `#`, suggestions will appear to help you find the right table.

**What gets included:**

- Column names and their types (text, number, date, etc.)
- Primary keys
- Row count
- Up to 3 sample rows so the AI can see real data

**Examples:**

- `#MY_DB.PUBLIC.USERS` -- "Tell me about this table's structure"
- `Write a query to find active users in #MY_DB.PUBLIC.USERS`

---

## The `/` Key -- Slash Commands

**What it does:** Runs a special command that controls Cortex Code itself.

Think of it like a menu of built-in actions. While the other special characters send information to the AI, slash commands talk to Cortex Code directly.

**How to use it:**

```
/command
```

**Examples:** `/help`, `/status`, `/quit`, `/theme`

Slash commands have their own dedicated reference -- see **SLASH_COMMANDS.md** for the full list and details.

---

## Quick Reference Table

| Character | Name | What it does | Example |
|-----------|------|--------------|---------|
| `@` | File | Attach a file | `@README.md` |
| `!` | Shell | Run a command | `!git status` |
| `$` | Skill | Activate a skill | `$cortex-code-guide` |
| `#` | Table | Reference Snowflake data | `#DB.SCHEMA.TABLE` |
| `/` | Slash | Run a Cortex Code command | `/help` |

---

## Tips

- **Mix and match.** You can use several of these in one message: `Look at @app.py and tell me about #MY_DB.PUBLIC.USERS`
- **Autocomplete is your friend.** It works for `@` (files), `#` (tables), and `$` (skills). Just start typing and pick from the suggestions.
- **Commands run where you are.** `!` commands run in whatever folder you're currently working in.
- **Spaces in paths?** Wrap them in curly braces: `@{my folder/my file.txt}`
