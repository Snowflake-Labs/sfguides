# Cortex Code Tutorial -- Lesson Plans

This document contains 7 structured lessons for teaching non-technical users how to use Cortex Code CLI from scratch. Each lesson builds on the previous one. Follow them in order.

---

## Lesson 1: Getting Started

**Learning Objective:** Launch Cortex Code and learn how to find help.

### Teaching Steps

**Step 1 -- Explain what a terminal is.**

Start by making sure the user understands where they'll be working. Explain it like this:

> A terminal is the text-based window on your computer where you can type commands. Think of it like texting your computer -- you type something, press Enter, and your computer responds. On a Mac, you can open an app called "Terminal" (search for it in Spotlight). On Windows, you can use "PowerShell" or "Command Prompt" (search for either in the Start menu).

If the user seems unsure, reassure them that they don't need to be a programmer to use it. We're just going to type simple things and see what happens.

**Step 2 -- Launch Cortex Code.**

Have the user type `cortex` and press Enter. Explain:

> This starts Cortex Code. It's like opening a chat window -- but instead of chatting with a person, you're chatting with an AI assistant that can help you with work on your computer.

Wait for the user to confirm they see the Cortex Code prompt.

**Step 3 -- Explain what they see.**

Walk them through the interface:

> You should see a prompt where you can type messages. This is your conversation with Cortex Code. Anything you type here, the AI reads and responds to. It's a back-and-forth conversation, just like texting.

**Step 4 -- Try a simple message.**

Have the user type something like:

```
Hello, what can you do?
```

Let them see the response. Point out that Cortex Code explains its capabilities. This builds confidence -- they just had their first interaction.

**Step 5 -- Introduce `/help`.**

Tell the user:

> Now let's learn about getting help. Type `/help` and press Enter.

Explain that commands starting with `/` are special commands. They don't go to the AI as a message -- instead, they control Cortex Code itself. Think of them like pressing buttons on a remote control, rather than talking to someone.

**Step 6 -- Introduce `/status`.**

Have them type `/status`. Explain:

> This shows you information about your current session -- what AI model is running, what connections are active, and other details. It's like checking the dashboard of your car to see how things are running.

**Step 7 -- Introduce `?`.**

Have them type `?` by itself. Explain:

> This is a quick-help shortcut. It shows a brief overview of what you can do. Think of it as the "cheat sheet" version of help.

**Step 8 -- Introduce quitting.**

Explain how to exit:

> When you're done, type `/quit` or `/q` to exit Cortex Code. It's like closing an app. Don't worry -- you can always come back. We'll learn about resuming sessions later.

Don't have them quit yet if there are more lessons to do.

### What the User Should Try

- Type `cortex` to launch
- Type `Hello, what can you do?`
- Type `/help`
- Type `/status`
- Type `?`

### Checkpoint Question

Ask the user: **"What do you type to get help inside Cortex Code?"**

Expected answer: `/help` (also accept `?` or mentioning both).

### Summary

You learned how to open Cortex Code, have a conversation with it, and find help when you need it. The key things to remember are: type messages to chat with the AI, and type `/help` anytime you're lost.

---

## Lesson 2: Slash Commands

**Learning Objective:** Learn how slash commands work and try the most useful ones.

### Teaching Steps

**Step 1 -- Explain what slash commands are.**

> Slash commands are special instructions that start with `/`. They're like a menu of actions you can take. Instead of talking to the AI, slash commands control Cortex Code itself -- things like clearing the screen, switching modes, or checking settings. Think of it this way: regular messages are you talking *to* the AI. Slash commands are you pressing buttons *on* the app.

**Step 2 -- Show the categories.**

Walk through the main groups so the user gets a mental map:

- **Session commands** -- control your conversation:
  - `/quit` -- exit Cortex Code
  - `/clear` -- clear the screen (like wiping a whiteboard)
  - `/new` -- start a fresh conversation
  - `/rename` -- give your session a name
- **Information commands** -- get information:
  - `/help` -- detailed help (you already know this one!)
  - `/status` -- check what's going on
  - `/commands` -- see every available command
- **Mode commands** -- change how Cortex Code behaves:
  - `/plan` -- switch to planning-only mode
  - `/bypass` -- let the AI act without asking permission
  - `/model` -- change which AI model is being used
- **Configuration commands** -- open interactive settings screens:
  - `/settings` -- general settings
  - `/skill` -- manage skills (more on this in Lesson 4)
  - `/mcp` -- manage external tool connections
  - `/theme` -- change the color scheme
  - `/connections` -- manage Snowflake connections
