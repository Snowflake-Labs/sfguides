# Slash Commands Reference

Slash commands are shortcuts you type into Cortex Code to do things quickly. Think of them like keyboard shortcuts in a word processor -- instead of clicking through menus, you just type a short command starting with `/`.

You don't need to memorize any of these. If you ever forget, just type `/commands` and Cortex Code will show you the full list.

---

## Session Commands

These commands help you manage your conversations -- starting new ones, going back to old ones, or tidying things up.

### `/quit` or `/q`
**What it does:** Closes Cortex Code and exits back to your regular terminal.

**When to use it:** When you're done working and want to leave.

---

### `/clear`
**What it does:** Clears the screen so it looks clean and empty. Think of it like wiping a whiteboard -- your conversation history is still saved, it just looks tidier.

**When to use it:** When your screen feels cluttered and you want a fresh visual start without losing anything.

---

### `/new`
**What it does:** Starts a brand new conversation from scratch. It's like opening a blank document -- your old conversation is saved, but you're starting fresh.

**When to use it:** When you want to switch to a completely different topic or task.

---

### `/fork`
**What it does:** Creates a copy of your current conversation from a specific point, like using "Save As" on a document. The original stays untouched, and you get a new branch to take in a different direction.

**When to use it:** When the conversation is going well but you want to explore a "what if" without messing up what you already have.

---

### `/resume`
**What it does:** Opens a list of your past conversations so you can pick one up where you left off. Think of it like a "Recent Files" menu.

**When to use it:** When you closed Cortex Code earlier and want to continue a previous conversation.

---

### `/rename <name>`
**What it does:** Gives your current conversation a name so it's easier to find later. Like saving a file with a meaningful name instead of "Untitled."

**When to use it:** When you're working on something important and want to find this conversation easily later.

**Example:** `/rename fixing the login bug` -- now this conversation is labeled "fixing the login bug" in your history.

---

### `/rewind [N]`
**What it does:** Undoes recent messages in your conversation. It's like pressing Ctrl+Z (undo) in a text editor. You can optionally say how many messages to undo.

**When to use it:** When the conversation went in a wrong direction and you want to go back to an earlier point and try again.

**Example:** `/rewind 3` -- undoes the last 3 exchanges.

---

### `/compact [instructions]`
**What it does:** Summarizes your conversation to make it shorter, freeing up space for Cortex Code to think. Imagine your conversation is a long email thread -- this command writes a "summary so far" and clears the rest.

You can optionally add instructions to tell it what to focus on in the summary.

**When to use it:** When your conversation has been going on for a while and Cortex Code starts to feel slow or forgetful.

**Example:** `/compact focus on the database changes we discussed` -- summarizes the conversation with emphasis on database topics.

---

## Information Commands

These commands help you find out what's going on -- what tools are available, what mode you're in, and how to get help.

### `/help`
**What it does:** Shows the help menu with useful information about how to use Cortex Code.

**When to use it:** Anytime you're unsure what to do or how something works.

---

### `/status`
**What it does:** Shows you the current state of things -- which AI model you're using, which Snowflake connection is active, what mode you're in, and other session details. Think of it like a dashboard or status bar.

**When to use it:** When you want to check what's going on behind the scenes.

---

### `/commands`
**What it does:** Lists every slash command available to you right now, including any custom ones you've created.

**When to use it:** When you can't remember a command name and want to browse the full list.

---

## Mode Commands

Modes change how Cortex Code behaves. Think of them like gears in a car -- different modes for different situations.

### `/plan`
**What it does:** Puts Cortex Code into "plan mode." In this mode, it can look at your code and think about what to do, but it won't actually change anything. It's like asking an architect to draw blueprints before building.

**When to use it:** When you want Cortex Code to come up with a plan first so you can review it before any changes are made.

---

### `/plan-off`
**What it does:** Turns off plan mode, letting Cortex Code make changes again.

**When to use it:** After you've reviewed a plan and you're ready to let Cortex Code start working.

---

### `/bypass`
**What it does:** Puts Cortex Code into "bypass mode." Normally, Cortex Code asks for your permission before doing things like running commands or editing files. In bypass mode, it does everything without asking. Think of it like giving someone your full trust to just get the job done.

**When to use it:** When you trust the task is safe and you don't want to keep clicking "approve" over and over.

---

### `/bypass-off`
**What it does:** Turns off bypass mode, so Cortex Code goes back to asking for permission before making changes.

**When to use it:** When you want to regain control and review actions before they happen.

---

### `/model <name>`
**What it does:** Switches which AI model Cortex Code uses to think and respond. Different models have different strengths -- some are faster, some are more thorough.

**When to use it:** When you want to try a different model for your current task.

**Example:** `/model haiku` -- switches to the Haiku model, which is faster but less detailed.

---

## Configuration Commands

These commands open interactive screens where you can change settings. You don't need to memorize options -- just run the command and follow the prompts on screen.

### `/settings`
**What it does:** Opens the settings editor where you can customize how Cortex Code behaves.

**When to use it:** When you want to tweak preferences like default behavior, permissions, or other options.

---

### `/mcp`
**What it does:** Opens a screen to manage MCP (Model Context Protocol) server connections. MCP servers give Cortex Code extra abilities by connecting to outside tools and services.

**When to use it:** When you want to connect Cortex Code to additional tools or services.

---

### `/skill`
**What it does:** Opens a screen to view, add, or remove skills. Skills are like recipes that teach Cortex Code how to handle specific types of tasks.

**When to use it:** When you want to see what skills are available or add new ones.

---

### `/hooks`
**What it does:** Opens a screen to manage hooks -- automated actions that run when certain things happen. Think of hooks like automatic rules: "whenever X happens, also do Y."

