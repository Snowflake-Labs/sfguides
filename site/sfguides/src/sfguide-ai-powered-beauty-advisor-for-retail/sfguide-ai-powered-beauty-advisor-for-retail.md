author: Amit Gupta, Joviane Bellegarde
id: sfguide-ai-powered-beauty-advisor-for-retail
summary: Build an AI-powered beauty advisor with Snowflake Cortex Agent that delivers a complete shopping experience from product discovery through checkout.
categories: snowflake-site:taxonomy/solution-center/certification/quickstart, snowflake-site:taxonomy/industry/retail-and-cpg, snowflake-site:taxonomy/product/ai, snowflake-site:taxonomy/product/applications-and-collaboration, snowflake-site:taxonomy/snowflake-feature/cortex-analyst, snowflake-site:taxonomy/snowflake-feature/cortex-search, snowflake-site:taxonomy/snowflake-feature/snowpark-container-services
tags: Cortex Agent, Cortex Analyst, Cortex Search, Hybrid Tables, SPCS, Retail, AI
language: en
environments: web
status: Hidden
feedback link: https://github.com/Snowflake-Labs/sfguides/issues
fork repo link: https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail

# AI-Powered Beauty Advisor for Retail with Snowflake Cortex Agent
<!-- ------------------------ -->
## Overview

In this guide, you will build an AI-powered beauty advisor that helps shoppers discover products, analyze their skin tone, match colors, manage a shopping cart, and check out — all through a conversational chatbot powered by Snowflake Cortex Agent.

The beauty advisor connects to a unified Product 360 data model across five domains — products, customers, inventory, social proof, and transactional cart — giving the agent full context to answer any question a shopper might ask.

![Architecture Overview](assets/architecture-overview.png)

| Snowflake Feature | How It's Used |
|-------------------|---------------|
| **Cortex Agent** | 16-tool orchestrator connecting all data domains |
| **Cortex Analyst** | 5 semantic views for natural-language-to-SQL queries |
| **Cortex Search** | Product catalog and social content discovery |
| **Hybrid Tables** | ACID-compliant shopping cart at 10-50ms latency |
| **SPCS** | Face recognition and skin analysis backend |
| **AI SQL** | Unstructured product label processing |
| **Vector Search** | Face embedding matching for customer recognition |

### Prerequisites
- A Snowflake account (<a href="https://signup.snowflake.com/" target="_blank">Enterprise edition</a> or higher)
- A user with the ACCOUNTADMIN role
- <a href="https://www.docker.com/products/docker-desktop/" target="_blank">Docker Desktop</a> installed and running

### What You'll Learn
- How to create a Cortex Agent with 16 tools spanning five data domains
- How to build semantic views for Cortex Analyst text-to-SQL
- How to set up Cortex Search services for product and social content discovery
- How to use Hybrid Tables for real-time transactional cart operations
- How to deploy a containerized ML backend on Snowpark Container Services
- How to use vector search for face recognition and customer matching

### What You'll Need
- A Snowflake account with ACCOUNTADMIN access
- Docker Desktop for pushing the backend container image
- Approximately 90 minutes

### What You'll Build
- A conversational beauty advisor chatbot on a merchant website
- A Product 360 data agent with analytics, search, and face analysis capabilities
- An AI-powered product label extraction pipeline
- A real-time shopping cart with checkout powered by Hybrid Tables

<!-- ------------------------ -->
## Set Up the Environment

Open a new SQL workspace in <a href="https://app.snowflake.com" target="_blank">Snowsight</a> and run the following script. It creates the role, database, schemas, warehouse, and compute resources you need for the rest of the guide.

Copy and paste the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/01_setup_database.sql" target="_blank">scripts/sql/01_setup_database.sql</a> into the workspace and run all statements.

This creates:

| Object | Name | Purpose |
|--------|------|---------|
| Role | `AGENT_COMMERCE_ROLE` | Owns all objects created in this guide |
| Database | `AGENT_COMMERCE` | Contains all schemas and data |
| Schemas | `CUSTOMERS`, `PRODUCTS`, `INVENTORY`, `SOCIAL`, `CART_OLTP`, `UTIL`, `DEMO_CONFIG` | One schema per data domain |
| Warehouse | `AGENT_COMMERCE_WH` (X-Small) | Runs queries and loads data |
| Compute Pool | `AGENT_COMMERCE_POOL` | Hosts the SPCS backend service |
| Image Repository | `UTIL.AGENT_COMMERCE_REPO` | Stores the Docker container image |