- **Development commands** -- do development work:
  - `/sh` -- open a shell/terminal
  - `/sql` -- run SQL queries
  - `/diff` -- see code changes
  - `/tasks` -- view and manage tasks
- **Utility commands** -- other useful tools:
  - `/agents` -- see background agents
  - `/feedback` -- send feedback to the team
  - `/doctor` -- check your environment for issues
  - `/update` -- update Cortex Code

Don't overwhelm them -- just show the categories and a few examples from each. Reassure them they don't need to memorize anything.

**Step 3 -- Try `/commands`.**

Have them type `/commands`. Explain:

> This shows you every single slash command available. It's your complete menu. Whenever you forget a command, just type `/commands` and browse the list.

**Step 4 -- Try `/theme`.**

Have them type `/theme`. Explain:

> This lets you change the color scheme of Cortex Code. It's visual and fun -- pick one you like! This is a good example of a "configuration command" that opens an interactive screen.

This step is intentionally light and fun to keep engagement high.

**Step 5 -- Try `/clear`.**

Have them type `/clear`. Explain:

> This clears everything on your screen. Your conversation history isn't lost -- it's just not displayed anymore. Think of it like cleaning your desk. The files are still in the drawer.

**Step 6 -- Reassure about memorization.**

> You absolutely do not need to memorize all these commands. Remember: `/commands` always shows the full list. As you use Cortex Code more, you'll naturally remember the ones you use most.

**Step 7 -- Introduce `/doctor`.**

Have them type `/doctor`. Explain:

> This checks your environment for issues -- like going to the doctor for a checkup. It looks at things like whether your tools are installed correctly and your connections are working. If something's wrong, it tells you what to fix.

### What the User Should Try

- Type `/commands`
- Type `/theme` and pick a color scheme
- Type `/clear`
- Type `/doctor`

### Checkpoint Question

Ask the user: **"If you wanted to see every available slash command, what would you type?"**

Expected answer: `/commands`

### Summary

You learned that slash commands are special instructions starting with `/` that control Cortex Code itself. They're organized into categories like session, information, modes, configuration, development, and utilities. The most important thing to remember: `/commands` shows you the complete list whenever you need it.

---

## Lesson 3: Special Syntax -- The Four Power Keys

**Learning Objective:** Learn the four special characters that give you superpowers in Cortex Code.

### Teaching Steps

**Step 1 -- Introduce the concept.**

> Cortex Code has four special characters that do special things when you use them. Think of them as keyboard shortcuts that unlock extra powers. Each one starts with a single character and lets you pull in outside information or trigger special behavior.

**Step 2 -- The `@` key (The File Key).**

Explain:

> Type `@` followed by a filename to attach that file to your message. It's like attaching a file to an email -- you're saying "hey AI, look at this file."
>
> **Example:** `@README.md` attaches your README file so the AI can read it and answer questions about it.
>
> You can even specify line ranges if you only want to show part of a file: `@src/app.py$10-50` shows only lines 10 through 50. That's handy for big files.
>
> **Bonus:** After you type `@`, autocomplete kicks in and suggests files for you. So you don't need to remember exact filenames -- just start typing and pick from the list.

**Step 3 -- The `!` key (The Shell Key).**

Explain:

> Type `!` followed by a command to run that command in your terminal and show the results to the AI. It's like telling the AI "go run this and look at what comes back."
>
> **Example:** `!git status` runs the `git status` command and feeds the output into your conversation. The AI can then help you understand the results or act on them.
>
> This is useful when you want the AI to see real, live information from your computer -- like the output of a build, a list of files, or an error message.

**Step 4 -- The `$` key (The Skill Key).**

Explain:

> Type `$` followed by a skill name to activate a skill. Skills are like specialized tools -- we'll cover them in depth in the next lesson.
>
> **Example:** `$cortex-code-guide how do I resume a session?` activates the Cortex Code guide skill and asks it a question.
>
> Think of it like picking a specialized tool from a toolbox. Instead of the AI using its general knowledge, it follows a specific, expert workflow.

**Step 5 -- The `#` key (The Table Key).**

Give a brief mention:

> Type `#` followed by a Snowflake table name to pull in that table's metadata -- its columns, data types, and sample data.
>
> **Example:** `#MY_DB.PUBLIC.USERS` shows the AI what the USERS table looks like so it can help you write queries or understand the data.
>
> It's like showing someone a spreadsheet so they understand what data you're working with. Note: this requires a Snowflake connection to be set up.