**When to use it:** When you want to set up automatic checks or actions (like running a linter every time code is saved).

---

### `/theme`
**What it does:** Lets you change the color theme of Cortex Code. Pick the one that's easiest on your eyes.

**When to use it:** When you want a different look -- lighter, darker, or a different color scheme.

---

### `/connections`
**What it does:** Opens a screen to manage your Snowflake database connections. You can add new connections, switch between them, or edit existing ones.

**When to use it:** When you need to connect to a different Snowflake account or change connection details.

---

## Development Commands

These commands are for working with code, running queries, and managing your development environment.

### `/add-dir <path>`
**What it does:** Tells Cortex Code to also look at files in another folder. By default, it only sees the folder you started in. This expands its view.

**When to use it:** When you need Cortex Code to work with files in a different folder than where you started.

**Example:** `/add-dir ../other-project` -- now Cortex Code can also see and work with files in that other project folder.

---

### `/sh <cmd>`
**What it does:** Runs a shell (terminal) command directly from within Cortex Code. It's a shortcut so you don't have to switch back to your regular terminal.

**When to use it:** When you need to quickly run a terminal command without leaving Cortex Code.

**Example:** `/sh git status` -- shows the current state of your git repository.

---

### `/sql <query>`
**What it does:** Runs a SQL query on your connected Snowflake database and shows the results.

**When to use it:** When you want to quickly check something in your database.

**Example:** `/sql SELECT COUNT(*) FROM my_table` -- counts the rows in "my_table."

---

### `/table`
**What it does:** Opens the most recent SQL results in a fullscreen, scrollable table view. Much easier to read than the default inline output.

**When to use it:** After running a SQL query, when you want a better view of the results.

---

### `/diff`
**What it does:** Opens a viewer showing your uncommitted code changes (things you've changed but haven't saved to git yet). Think of it like a "Track Changes" view in a word processor.

**When to use it:** When you want to review what's been changed before committing.

---

### `/tasks`
**What it does:** Shows you any background tasks that Cortex Code is currently running. Sometimes it works on multiple things at once, and this lets you see what's happening.

**When to use it:** When you want to check on work that's running in the background.

---

### `/worktree`
**What it does:** Manages git worktrees, which let you work on multiple branches of your code at the same time in separate folders.

**When to use it:** When you need to work on different code branches simultaneously.

---

### `/sandbox`
**What it does:** Configures sandbox settings. The sandbox is a safety boundary that controls what Cortex Code can and can't do on your system.

**When to use it:** When you need to adjust what Cortex Code is allowed to access.

---

## Utility Commands

Helpful tools for maintenance, diagnostics, and extra features.

### `/fdbt`
**What it does:** Opens DBT (data build tool) operations for working with data transformation projects.

**When to use it:** When you're working with a DBT project and need to run DBT-related tasks.

---

### `/lineage <model>`
**What it does:** Shows you a visual map of how data flows through your models -- where it comes from and where it goes. Think of it like a family tree for your data.

**When to use it:** When you want to understand the relationships between your data models.

**Example:** `/lineage my_model` -- shows what feeds into "my_model" and what depends on it.

---

### `/agents`
**What it does:** Shows you any AI sub-agents that are currently running. Sub-agents are helpers that Cortex Code spins up to work on specific parts of a task.

**When to use it:** When you want to see what helper agents are active and what they're doing.

---

### `/setup-jupyter`
**What it does:** Sets up Jupyter notebook integration so Cortex Code can work with `.ipynb` notebook files.

**When to use it:** When you want to use Jupyter notebooks with Cortex Code for the first time.

---

### `/feedback`
**What it does:** Opens a form to send feedback directly to the Cortex Code team. Bug reports, feature requests, or just thoughts -- they all help.

**When to use it:** When something isn't working right, or when you have a suggestion.

---

### `/clear-cache`
**What it does:** Clears cached (temporarily stored) data. Sometimes old cached data can cause issues, and this gives you a fresh start.

**When to use it:** When something seems stale or not updating properly.

---

### `/doctor`
**What it does:** Runs a health checkup on your environment. It checks that everything is set up correctly and flags any issues it finds. Think of it like taking your car in for a diagnostic scan.

**When to use it:** When something isn't working and you're not sure why. Start here.

---

### `/update`
**What it does:** Updates Cortex Code to the latest version so you have the newest features and bug fixes.

**When to use it:** When you want to make sure you're running the most recent version.

---

## Custom Slash Commands

You're not limited to the built-in commands -- you can create your own! A custom slash command is just a markdown file (`.md`) that contains instructions for Cortex Code. The file name becomes the command name.

### Where to put them

- **For a specific project:** Put `.md` files in your project's `.cortex/commands/` folder. These commands will only be available when you're working in that project.
- **For everywhere:** Put `.md` files in `~/.snowflake/cortex/commands/` (in your home directory). These commands will be available no matter which project you're in.

### How it works

If you create a file called `.cortex/commands/review.md`, you can then type `/review` in Cortex Code and it will follow the instructions in that file.

This is a great way to save workflows you repeat often -- like a code review checklist, a deployment procedure, or a standard way to set up a new feature.

---

## Tips

- **You don't need to memorize these.** Just type `/commands` to see the full list anytime.
- **Slash commands are not case-sensitive.** `/Help`, `/help`, and `/HELP` all work the same way.
- **Configuration commands open interactive screens.** You don't need to know all the options in advance -- just run the command and follow the prompts.
- **`/doctor` is your best friend when things break.** If something isn't working, run `/doctor` first. It usually points you in the right direction.
- **`/status` is a quick sanity check.** Not sure which database you're connected to or what model you're using? `/status` tells you everything.