To verify, run:

```sql
SHOW SCHEMAS IN DATABASE AGENT_COMMERCE;
```

You should see 7 schemas (plus `INFORMATION_SCHEMA` and `PUBLIC`).

> NOTE: The script dynamically grants `AGENT_COMMERCE_ROLE` to your current user. If you need to grant access to other users, uncomment the `GRANT ROLE` statement at the bottom of the script.

<!-- ------------------------ -->
## Connect to the GitHub Repository

Before loading data, connect Snowflake to the GitHub repository that contains all the source files for this guide. This uses Snowflake's <a href="https://docs.snowflake.com/en/developer-guide/git/git-setting-up" target="_blank">Git Integration</a> feature.

Run the following in your Snowsight workspace:

```sql
USE ROLE ACCOUNTADMIN;

CREATE OR REPLACE API INTEGRATION agent_commerce_git_integration
    API_PROVIDER = GIT_HTTPS_API
    API_ALLOWED_PREFIXES = ('https://github.com/Snowflake-Labs/')
    ENABLED = TRUE
    COMMENT = 'Git integration for Agent Commerce guide repository';

GRANT USAGE ON INTEGRATION agent_commerce_git_integration TO ROLE AGENT_COMMERCE_ROLE;

USE ROLE AGENT_COMMERCE_ROLE;
USE DATABASE AGENT_COMMERCE;
USE SCHEMA UTIL;

CREATE OR REPLACE GIT REPOSITORY UTIL.AGENT_COMMERCE_GIT
    API_INTEGRATION = agent_commerce_git_integration
    ORIGIN = 'https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail.git'
    COMMENT = 'Agent Commerce source code and data';

ALTER GIT REPOSITORY UTIL.AGENT_COMMERCE_GIT FETCH;
```

To verify, list the repository contents:

```sql
LIST @UTIL.AGENT_COMMERCE_GIT/branches/main/scripts/sql/;
```

You should see all 13 SQL scripts listed.

<!-- ------------------------ -->
## Create Tables and Load Data

This step creates approximately 25 tables across all schemas and loads them with sample data — beauty products, customer profiles, inventory, social reviews, and shopping cart records.

### Create the Tables

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/02_create_tables.sql" target="_blank">scripts/sql/02_create_tables.sql</a> in your workspace.

This creates regular tables for products, customers, inventory, and social data. It also creates **Hybrid Tables** in the `CART_OLTP` schema — these provide ACID transactions, row-level locking, and foreign key enforcement for the shopping cart, giving the chatbot low-latency cart operations (10-50ms).

> NOTE: Hybrid Tables require Enterprise edition. If you see errors creating Hybrid Tables, confirm your account edition supports them.

### Load Sample Data

Next, run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/03_load_sample_data.sql" target="_blank">scripts/sql/03_load_sample_data.sql</a>.

This copies 24 CSV files and product images from the Git repository into Snowflake, then loads them into all tables. It uses `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` so column order in the CSVs does not matter.

At the end of the script, a verification query shows the row count for every table. Confirm all 23 tables have data loaded:

```sql
SELECT 'PRODUCTS.PRODUCTS' AS table_name, COUNT(*) AS row_count FROM PRODUCTS.PRODUCTS
UNION ALL SELECT 'CUSTOMERS.CUSTOMERS', COUNT(*) FROM CUSTOMERS.CUSTOMERS
UNION ALL SELECT 'INVENTORY.LOCATIONS', COUNT(*) FROM INVENTORY.LOCATIONS
UNION ALL SELECT 'SOCIAL.PRODUCT_REVIEWS', COUNT(*) FROM SOCIAL.PRODUCT_REVIEWS
UNION ALL SELECT 'CART_OLTP.ORDERS', COUNT(*) FROM CART_OLTP.ORDERS
ORDER BY table_name;
```

<!-- ------------------------ -->
## Build Semantic Views for Product Analytics