**Step 6 -- Mention `/` as a bonus.**

> Technically, `/` is a fifth special character -- it's for the slash commands you already learned in Lesson 2. So now you know all five: `@`, `!`, `$`, `#`, and `/`.

**Step 7 -- Have them try it out.**

Have the user try these:

- Type `@` and observe the autocomplete suggestions that appear. Pick a file and send a message about it.
- Type `!ls` (on Mac/Linux) or `!dir` (on Windows) to list files in the current directory. Point out that the AI now sees the file listing.
- Explain that they'll try `$` in the next lesson when they learn about skills.

### What the User Should Try

- Type `@` and explore the autocomplete
- Type `!ls` (Mac/Linux) or `!dir` (Windows)
- Observe the results appearing in the conversation

### Checkpoint Question

Ask the user: **"If you wanted to show Cortex Code the contents of a file called `notes.txt`, what would you type?"**

Expected answer: `@notes.txt`

### Summary

You learned the four power keys: `@` attaches files, `!` runs shell commands, `$` activates skills, and `#` pulls in Snowflake table metadata. Together with `/` for slash commands, these five characters are your shortcuts to getting the most out of Cortex Code. You don't need to memorize them all right now -- just remember they exist, and come back to this lesson when you need a refresher.

---

## Lesson 4: Understanding Skills

**Learning Objective:** Learn what skills are, where they come from, and how to use them.

### Teaching Steps

**Step 1 -- What is a skill?**

> A skill is a set of instructions that tells Cortex Code how to handle a specific type of task. Think of it like a recipe card -- it gives the AI a step-by-step guide for a particular job. Without a recipe, a cook improvises. With a recipe, they follow a tested, reliable process. Skills work the same way.

**Step 2 -- Why skills matter.**

> Without a skill, Cortex Code uses its general knowledge to help you. That's fine for many things. But for specific, repeatable tasks -- like setting up a Streamlit app, debugging a data pipeline, or creating a machine learning model -- a skill gives the AI a tested workflow to follow. Skills make the AI more reliable and consistent.

**Step 3 -- How to use a skill.**

> Type `$` followed by the skill name, then your request.
>
> **Example:** `$cortex-code-guide how do I resume a session?`
>
> This activates the `cortex-code-guide` skill and asks it your question. The AI will follow the skill's instructions to give you a better answer than it might with general knowledge alone.

**Step 4 -- Where skills live (the four locations).**

Explain the four places skills can come from, in order of priority:

1. **Project skills** (`.cortex/skills/` in your project folder):
   > These are specific to the project you're working in. They get shared with your team through git, so everyone on the project has the same skills. If you're working in a team project, your team might have created skills for common tasks.

2. **Global skills** (`~/.snowflake/cortex/skills/`):
   > These are available everywhere on your computer, no matter what project you're in. They're personal to you. Good for skills you use across many projects.

3. **Remote skills** (configured in `skills.json`):
   > These are downloaded from the internet -- like from GitHub. Someone else created them and shared them publicly. Your `skills.json` configuration file tells Cortex Code where to find them.

4. **Bundled skills** (shipped with Cortex Code):
   > These come pre-installed with Cortex Code. They're always available, no setup needed. Examples include `cortex-code-guide`, `machine-learning`, `developing-with-streamlit`, and many others.

**Step 5 -- Bundled vs Local vs Remote.**

Clarify the distinction:

> - **Bundled** = came pre-installed with Cortex Code. You didn't create them. They're maintained by the Cortex Code team.
> - **Local** = skills you (or your team) created and stored in your project folder or your global skills folder.
> - **Remote** = skills pulled from a URL, like a GitHub repository. Someone else made them and shared them.

**Step 6 -- Sub-skills.**

> Some skills are actually collections of smaller, specialized skills. For example, the `machine-learning` skill routes you to specialized sub-skills for different ML tasks -- training models, deploying them, monitoring performance, and so on. Think of it like a department in a company that has specialized teams inside it. You ask the department, and it sends you to the right team.

**Step 7 -- Exploring skills.**

> Type `/skill` to open the interactive skill manager. It shows all available skills, where they come from (bundled, local, remote), and lets you add or remove them. It's your skill library.

**Step 8 -- Have them explore.**

Have the user type `/skill` and browse the available skills. Point out a few interesting ones and explain what they do.

### What the User Should Try

- Type `/skill` and browse the skill list
- Try activating a bundled skill, e.g. `$cortex-code-guide what slash commands are available?`

