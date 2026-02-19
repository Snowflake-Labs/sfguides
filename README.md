# Snowflake Tutorial Skills for Cortex Code

Interactive, agent-guided tutorials that teach Snowflake features through hands-on learning. Built using the "Agent-as-the-Teacher" pattern for [Cortex Code](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-code).

## Quick Install

```bash
# Install all tutorial skills with one command
npx skills add https://github.com/Snowflake-Labs/sfguides
```

Or install individual skills:

```bash
npx skills add https://github.com/Snowflake-Labs/sfguides/tree/main/dynamic-tables-tutorial
npx skills add https://github.com/Snowflake-Labs/sfguides/tree/main/cortex-classify-tutorial
npx skills add https://github.com/Snowflake-Labs/sfguides/tree/main/cortex-classify-notebook
```

> Powered by [skills](https://github.com/vercel-labs/skills) from Vercel Labs

## Available Skills

| Skill | Description |
|-------|-------------|
| `dynamic-tables-tutorial` | Build data pipelines with automatic refresh, incremental processing, and CDC patterns |
| `cortex-classify-tutorial` | Classify unstructured text using Cortex AI (Python & SQL) |
| `cortex-classify-notebook` | Deploy the CLASSIFY_TEXT tutorial notebook to your Snowflake account |

## Usage

Once installed, start a new Cortex Code session and invoke the skill:

```
You: teach me dynamic tables

Agent: [Fetches latest docs, welcomes you, and begins the tutorial]
```

The agent will:
1. **Fetch the latest documentation** from Snowflake
2. **Detect your environment** (prefers SNOWFLAKE_LEARNING if available)
3. **Guide you step-by-step** through the tutorial
4. **Explain before executing** any command
5. **Pause for confirmation** before each step
6. **Answer questions** using deep reference materials
7. **Verify your work** at the end

## Requirements

- [Cortex Code CLI](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-code) installed
- Snowflake account with Cortex AI enabled
- For best experience: SNOWFLAKE_LEARNING environment ([BCR-1992](https://docs.snowflake.com/en/release-notes/bcr-bundles/un-bundled/bcr-1992))

## Skill Structure

Each skill follows the standard Cortex Code skill format:

```
skill-name/
├── SKILL.md           # Main agent instructions (YAML frontmatter + markdown)
└── references/        # Deep reference materials for answering questions
    ├── LESSONS.md     # Step-by-step code for the tutorial
    ├── FAQ.md         # Quick answers
    ├── TROUBLESHOOTING.md
    └── ...            # Topic-specific deep dives
```

## Teaching Philosophy

These skills implement the "Agent-as-the-Teacher" pattern:

1. **Explain-Before-Execute**: Every command is explained before running
2. **Pause for Confirmation**: User confirms each step (even if auto-allowed)
3. **One Step at a Time**: Small, digestible chunks
4. **Deep Reference Materials**: Agent can answer any question thoroughly
5. **Verify Understanding**: Checkpoint questions between lessons
6. **Always Current**: Fetches latest docs at tutorial start

## Contributing

Contributions welcome! Please ensure:
- SKILL.md follows the [Cortex Code skill format](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-code-skills)
- Include comprehensive reference materials
- Follow the explain-before-execute pattern
- Test with a fresh Snowflake account

## License

Apache 2.0 License - see [LICENSE](LICENSE)

## Resources

- [Cortex Code Documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-code)
- [Snowflake Documentation](https://docs.snowflake.com)
- [SNOWFLAKE_LEARNING Environment](https://docs.snowflake.com/en/release-notes/bcr-bundles/un-bundled/bcr-1992)