Semantic views let you ask natural language questions about your data. Cortex Analyst converts those questions into SQL using the metadata you define — table relationships, business metrics, and common synonyms.

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/04_create_semantic_views.sql" target="_blank">scripts/sql/04_create_semantic_views.sql</a> in your workspace.

This creates five semantic views:

| Semantic View | Schema | What It Answers |
|---------------|--------|----------------|
| `CUSTOMER_SEMANTIC_VIEW` | CUSTOMERS | "How many Gold-tier customers do we have?" |
| `PRODUCT_SEMANTIC_VIEW` | PRODUCTS | "What are our top 5 most expensive lipsticks?" |
| `INVENTORY_SEMANTIC_VIEW` | INVENTORY | "Which products are below reorder point?" |
| `SOCIAL_PROOF_SEMANTIC_VIEW` | SOCIAL | "What's the average rating for MAC products?" |
| `CART_SEMANTIC_VIEW` | CART_OLTP | "How many orders were completed this month?" |

Each semantic view defines the tables, relationships, facts (numeric measures), dimensions (grouping attributes), and metrics (calculated business KPIs) that Cortex Analyst uses to generate accurate SQL.

To verify, run:

```sql
SHOW SEMANTIC VIEWS IN DATABASE AGENT_COMMERCE;
```

You should see 5 semantic views.

<!-- ------------------------ -->
## Set Up Search and Vector Capabilities

This step creates the search services and vector search procedures that power the agent's ability to find products, discover social content, and recognize customers by face.

### Create Cortex Search Services

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/05_create_cortex_search.sql" target="_blank">scripts/sql/05_create_cortex_search.sql</a>.

This creates two Cortex Search services:

| Service | What It Searches |
|---------|-----------------|
| `PRODUCTS.PRODUCT_SEARCH_SERVICE` | Product names, descriptions, ingredients, warnings, and labels — unified from multiple tables |
| `SOCIAL.SOCIAL_SEARCH_SERVICE` | Product reviews, social media mentions, and influencer content |

Both services refresh automatically every hour (`TARGET_LAG = '1 hour'`).

> NOTE: Search service creation takes 1-3 minutes while the initial index is built. Wait for the statements to complete before proceeding.

To verify:

```sql
SHOW CORTEX SEARCH SERVICES IN DATABASE AGENT_COMMERCE;
```

### Create Vector Search Procedures

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/06_create_vector_embeddings.sql" target="_blank">scripts/sql/06_create_vector_embeddings.sql</a>.

This creates six stored procedures for face-based customer recognition using 128-dimensional vector embeddings:

- `FIND_CUSTOMER_BY_FACE` — Match an uploaded face against registered customers
- `CHECK_CUSTOMER_EXISTS` — Quick check if a customer has registered face data
- `REGISTER_FACE_EMBEDDING` — Save a new face embedding for a customer
- `FIND_SIMILAR_CUSTOMERS` — Find customers with similar facial features
- `DEACTIVATE_FACE_EMBEDDING` — Soft-delete a face embedding (GDPR-friendly)
- `DEACTIVATE_ALL_CUSTOMER_EMBEDDINGS` — Remove all face data for a customer

These procedures use `VECTOR_COSINE_SIMILARITY` and `VECTOR_L2_DISTANCE` for fast similarity matching.

<!-- ------------------------ -->
## Create Agent Tools

The Cortex Agent needs tools to take actions — analyzing faces, matching product colors, and managing the shopping cart. This step creates those tools as SQL functions and procedures the agent can call.

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/07_create_agent_tools.sql" target="_blank">scripts/sql/07_create_agent_tools.sql</a>.

This creates nine tools:

| Tool | What It Does |
|------|-------------|
| `TOOL_ANALYZE_FACE` | Sends a face image to the ML backend for skin tone analysis |
| `TOOL_IDENTIFY_CUSTOMER` | Matches a face embedding against registered customers |
| `TOOL_MATCH_PRODUCTS` | Finds products matching a skin tone color using CIEDE2000 color science |
| `TOOL_CREATE_CART_SESSION` | Starts a new shopping cart |
| `TOOL_GET_CART_SESSION` | Retrieves current cart contents and totals |
| `TOOL_ADD_TO_CART` | Adds a product to the cart |
| `TOOL_UPDATE_CART_ITEM` | Changes quantity of a cart item |
| `TOOL_REMOVE_FROM_CART` | Removes an item from the cart |
| `TOOL_SUBMIT_ORDER` | Completes checkout — creates an order and processes payment |