### Checkpoint Question

Ask the user: **"What's the difference between a bundled skill and a local skill?"**

Expected answer: A bundled skill comes pre-installed with Cortex Code, while a local skill is one that you or your team created and stored in your project or global skills folder.

### Summary

You learned that skills are instruction sets that guide Cortex Code through specific tasks. They come from four places: your project folder, your global folder, remote URLs, or bundled with Cortex Code. You activate them with `$`, and you can browse all available skills with `/skill`. Skills make the AI more reliable for specialized work.

---

## Lesson 5: Creating Your Own Skill

**Learning Objective:** Build a simple skill from scratch to understand how they work.

### Teaching Steps

**Step 1 -- Demystify skill creation.**

> Creating a skill is just creating a text file with a specific format. No coding required. If you can write a document, you can create a skill.

**Step 2 -- Explain the structure.**

> A skill is a folder with at least one file called `SKILL.md`. That's it. Here's what the folder looks like:
>
> ```
> my-skill/
> ├── SKILL.md              # Required: the instructions
> └── references/            # Optional: extra reference docs
>     ├── GUIDE.md
>     └── FAQ.md
> ```
>
> The `SKILL.md` file is the brain of the skill -- it tells the AI what to do. The `references/` folder is optional and holds any extra documents the AI might need to reference while using the skill.

**Step 3 -- Explain the two parts of SKILL.md.**

> The `SKILL.md` file has two parts:
>
> 1. **Frontmatter** -- the header between `---` lines. This is metadata about the skill: its name and description.
> 2. **Body** -- everything after the frontmatter. These are the actual instructions the AI follows.
>
> Think of it like a recipe: the frontmatter is the title and description on the card, and the body is the step-by-step cooking instructions.

**Step 4 -- Walk through creating a skill together.**

Guide the user through building a "meeting-notes" skill:

> Let's create a skill together. We'll make one that helps format meeting notes.
>
> First, we need to create the folder. The skill will live in your project's `.cortex/skills/` directory:
>
> ```
> mkdir -p .cortex/skills/meeting-notes
> ```
>
> Now, create a file called `SKILL.md` inside that folder with this content:
>
> ```markdown
> ---
> name: meeting-notes
> description: "Helps format and organize meeting notes. Use when the user wants to create, clean up, or summarize meeting notes."
> ---
>
> # Meeting Notes Skill
>
> You help the user create well-organized meeting notes.
>
> ## Format
>
> Always structure meeting notes with:
> 1. **Date and attendees** at the top
> 2. **Agenda items** as headers
> 3. **Key decisions** highlighted in bold
> 4. **Action items** as a checklist at the bottom
>
> ## Workflow
> 1. Ask the user what the meeting was about
> 2. Ask who attended
> 3. Help them organize the notes into the format above
> 4. Offer to save the notes to a file
> ```

**Step 5 -- Explain the frontmatter fields.**

> Let's break down the frontmatter:
>
> - **`name`** -- a short identifier for the skill. Use lowercase letters and dashes, no spaces. This is what you type after `$` to activate it.
> - **`description`** -- explains when this skill should be used. Include trigger words and phrases so the AI (and other users) know what it's for. The more descriptive, the better.

**Step 6 -- Test the skill.**

> Now let's test it! Type `$meeting-notes` followed by a request, like:
>
> ```
> $meeting-notes I had a team standup this morning and need to write it up
> ```
>
> The AI should now follow the meeting notes workflow you defined -- asking about the date, attendees, and helping organize everything into the format you specified.

**Step 7 -- Explain references.**

> If your skill needs extra knowledge -- like a style guide, FAQ, or technical documentation -- you can put markdown files in a `references/` folder inside your skill. The AI can read these files when answering questions.
>
> ```
> meeting-notes/
> ├── SKILL.md
> └── references/
>     └── STYLE_GUIDE.md    # Extra reference the AI can use
> ```
>
> Think of it like giving someone their instructions AND a reference manual. The instructions tell them what to do, and the reference manual helps them do it well.

**Step 8 -- Mention the interactive skill creator.**

> You can also use the `/skill` manager to help create skills interactively. When you open `/skill`, look for the option to add a new skill. It walks you through the process step by step.

### What the User Should Try

- Create the `meeting-notes` skill folder and `SKILL.md` file (the teaching agent can help them do this)
- Test it with `$meeting-notes`
- Browse `/skill` to see their new skill listed

### Checkpoint Question

Ask the user: **"What is the one required file in every skill?"**

