# DATA PROCESSING BRIEF — KAM SUPPLY INTELLIGENCE AGENT

---

**Prepared for:** Data Protection Officer
**Project:** KAM Supply Intelligence Agent — Car Rental Distribution Industry
**Date:** May 2026
**Status:** MVP / Lab prototype (mock data; production migration planned)

---

### What is this system?

The KAM Supply Intelligence Agent is an internal conversational tool that allows Key Account Managers (KAMs) to type plain-language questions in a Slack channel and receive structured answers combining client relationship data and operational product data. It is not a customer-facing system. All users are internal employees.

---

### 1. What personal data does the system process?

The system processes two categories of personal data:

**Internal employee data (KAMs and Account Directors — internal users of the agent):**
- Slack display name and user ID — captured as part of every message event routed through n8n
- The plain-text question typed by the KAM, which contain client names, supplier names and potentially deal-specific context

**Client-side account data (held in Salesforce — about business contacts at client organisations):**
- Full name of the assigned Key Account Manager (an employee field on the Salesforce Account object)
- Account owner name (the internal owner/KAM assigned to each account in Salesforce)

The Supabase operational database contains no directly personal data in its current mock state — it stores supplier names, product codes, route data, and client-supplier linkages, none of which are natural persons. In a production migration, this assessment would need to be revisited if contact-level data is added.

**No special-category data is intentionally collected.** However, product or route data connected to specific individuals in a production environment could theoretically carry inferred attributes; this is not a risk in the current mock-data implementation.

---

### 2. Where does the data come from?

| Data | Source |
|---|---|
| KAM question text and Slack identity | Submitted directly by the internal user in the #kam-agent Slack channel |
| Client account profile (name, tier, business model, contract status, KAM name) | Pulled live from Salesforce Developer Edition via SOQL query on each request |
| Operational data (suppliers, products, routes) | Queried live from Supabase (cloud PostgreSQL) on each request |
| Schema documentation for SQL generation | Stored locally in ChromaDB (vector store on the developer's machine) — no personal data |

---

### 3. What is the data used for?

| Purpose | Description |
|---|---|
| **P1 — Answer generation** | The KAM's question and extracted client name are used to fetch the relevant Salesforce record and generate and execute a SQL query against Supabase, then format a combined answer |
| **P2 — SQL generation via LLM** | The question text, extracted client name, and retrieved schema context are sent to OpenAI's API (GPT-4o-mini) to generate a PostgreSQL SELECT statement |
| **P3 — Answer formatting and delivery** | The combined Salesforce and Supabase results are assembled into a structured text response and posted back to the Slack channel; optionally exported as an XLSX file |
| **P4 — Schema retrieval (RAG)** | The question text is embedded via OpenAI's embeddings API and compared against ChromaDB schema documents to identify relevant table and column definitions before SQL generation — no personal data is stored in ChromaDB |

---

### 4. Who processes the data?

| Entity | Role | What they receive |
|---|---|---|
| The agent system itself (Flask server, LangGraph, running locally or on a private host) | Internal processor | All data flows through this layer |
| **OpenAI API (USA)** | **External processor — third country** | Receives: the KAM's question text, the extracted client name, schema context, and Salesforce result data as part of the prompt sent to GPT-4o-mini; also receives the question text for embedding via the embeddings API |
| **Salesforce (USA)** | **External processor / controller — third country** | Holds and returns client account records including the assigned KAM's name; queried live on each request |
| **Supabase (USA — free tier)** | **External processor — third country** | Holds and returns operational data; SQL is executed against this database on each request |
| **n8n (self-hosted or cloud)** | Orchestration layer | Routes Slack messages to the Flask server and delivers answers back to Slack; receives the full question text and the full formatted answer |
| **Slack (USA)** | Communication platform | Hosts the channel through which questions are submitted and answers are delivered; retains message history per Slack's own data retention settings |

---

### 5. Where is data stored and processed?

| Data store | Location | Notes |
|---|---|---|
| Salesforce Developer Edition | USA (Salesforce infrastructure) | Client account records including KAM names |
| Supabase (free tier) | USA (AWS us-east-1 by default) | Operational product and supplier data |
| OpenAI API | USA | Question text and Salesforce result data sent in API requests; OpenAI's data retention policy for API inputs applies |
| ChromaDB | Local machine (developer's laptop) | Schema documentation only — no personal data |
| Slack | USA (Slack/Salesforce infrastructure) | Message history retained per workspace settings |
| XLSX exports | Local machine (`./exports/` folder) | Generated on demand; accumulate locally and are not automatically deleted |
| n8n | Self-hosted (location depends on deployment) | Passes data in transit only; no persistent storage of personal data by default |

**Key finding for DPO:** All three primary external processors (Salesforce, Supabase, OpenAI) are US-based entities. In a production deployment, this would require a Chapter V transfer mechanism for each — DPF adequacy reliance, SCCs, or both — plus a Transfer Impact Assessment for the OpenAI API relationship given the nature of the data sent in prompts.

---

### 6. Does the system make or assist in decisions that affect people?

The system does not make autonomous decisions about people. It is a query and retrieval tool: it answers factual questions about client accounts and operational product data, and delivers those answers to a human KAM who then acts on them.

The closest analogue to a consequential output is the **account tier and contract status retrieved from Salesforce** — which the agent surfaces as part of every answer and which may influence how the KAM prioritises or prepares for a client interaction. This is an information-retrieval function, not automated scoring or ranking, and a human always reviews the output before any action is taken.

**Art. 22 GDPR assessment:** No automated decision-making with legal or similarly significant effects. The system is a decision-support tool for internal staff. No individual's access to a service, price, or contractual status is altered by the agent's outputs.

---

*This brief reflects the system as of Sprint 3, May 2026. It covers the mock-data MVP implementation. A revised brief will be required before any production migration connecting live customer or employee data.*