The cart tools operate on Hybrid Tables, giving the agent real-time transactional operations.

<!-- ------------------------ -->
## Build the Cortex Agent

Now you bring everything together. The Cortex Agent is the orchestration layer that decides which tools to call based on the shopper's question.

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/08_create_cortex_agent.sql" target="_blank">scripts/sql/08_create_cortex_agent.sql</a>.

This creates `UTIL.AGENTIC_COMMERCE_ASSISTANT` — a Cortex Agent powered by `claude-4-sonnet` with 16 tools:

| Tool Category | Count | Tools |
|--------------|-------|-------|
| Cortex Analyst (text-to-SQL) | 5 | CustomerAnalyst, ProductAnalyst, InventoryAnalyst, SocialAnalyst, CheckoutAnalyst |
| Cortex Search | 2 | ProductSearch, SocialSearch |
| Face & Color (functions) | 3 | AnalyzeFace, IdentifyCustomer, MatchProducts |
| Cart & Checkout (procedures) | 6 | ACP_CreateCart, ACP_GetCart, ACP_AddItem, ACP_UpdateItem, ACP_RemoveItem, ACP_Checkout |

The agent's system prompt instructs it to follow a privacy-first flow for face recognition — always asking permission before analyzing a face photo, and explaining what data will be used.

To verify:

```sql
SHOW AGENTS IN SCHEMA UTIL;
```

You should see `AGENTIC_COMMERCE_ASSISTANT` listed.

<!-- ------------------------ -->
## Deploy the Application on SPCS

The beauty advisor's face recognition and skin analysis backend runs as a containerized service on Snowpark Container Services (SPCS). A pre-built Docker image is available on Docker Hub.

### Push the Docker Image to Snowflake

Open a terminal on your local machine and run:

```bash
docker pull amitgupta392/agent-commerce-backend:latest
```

Next, log in to your Snowflake image registry and push the image. Replace `<your-account>` with your Snowflake account identifier (e.g., `sfsenorthamerica-myaccount`):

```bash
export REGISTRY="<your-account>.registry.snowflakecomputing.com"

docker login $REGISTRY

docker tag amitgupta392/agent-commerce-backend:latest \
  $REGISTRY/agent_commerce/util/agent_commerce_repo/agent-commerce-backend:latest

docker push $REGISTRY/agent_commerce/util/agent_commerce_repo/agent-commerce-backend:latest
```

> NOTE: The image is approximately 2GB. The push may take a few minutes depending on your connection speed.

To find your exact registry URL, run this in Snowsight:

```sql
SELECT CONCAT(
    CURRENT_ACCOUNT_NAME(),
    '.registry.snowflakecomputing.com/',
    'agent_commerce/util/agent_commerce_repo'
) AS docker_registry_url;
```

### Create the SPCS Service

Back in Snowsight, run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/09_deploy_spcs.sql" target="_blank">scripts/sql/09_deploy_spcs.sql</a>.

This creates the `AGENT_COMMERCE_BACKEND` service with a public endpoint for the face analysis API.

Wait for the service to reach READY status:

```sql
SELECT SYSTEM$GET_SERVICE_STATUS('UTIL.AGENT_COMMERCE_BACKEND');
```

> NOTE: The service typically takes 3-5 minutes to start. Run the status check periodically until you see `READY`.

Once ready, get the public endpoint URL:

```sql
SHOW ENDPOINTS IN SERVICE UTIL.AGENT_COMMERCE_BACKEND;
```

Save this URL — you will use it to access the beauty advisor.

### Load Face Images and Demo Customer

Run the full contents of <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/10_load_face_images_and_embeddings.sql" target="_blank">scripts/sql/10_load_face_images_and_embeddings.sql</a>, then run <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/11_add_demo_customer.sql" target="_blank">scripts/sql/11_add_demo_customer.sql</a>.

The first script processes face images into 128-dimensional vector embeddings. The second adds a demo customer ("Sarah Johnson", Gold tier, 2,500 loyalty points) so you can test face recognition immediately.