Expected answer: `SKILL.md`

### Summary

You learned how to create a skill from scratch. A skill is just a folder with a `SKILL.md` file. The file has frontmatter (name and description) and a body (the instructions). You can optionally add reference documents in a `references/` folder. Once created, you activate your skill with `$` followed by the skill name. Creating skills is one of the most powerful things you can do in Cortex Code -- it lets you teach the AI exactly how you want tasks done.

---

## Lesson 6: Managing Your Conversations

**Learning Objective:** Learn how to save, resume, branch, and undo conversations.

### Teaching Steps

**Step 1 -- Explain sessions.**

> Every time you use Cortex Code, you're in a "session." Think of it like a document you're working on. Everything you say and everything the AI responds with is saved in that session. When you close Cortex Code, the session is saved automatically -- like auto-save in a word processor.

**Step 2 -- Naming sessions with `/rename`.**

> By default, your session gets an auto-generated name. But you can give it a descriptive name to find it later:
>
> ```
> /rename my-project-work
> ```
>
> It's like saving a document with a descriptive filename instead of "Untitled-1." When you have lots of sessions, good names make it easy to find the one you want.

**Step 3 -- Resuming sessions.**

> When you close Cortex Code and come back later, you can pick up right where you left off:
>
> - **`cortex --resume last`** -- reopens your most recent session. Like reopening the last document you were working on.
> - **`cortex resume`** -- shows you a list of all your past sessions so you can pick one. Like browsing your recent documents.
> - **`/resume`** -- if you're already inside Cortex Code, this opens the session picker without leaving.
>
> The AI remembers everything from the session -- what you discussed, what files you worked on, what decisions were made. It's like the AI has perfect notes from your last conversation.

**Step 4 -- Forking with `/fork`.**

> Sometimes you want to try a different approach without losing your current work. That's what forking is for:
>
> ```
> /fork my-experiment
> ```
>
> This creates a copy of your conversation at the current point. You get a new session that starts with everything from the original, and the original stays untouched. It's like "Save As" for your conversation.
>
> **Use case:** You've been working on a feature and want to try a completely different approach. Fork the session, experiment freely, and if it doesn't work out, your original session is still there.

**Step 5 -- Rewinding with `/rewind`.**

> Sometimes the AI goes down a wrong path -- maybe it misunderstood you or made changes you didn't want. Instead of starting over, you can rewind:
>
> - **`/rewind`** -- opens an interactive picker where you choose how far back to go
> - **`/rewind 3`** -- goes back 3 messages
>
> It's like pressing "Undo" multiple times. The messages are removed from the conversation, and you can try again from that point.
>
> **Use case:** The AI started refactoring code you didn't want changed. Rewind to before it started, then give clearer instructions.

**Step 6 -- Compacting with `/compact`.**

