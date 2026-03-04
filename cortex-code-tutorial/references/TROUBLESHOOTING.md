# Troubleshooting Guide

Something not working? Don't worry -- most problems have simple fixes. This guide is organized by **what you're seeing**, so just find your symptom and follow the steps.

Think of this guide like a "What's wrong with my car?" checklist. You don't need to understand the engine -- just match the symptom to the fix.

---

## "Cortex Code won't start"

**What you see:** You type `cortex` in your terminal and nothing happens, or you get an error.

**Why this happens:** Cortex Code might not be installed yet, or your terminal might not know where to find it.

**How to fix it:**

1. First, check if it's installed. Type this and press Enter:
   ```
   cortex --version
   ```
   If you see a version number (like `1.2.3`), it's installed -- skip to step 3.

2. If you get an error, try:
   ```
   cortex --help
   ```
   Still nothing? Cortex Code probably isn't installed. Head to the [official installation docs](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli) and follow the steps for your operating system.

3. If it *is* installed but still won't start, make sure your terminal is in the right place. Navigate to your project folder:
   ```
   cd /path/to/your/project
   ```
   Then try `cortex` again.

---

## "It says 'command not found'"

**What you see:** Something like `cortex: command not found` or `'cortex' is not recognized`.

**Why this happens:** Your computer has a list of places it looks for programs -- think of it like a phonebook for apps. This list is called your **PATH**. If Cortex Code isn't in that phonebook, your computer doesn't know where to find it.

**How to fix it:**

1. The most common fix is to simply reinstall Cortex Code following the [official installation guide](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli). The installer usually adds itself to your PATH automatically.

2. If you just installed it, try closing your terminal completely and opening a new one. Sometimes the PATH doesn't update until you start a fresh terminal window.

3. If you installed via a package manager (like `npm` or `pip`), make sure that package manager's install location is on your PATH. The installation docs cover this for each method.

---

## "It's running slowly or seems confused"

**What you see:** Cortex Code takes a long time to respond, gives answers that don't seem related to what you asked, or seems to "forget" things you told it earlier.

**Why this happens:** Think of a conversation like a notepad. The longer the conversation goes, the more pages you've filled up. Eventually, it gets hard to flip back through all those pages to find the important stuff. Cortex Code works the same way -- very long conversations can slow it down and make it less focused.

**How to fix it:**

1. **Summarize the conversation** to free up space. Type:
   ```
   /compact
   ```
   This is like writing a summary of your notes so far and starting a fresh page. Cortex Code keeps the key points but clears out the clutter.

2. **Start a fresh session** if things are really tangled:
   ```
   /new
   ```
   This gives you a completely clean slate -- like opening a brand new notepad.

3. If it's slow on a specific task, try breaking your request into smaller, simpler pieces instead of one big ask.

---

## "It can't find my files"

**What you see:** Cortex Code says it can't find a file you know exists, or it seems to be looking in the wrong place.

**Why this happens:** Cortex Code can only see files in the directory (folder) where you started it. If your files are somewhere else, it's like looking for a book in the wrong room of a library.

**How to fix it:**

1. **Check where you are right now.** Type:
   ```
   !pwd
   ```
   This shows your current directory (your "location" in the file system).

2. **See what files are visible from here:**
   - On Mac or Linux: `!ls`
   - On Windows: `!dir`

3. **If your files are in a different folder**, you can add that folder so Cortex Code can see it:
   ```
   /add-dir /path/to/your/folder
   ```

4. You can also start Cortex Code from the right directory in the first place by navigating there first:
   ```
   cd /path/to/your/project
   cortex
   ```

---

## "A skill isn't working"

**What you see:** You try to run a skill and nothing happens, or you get an error saying the skill wasn't found.

**Why this happens:** Skills have a specific syntax, and small typos can trip things up.

**How to fix it:**

1. **Make sure you're using the dollar sign**, not a slash. Skills use `$`, like this:
   ```
   $skill-name
   ```
   Not `/skill-name` -- that's for commands, which are a different thing.

2. **Check that the skill exists.** Type:
   ```
   /skills
   ```
   This shows you all available skills. Look for the one you want in the list.

3. **Double-check for typos.** Skill names are case-sensitive and must be exact. `$Commit` is not the same as `$commit`.

4. **If it's a project-specific skill**, make sure you started Cortex Code from within that project's directory. Project skills are only visible when you're in the right project.

---

## "It won't make changes to my files"

**What you see:** You ask Cortex Code to edit a file, but it only describes what it *would* do without actually doing it.

**Why this happens:** You're probably in **Plan mode**. Plan mode is like a "preview" mode -- Cortex Code shows you what it plans to do, but waits for your approval before touching anything. This is a safety feature, not a bug.

**How to fix it:**

1. **Check your current mode:**
   ```
   /status
   ```
   Look for the mode indicator. If it says "Plan," that's your answer.

2. **Switch out of Plan mode** so it can make changes directly:
   ```
   /plan-off
   ```

3. **If it's not a mode issue**, the file itself might be read-only on your computer. Cortex Code can only edit files that your user account has permission to change. Check the file's permissions in your file manager or terminal.

---

