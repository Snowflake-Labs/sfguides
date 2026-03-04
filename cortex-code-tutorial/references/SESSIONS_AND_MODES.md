# Sessions and Modes Reference

## What is a Session?

- A session is one conversation with Cortex Code.
- Think of it like a document -- it has a history of everything you and the AI said.
- Sessions are automatically saved so you can come back to them later.
- Each session has a unique ID and can optionally have a name.

## Session Commands

### Starting Fresh

- `cortex` -- starts a new session.
- `/new` -- starts a new session from inside Cortex Code.
- `/clear` -- clears the screen but keeps the same session.

### Naming Your Session

- `/rename my-project` -- gives the session a descriptive name.
- Like saving a document with a filename instead of "Untitled."
- Named sessions are easier to find later.

### Resuming a Past Session

- `cortex --resume last` or `cortex -r last` -- picks up your most recent session.
- `cortex resume` -- shows a list of up to 100 past sessions to choose from.
- `/resume` -- same list, from inside Cortex Code.
- Everything is restored: the conversation, context, and state.

### Forking (Branching Off)

- `/fork` -- creates a copy of your conversation at a point you choose.
- `/fork my-experiment` -- same, with a name.
- Like "Save As" -- you get a new branch without losing the original.
- Great for: trying a different approach without risking your current work.
- Opens an interactive picker to choose which message to branch from.

### Rewinding (Undo)

- `/rewind` -- opens an interactive picker to choose how far back to go.
- `/rewind 3` -- goes back 3 user messages.
- Like pressing Ctrl+Z (undo) on your conversation.
- Rewound messages are discarded permanently.
- Great for: when the AI went down a wrong path.

### Compacting (Summarizing)

- `/compact` -- summarizes the conversation and clears the history.
- `/compact focus on the API changes` -- summarizes with specific focus.
- Think of it like writing meeting minutes -- you keep the key points, discard the chatter.
- Use when: the conversation is getting very long and the AI seems to slow down or lose track.
- The summary stays in context so nothing important is truly lost.

## Modes

Cortex Code has three modes that control how much freedom the AI has:

### Confirm Actions Mode (Default)

- The AI asks your permission before making changes.
- Like a coworker who says "I'm going to edit this file -- is that OK?"
- **Safest mode** -- you approve everything before it happens.
- You'll see prompts like "Allow this?" with options to approve or deny.

### Plan Mode

- The AI can look at things and make plans, but CANNOT make any changes.
- Like asking an architect to draw blueprints but not start building.
- Enable: `/plan` or press `Ctrl+P`.
- Disable: `/plan-off` or press `Ctrl+P` again.
- Great for: reviewing code, exploring ideas, planning big changes safely.

### Bypass Mode

- The AI does everything without asking permission.
- Like telling your coworker "I trust you, just get it done."
- Enable: `/bypass`.
- Disable: `/bypass-off`.
- **Use with caution** -- the AI will make changes without checking with you.
- Great for: repetitive tasks where you trust the AI's judgment.

### Switching Modes Quickly

- `Shift+Tab` -- cycles between Confirm Actions and Bypass mode.
- `/status` -- shows which mode you're currently in.

### When to Use Each Mode

| Situation | Recommended Mode |
|-----------|-----------------|
| Normal work | Confirm Actions |
| Exploring code or planning | Plan Mode |
| Repetitive formatting or simple tasks | Bypass Mode |
| Working on important/sensitive files | Confirm Actions |
| Learning Cortex Code (like right now!) | Confirm Actions |

## Keyboard Shortcuts for Sessions and Modes

| Shortcut | What it does |
|----------|-------------|
| `Ctrl+P` | Toggle plan mode |
| `Shift+Tab` | Cycle between Confirm and Bypass mode |
| `Ctrl+O` | Cycle display mode (compact / expanded / transcript) |
| `Ctrl+R` | Search through your prompt history |
| `Up Arrow` | Previous command |

## Tips

- Name your important sessions with `/rename` -- you'll thank yourself later.
- Use `/fork` before trying something risky.
- Use `/rewind` as your "undo button" when things go wrong.
- Use `/compact` when the conversation feels sluggish.
- Start in Confirm Actions mode until you're comfortable, then try Bypass for simple tasks.