<!-- ------------------------ -->
## Explore the Beauty Advisor

Open the public endpoint URL from the previous step in your browser. You should see the beauty advisor chatbot interface.

Try these interactions to exercise different capabilities:

### Product Discovery (Cortex Search)

Type: **"What moisturizers do you recommend for sensitive skin?"**

The agent uses `ProductSearch` to find relevant products from the catalog and returns recommendations with images and pricing.

### Analytics Question (Cortex Analyst)

Type: **"What are the top 5 best-selling lipstick brands?"**

The agent uses `ProductAnalyst` to convert your question into SQL and returns data-backed results.

### Face Analysis (SPCS Backend)

Upload a photo using the camera or upload button, then type: **"Analyze my skin tone and recommend matching foundations."**

The agent calls `AnalyzeFace` to detect your skin tone, then uses `MatchProducts` to find foundations that match your complexion using CIEDE2000 color science.

### Shopping Cart (Hybrid Tables)

Type: **"Add the first foundation to my cart."**

The agent calls `ACP_CreateCart` and `ACP_AddItem` to create a real-time cart session. Follow up with:

- **"What's in my cart?"** — calls `ACP_GetCart`
- **"Change the quantity to 2"** — calls `ACP_UpdateItem`
- **"Checkout"** — calls `ACP_Checkout` to place the order

### Social Proof (Cortex Search)

Type: **"What are people saying about NARS products on social media?"**

The agent uses `SocialSearch` to find reviews, influencer mentions, and social media posts about the brand.

<!-- ------------------------ -->
## Conclusion And Resources

Congratulations! You have built an AI-powered beauty advisor that combines six Snowflake capabilities into a single conversational experience:

### What You Learned
- How to create a **Cortex Agent** with 16 tools spanning analytics, search, ML, and transactional operations
- How to define **semantic views** for Cortex Analyst text-to-SQL across five data domains
- How to set up **Cortex Search** services for product and social content discovery
- How to use **Hybrid Tables** for real-time ACID cart and checkout operations
- How to deploy a containerized ML service on **Snowpark Container Services**
- How to use **vector search** with 128-dimensional face embeddings for customer recognition

### Cleanup

When you are done, run <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail/blob/main/scripts/sql/99_teardown.sql" target="_blank">scripts/sql/99_teardown.sql</a> to remove all objects created in this guide:

```sql
-- Switch to AGENT_COMMERCE_ROLE first
USE ROLE AGENT_COMMERCE_ROLE;
USE DATABASE AGENT_COMMERCE;

DROP SERVICE IF EXISTS UTIL.AGENT_COMMERCE_BACKEND;
DROP CORTEX SEARCH SERVICE IF EXISTS PRODUCTS.PRODUCT_SEARCH_SERVICE;
DROP CORTEX SEARCH SERVICE IF EXISTS SOCIAL.SOCIAL_SEARCH_SERVICE;
DROP AGENT IF EXISTS UTIL.AGENTIC_COMMERCE_ASSISTANT;

-- Switch to ACCOUNTADMIN for account-level objects
USE ROLE ACCOUNTADMIN;
DROP COMPUTE POOL IF EXISTS AGENT_COMMERCE_POOL;
DROP DATABASE IF EXISTS AGENT_COMMERCE;
DROP WAREHOUSE IF EXISTS AGENT_COMMERCE_WH;
DROP INTEGRATION IF EXISTS agent_commerce_git_integration;
DROP ROLE IF EXISTS AGENT_COMMERCE_ROLE;
```

### Related Resources
- <a href="https://github.com/Snowflake-Labs/sfguide-ai-powered-beauty-advisor-for-retail" target="_blank">GitHub Repository</a>
- <a href="https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents" target="_blank">Cortex Agent Documentation</a>
- <a href="https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst" target="_blank">Cortex Analyst Documentation</a>
- <a href="https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search" target="_blank">Cortex Search Documentation</a>
- <a href="https://docs.snowflake.com/en/sql-reference/sql/create-hybrid-table" target="_blank">Hybrid Tables Documentation</a>
- <a href="https://docs.snowflake.com/en/developer-guide/snowpark-container-services/overview" target="_blank">Snowpark Container Services Documentation</a>
