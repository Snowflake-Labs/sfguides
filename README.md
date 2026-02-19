# Snowflake Tutorial Skills for Cortex Code

Interactive, agent-guided tutorials that teach Snowflake features through hands-on learning. Built using the "Agent-as-the-Teacher" pattern for [Cortex Code](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-code).

## Available Skills

| Skill | Description | Invoke With |
|-------|-------------|-------------|
| `dynamic-tables-tutorial` | Build data pipelines with automatic refresh, incremental processing, and CDC patterns | "teach me dynamic tables" |
| `cortex-classify-tutorial` | Classify unstructured text using Cortex AI (Python & SQL) | "teach me Cortex CLASSIFY_TEXT" |
| `cortex-classify-notebook` | Deploy the CLASSIFY_TEXT tutorial notebook to your Snowflake account | "deploy the Cortex classify notebook" |

## Installation

### Install All Skills

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/snowflake-tutorial-skills.git

# Add all skills to Cortex Code
cortex skill add ./snowflake-tutorial-skills/dynamic-tables-tutorial
cortex skill add ./snowflake-tutorial-skills/cortex-classify-tutorial
cortex skill add ./snowflake-tutorial-skills/cortex-classify-notebook
```

### Install Individual Skills

```bash
# Just Dynamic Tables
cortex skill add ./snowflake-tutorial-skills/dynamic-tables-tutorial

# Just Cortex Classify (agent-guided)
cortex skill add ./snowflake-tutorial-skills/cortex-classify-tutorial

# Just Cortex Classify (notebook deployment)
cortex skill add ./snowflake-tutorial-skills/cortex-classify-notebook
```

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

## Skill Details

### Dynamic Tables Tutorial

**What you'll learn:**
- How Dynamic Tables automatically maintain fresh data with TARGET_LAG
- How incremental refresh processes only changed rows
- The difference between Dynamic Tables and Materialized Views
- How Dynamic Tables simplify Change Data Capture (CDC)
- Monitoring and troubleshooting refresh operations

**Lessons:**
1. Data Loading - Load Tasty Bytes menu data from S3
2. Creating Dynamic Tables - Build with TARGET_LAG
3. Incremental Refresh - Generate new data, verify incremental behavior
4. Materialized View Migration - Compare and convert MVs
5. CDC Comparison - Streams+Tasks vs Dynamic Tables
6. Cleanup - Verify and clean up

### Cortex Classify Tutorial

**What you'll learn:**
- How to classify unstructured text into custom categories
- Using Cortex CLASSIFY_TEXT in Python (single string and DataFrame)
- Using Cortex CLASSIFY_TEXT in SQL
- Writing effective task descriptions for better results

**Lessons:**
1. Setup & Data Loading - Load truck reviews from S3
2. Classify Single String - Use Python `cortex.classify_text` on one review
3. Classify DataFrame Column - Classify entire column with task description
4. Classify in SQL - Use `SNOWFLAKE.CORTEX.CLASSIFY_TEXT` in SQL

### Cortex Classify Notebook

Deploys a ready-to-run Jupyter notebook to your Snowflake account. The notebook itself guides you through the same content as the agent-guided tutorial, but in Snowsight's notebook interface.

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
- [Dynamic Tables Documentation](https://docs.snowflake.com/en/user-guide/dynamic-tables-about)
- [Cortex CLASSIFY_TEXT Documentation](https://docs.snowflake.com/en/sql-reference/functions/classify_text-snowflake-cortex)
- [SNOWFLAKE_LEARNING Environment](https://docs.snowflake.com/en/release-notes/bcr-bundles/un-bundled/bcr-1992)