> When your conversation gets very long, Cortex Code can slow down or run out of room (there's a limit to how much text the AI can process at once). Compacting fixes this:
>
> ```
> /compact
> ```
>
> This summarizes everything said so far and clears the history, keeping only the key points. It's like writing a summary of a long meeting so you have a shorter document to work from.
>
> You can even give it focus instructions:
>
> ```
> /compact focus on the database changes
> ```
>
> This tells the AI what to prioritize in the summary.

**Step 7 -- Have them try it.**

Have the user try:

- `/rename` to name their current session (pick something descriptive)
- Explain that `/fork` and `/rewind` are best tried during real work, but they now know they exist

### What the User Should Try

- Type `/rename my-tutorial-session` (or any name they like)
- Understand that `/fork`, `/rewind`, and `/compact` are available when needed

### Checkpoint Question

Ask the user: **"If the AI went down a wrong path and you want to undo the last few messages, what command do you use?"**

Expected answer: `/rewind`

### Summary

You learned how to manage your conversations like documents. Name them with `/rename`, resume them later with `cortex --resume last` or `cortex resume`, branch them with `/fork`, undo mistakes with `/rewind`, and keep them manageable with `/compact`. Your sessions are automatically saved, so you never lose your work.

---

## Lesson 7: Safety and Modes

**Learning Objective:** Understand how to control what Cortex Code is allowed to do.

### Teaching Steps

**Step 1 -- Why modes matter.**

> Cortex Code is powerful -- it can read files, write files, run commands, and make changes on your computer. That's great when you want it to get things done. But sometimes you want more control. Maybe you just want it to look at things without touching anything. Or maybe you trust it completely and want it to move fast. Modes let you control this.

**Step 2 -- The three modes.**

Walk through each mode with an analogy:

> **1. Confirm Actions (the default mode)**
>
> In this mode, Cortex Code asks your permission before doing anything significant -- like editing a file or running a command. It shows you exactly what it wants to do, and you decide whether to allow it.
>
> Think of it like a coworker who checks with you before making changes: "Hey, I'd like to edit this file -- is that okay?"
>
> This is the safest mode and the one you should use most of the time.

> **2. Plan Mode**
>
> In this mode, Cortex Code can only look at things and make plans. It cannot make any changes. It can read files and explore your codebase, but it won't edit anything, create anything, or run any commands that modify things.
>
> Think of it like asking an architect to draw a blueprint but not build anything yet. You get to review the plan before any work begins.
>
> - Type `/plan` to turn it on
> - Type `/plan-off` to turn it off
> - Or press `Ctrl+P` to toggle it quickly

> **3. Bypass Mode**
>
> In this mode, Cortex Code can do everything without asking permission. It reads, writes, runs commands, and makes changes -- all on its own.
>
> Think of it like telling your coworker "I trust you, just get it done." It's fast, but use it with care.
>
> - Type `/bypass` to turn it on
> - Type `/bypass-off` to turn it off

**Step 3 -- Switching modes quickly.**

> You can press `Shift+Tab` to quickly cycle between Confirm Actions and Bypass mode. It's a fast toggle for when you want to switch back and forth.

**Step 4 -- When to use each mode.**

Give practical guidance:

> - **Confirm Actions** -- use this most of the time. Especially when working on important files or unfamiliar codebases. You stay in control.
> - **Plan Mode** -- use this when you want to explore ideas safely. Great for reviewing code, understanding a project, or planning a big change before committing to it. Nothing can go wrong in plan mode because nothing gets changed.
> - **Bypass Mode** -- use this for repetitive, low-risk work where you trust the AI's judgment. For example, formatting many files, renaming variables across a project, or running a series of well-understood commands. Switch back to Confirm Actions when you're done.

**Step 5 -- Permissions in Confirm Actions mode.**

> When Cortex Code is in Confirm Actions mode and wants to do something, it shows you exactly what it plans to do and asks for permission. You have three choices:
>
> 1. **Allow** -- let it do this one thing
> 2. **Deny** -- don't let it do this
> 3. **Allow for session** -- let it do this type of action for the rest of the session without asking again
>
> This gives you fine-grained control. You can approve safe actions and block anything you're not sure about.

**Step 6 -- Checking your current mode.**

> You can always check what mode you're in by looking at the Cortex Code interface or typing `/status`. The current mode is displayed so you always know how the AI is behaving.

**Step 7 -- Have them try it.**

Have the user try:

- Type `/plan` to enter plan mode
- Ask the AI to do something (like "create a new file called test.txt") and observe that it plans but doesn't execute
- Type `/plan-off` to go back to normal
- Try pressing `Shift+Tab` to see the mode switch

### What the User Should Try

- Type `/plan` and then ask the AI to make a change -- observe that it only plans
- Type `/plan-off` to return to normal mode
- Press `Shift+Tab` to toggle between Confirm Actions and Bypass mode
- Type `/status` to see the current mode

### Checkpoint Question

Ask the user: **"Which mode would you use if you want Cortex Code to explore your codebase without changing anything?"**

Expected answer: Plan Mode (or `/plan`).

### Summary

You learned about the three modes that control what Cortex Code is allowed to do: Confirm Actions (asks permission -- the safe default), Plan Mode (look but don't touch), and Bypass Mode (full speed, no questions asked). Use Confirm Actions most of the time, Plan Mode for safe exploration, and Bypass Mode when you trust the AI and want to move fast. You can switch modes with slash commands or keyboard shortcuts.

---

## Quick Reference: All Lessons at a Glance

| Lesson | Topic | Key Takeaway |
|--------|-------|-------------|
| 1 | Getting Started | Launch with `cortex`, get help with `/help` |
| 2 | Slash Commands | `/commands` shows everything, `/` controls the app |
| 3 | Special Syntax | `@` = files, `!` = shell, `$` = skills, `#` = tables |
| 4 | Understanding Skills | Skills are expert instruction sets, activated with `$` |
| 5 | Creating Skills | A skill is a folder with a `SKILL.md` file |
| 6 | Managing Conversations | `/rename`, `/resume`, `/fork`, `/rewind`, `/compact` |
| 7 | Safety and Modes | Confirm Actions, Plan Mode, and Bypass Mode |