## "I can't connect to Snowflake"

**What you see:** Errors about Snowflake connections failing, or Cortex Code says it can't reach your Snowflake account.

**Why this happens:** Your connection details might be missing, incorrect, or expired. It's like trying to log into a website with the wrong password.

**How to fix it:**

1. **Check your current connections:**
   ```
   /connections
   ```
   This shows what connections Cortex Code knows about.

2. **See all available connections** with more detail:
   ```
   cortex connections list
   ```

3. **Make sure your config file exists.** Cortex Code looks for connection details in a file at:
   ```
   ~/.snowflake/connections.toml
   ```
   If that file doesn't exist, you'll need to create one. Check the [Snowflake documentation](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli) for the format.

4. **If your credentials have expired**, you may need to log in again through your Snowflake account. Tokens and passwords can expire over time -- just refresh them and try again.

---

## "The AI went down a wrong path"

**What you see:** Cortex Code started doing something you didn't ask for, or it misunderstood your request and is heading in the wrong direction.

**Why this happens:** AI doesn't always interpret requests perfectly. It's like giving driving directions -- sometimes a small ambiguity leads to a wrong turn.

**How to fix it:**

1. **Undo recent messages** to go back to a good point in the conversation:
   ```
   /rewind
   ```
   This rolls back recent exchanges so you can try again.

2. **Try a different approach in a safe copy.** Use `/fork` first to create a branch of your conversation, then experiment without affecting the original:
   ```
   /fork
   ```
   If the new approach doesn't work, you still have the original to go back to.

3. **Just tell it directly.** You can always say something like:
   > "That's not what I wanted. Let's try a different approach."

   Cortex Code will course-correct based on your feedback.

---

## "I accidentally let it change something I didn't want changed"

**What you see:** Cortex Code edited a file and you wish it hadn't, or the changes broke something.

**Why this happens:** In faster modes (like Bypass), Cortex Code makes changes without asking first. Sometimes it makes a change that isn't quite right.

**How to fix it:**

1. **If you're using git** (a version tracking tool), you can see exactly what changed:
   ```
   git diff
   ```
   And undo changes to a specific file:
   ```
   git checkout -- filename
   ```

2. **Roll back the conversation** to before the change happened:
   ```
   /rewind
   ```

3. **For next time**, consider using **Plan mode** so you can review changes before they happen:
   ```
   /plan
   ```
   This puts a "safety net" in place -- Cortex Code will show you what it wants to do and wait for your OK.

---

## "Permission denied errors"

**What you see:** Error messages containing "permission denied" or "access denied" when Cortex Code tries to read, write, or execute something.

**Why this happens:** Cortex Code runs with the same permissions as your terminal user -- think of it as borrowing your keycard to get into rooms. If *you* don't have access to a file or folder, neither does Cortex Code.

**How to fix it:**

1. **Check if you can access the file yourself.** Try opening it in a text editor or listing it in the terminal. If you can't, the issue is with your user permissions, not Cortex Code.

2. **Work in a directory you own.** Your home folder and project directories are usually fine. System folders and other users' files may be restricted.

3. **On Mac/Linux**, you can check a file's permissions with:
   ```
   !ls -la filename
   ```
   If you need to change permissions, talk to your system administrator or look up `chmod` (but be careful -- changing permissions incorrectly can cause other issues).

---

## "It keeps asking for permission"

**What you see:** Every time Cortex Code wants to do something -- read a file, run a command, edit code -- it stops and asks "Is this OK?"

**Why this happens:** That's the **Confirm Actions** mode, and it's on by default. It's a safety feature, like a car that asks "Are you sure?" before every turn. Great for beginners, but it can slow you down once you're comfortable.

**How to fix it:**

1. **Switch to Bypass mode** to let it work without asking every time:
   ```
   /bypass
   ```

2. **Toggle quickly** using the keyboard shortcut: press `Shift+Tab` to flip between Confirm and Bypass modes.

3. **When prompted for permission**, look for the option to approve "for the rest of this session." This lets the current action type proceed without asking again until you start a new session.

4. **Switch back anytime** if you want the safety net again:
   ```
   /confirm
   ```

---

## General Tips

When something goes wrong, here's your go-to checklist:

- **Run `/doctor` first.** This is your one-stop diagnostic tool. It checks your environment and tells you if anything looks off -- like a health checkup for your setup.

- **Check `/status`** to see your current mode, which AI model you're using, and your active Snowflake connection. A lot of issues come from being in the wrong mode or connected to the wrong account.

- **When in doubt, start fresh** with `/new`. A clean session fixes more problems than you'd expect.

- **Check the FAQ** for quick answers to the most common questions.

- **Report bugs** using `/feedback`. The Cortex Code team reads these and uses them to improve the tool.

---

## Getting Help

If this guide didn't solve your problem, here's where to go next:

| What you need | What to do |
|---|---|
| Quick help | Type `?` |
| Help menu | Type `/help` |
| Full reference guide | Type `$cortex-code-guide` |
| Official documentation | Visit [Cortex Code CLI docs](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli) |

Remember: there's no such thing as a silly question. Everyone starts somewhere, and the tools are here to help you, not the other way around.
