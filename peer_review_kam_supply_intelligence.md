# Peer Review — KAM Supply Intelligence Agent
## Independent GDPR Audit Based on Data Processing Brief

**Auditor:** Olalla Murciego
**System audited:** KAM Supply Intelligence Agent — Car Rental Distribution Industry
**Builder:** Dilia [surname]
**Date:** May 2026
**Brief reviewed:** Data Processing Brief, May 2026 (mock-data MVP)

---

## Phase 1: Annotations

**Personal data categories identified:**
- Slack display name and user ID (internal employee data)
- Plain-text KAM questions, which may contain client names, supplier names, and deal context (internal employee data, potentially including third-party business contact names)
- KAM full name as assigned account owner in Salesforce (internal employee data)
- Client account profile data pulled from Salesforce: account name, tier, business model, contract status, assigned KAM name

**Special-category data:** Not identified and not inferable from the brief in the current mock-data implementation. The brief correctly notes that production migration could change this if contact-level data is added.

**Data flows crossing an EU border:** All three primary external processors — OpenAI, Salesforce, Supabase — are US-based. Every session involving a KAM question routes data to at least one US processor. Slack is also US-based. No transfer mechanism is documented in the brief for any of these relationships.

**Potentially incompatible purpose reuse:** Not identified. The brief describes a single coherent use case (internal query and retrieval) with no apparent secondary use of data.

**Items requiring clarification:**
- Slack's role is listed in section 5 but absent from section 4's role map
- XLSX export retention is flagged but not assessed for risk
- No lawful basis is proposed for any processing purpose
- No DPA status is confirmed for any processor
- No mention of data subject rights operationalisation

---

## Phase 2: Personal Data and Role Map

### Personal Data Summary

| Data Category | Source | Purpose(s) | Crosses EU Border? | Special Category? |
|---|---|---|---|---|
| Slack display name and user ID | KAM submitting question via Slack | System operation; routing messages to Flask server and back | YES — Slack (USA) retains message history | No |
| KAM question text (may contain client names, deal context) | KAM submitting question via Slack | Answer generation (P1); SQL generation via LLM (P2); schema retrieval via embeddings (P4) | YES — sent to OpenAI (USA) for inference and embeddings | No |
| KAM full name (Salesforce account owner field) | Salesforce Developer Edition (queried live) | Included in Salesforce result data assembled into answers (P3) | YES — held by Salesforce (USA); also sent to OpenAI as part of prompt context | No |
| Client account profile (name, tier, business model, contract status) | Salesforce Developer Edition (queried live) | Answer generation and delivery to KAM (P1, P3) | YES — held by Salesforce (USA); also sent to OpenAI as part of prompt | No |

### Role Map

| Entity | Role | Processing Activity | DPA Needed? |
|---|---|---|---|
| Builder / organisation deploying the agent | Controller | Determines purposes and means of all processing; configures all integrations | N/A |
| OpenAI API (USA) | Processor | LLM inference on KAM question text, client names, and Salesforce result data; embeddings generation for schema retrieval | YES — not confirmed in brief |
| Salesforce (USA) | Processor / Controller (own platform data) | Holds and returns client account records including KAM names; queried live on each request | YES — not confirmed in brief |
| Supabase (USA) | Processor | Holds and returns operational data; SQL executed against this database on each request | YES — not confirmed in brief |
| Slack (USA) | Processor | Routes KAM messages to n8n; delivers answers back to channel; retains message history per workspace settings | YES — not confirmed in brief; absent from brief's role map |
| n8n | Processor | Orchestrates workflow; routes data between Slack, Flask server, and delivery layer | YES if using n8n Cloud — deployment not confirmed in brief |
| Flask server / LangGraph (local or private host) | Internal processor | Core agent logic; all data flows through this layer | N/A if internal |

**International transfer finding:** Four US-based processors handle personal data on every session (OpenAI, Salesforce, Supabase, Slack). No Chapter V transfer mechanism is documented in the brief for any of them. This is the most structurally significant gap in the system.

---

## Phase 3: Clarifying Questions Log

**Question 1: What is the lawful basis for processing KAM employee data via an external LLM?**

Why it matters: Art. 6 GDPR requires a lawful basis for every processing purpose. Sending employee Slack messages and names to a US-based LLM API requires a basis that covers both the processing itself and the international transfer. Employment contract (Art. 6(1)(b)) is the most likely candidate for internal tool use, but it needs to be confirmed and documented.

Provisional assumption: Legitimate interests (Art. 6(1)(f)) or contract is likely intended. However, without confirmation, no lawful basis can be assumed to be in place.

**Question 2: Have Data Processing Agreements been executed with OpenAI, Salesforce, Supabase, and Slack?**

Why it matters: Art. 28 GDPR requires a DPA for every controller/processor relationship. All four named processors are US-based and handle personal data on every session. If no DPAs are in place, processing is unlawful regardless of whether the system is in testing mode, because real employee data (Slack user IDs, KAM names) is already flowing.

Provisional assumption: No DPAs have been executed for this specific project deployment. OpenAI, Salesforce, and Slack all provide standard DPAs for API customers — the question is whether they have been formally accepted for this use case.

**Question 3: What are the Slack workspace retention settings, and who controls them?**

Why it matters: Slack retains message history by default. Every KAM question — including embedded client names and deal context — is retained by Slack under its own policies unless the workspace is configured otherwise. The controller may have limited ability to enforce deletion timelines for data held by Slack, which affects the ability to respond to erasure requests under Art. 17.

Provisional assumption: Default Slack retention settings apply, meaning message history is retained indefinitely or per Slack's standard policy. No custom retention configuration has been documented.

**Question 4: What happens to the XLSX exports stored locally?**

Why it matters: The brief notes that XLSX files accumulate in a local `./exports/` folder and are not automatically deleted. These files contain Salesforce account data and KAM names with no documented retention schedule, no access controls, and no deletion mechanism. This creates a data minimisation and retention risk under Art. 5(1)(c) and (e).

Provisional assumption: XLSX exports are currently unmanaged from a data protection perspective. No retention schedule exists and no deletion process has been documented.

**Question 5: Will the system connect to live Salesforce data before a DPIA is conducted?**

Why it matters: The brief states that a production migration is planned. In production, the system would process live client contact data at scale, combining it with employee data and routing it to multiple US processors. This likely triggers the DPIA requirement under Art. 35 — specifically the criteria of combining datasets, processing employee data via innovative technology, and large-scale international transfer. A DPIA must be completed before production go-live, not after.

Provisional assumption: No DPIA has been conducted for the current implementation or planned for the production migration.

---

## Phase 4: Audit Report

### Section 1: System Summary

The KAM Supply Intelligence Agent is an internal Slack-based conversational tool that allows Key Account Managers to ask plain-language questions and receive structured answers combining live Salesforce CRM data and operational supply data from a Supabase database. The system uses LangGraph for agent orchestration, a Flask server for core logic, and GPT-4o-mini for both SQL generation and answer formatting. It is not customer-facing — all users are internal employees. The current implementation uses mock data in Supabase, but the Salesforce integration pulls live data including KAM names and client account profiles. A production migration connecting live client contact data is planned.

### Section 2: Data and Role Map

The system processes two categories of personal data: internal employee data (Slack identities and KAM question text, KAM names as Salesforce account owners) and client-side account data (account profiles pulled live from Salesforce, including tier, contract status, and assigned KAM). The controller is the organisation deploying the agent. Four external processors handle personal data on every session: OpenAI, Salesforce, Supabase, and Slack — all US-based. Every session constitutes an EEA-to-US international transfer across all four relationships. No transfer mechanism is documented for any of them in the brief. No DPA status is confirmed for any processor.

### Section 3: Compliance Findings

---

**Finding 1 — International Transfer Mechanism**
Severity: Blocking

All four primary external processors (OpenAI, Salesforce, Supabase, Slack) are US-based. Personal data — including employee Slack messages, KAM names, and Salesforce account data — crosses the EEA border on every session. No Chapter V transfer mechanism (adequacy decision, SCCs, or otherwise) is documented in the brief for any of these relationships. The EU-US Data Privacy Framework covers some Salesforce and Slack entities but requires verification that the specific entities used are DPF-certified. OpenAI and Supabase require SCCs or another valid mechanism.

Recommended action: Verify DPF certification status for Salesforce and Slack entities in use. Execute SCCs with OpenAI and Supabase before any personal data flows. Document all transfer mechanisms in the RoPA.

Escalation needed: Yes — legal counsel, to assess DPF reliance and execute SCCs.

---

**Finding 2 — Data Processing Agreements**
Severity: Blocking

Art. 28 GDPR requires a DPA for every controller/processor relationship. The brief identifies four external processors (OpenAI, Salesforce, Supabase, Slack) and does not confirm whether any DPA has been executed for this project deployment. Real employee data is already flowing through at least OpenAI and Slack in the current implementation. Processing without a DPA in place is unlawful regardless of whether the system is in testing mode.

Recommended action: Execute DPAs with OpenAI, Salesforce, Supabase, and Slack before the next session involving personal data. All four vendors provide standard DPAs for API customers — this is a documentation task, not a negotiation.

Escalation needed: Yes — legal counsel or DPO to execute and file agreements.

---

**Finding 3 — Lawful Basis**
Severity: Blocking

The brief does not propose a lawful basis for any processing purpose. For internal employee data processed via a third-party LLM, the most defensible bases are contract (Art. 6(1)(b) — processing necessary for the performance of the employment relationship) or legitimate interests (Art. 6(1)(f)). For client account data held in Salesforce, the lawful basis would depend on the nature of the relationship between the deploying organisation and its clients. Neither has been assessed or documented.

Recommended action: Document a lawful basis for each processing purpose before any personal data is processed. If relying on legitimate interests for any purpose, complete and document a Legitimate Interests Assessment.

Escalation needed: Yes — legal counsel or DPO to confirm basis selection.

---

**Finding 4 — DPIA Requirement for Production Migration**
Severity: Blocking

The brief confirms that a production migration is planned. In production, the system will process live client contact data combined with employee data, routing both to multiple US processors via an LLM. This likely triggers at least three of the EDPB's nine DPIA criteria: combining multiple datasets, processing via innovative technology (LLM-based agent), and large-scale international transfer. A DPIA must be completed before production go-live, not after.

Recommended action: Conduct a DPIA before any production migration. If the DPIA identifies high residual risk, consult the supervisory authority before proceeding.

Escalation needed: Yes — DPO (mandatory involvement under Art. 35).

---

**Finding 5 — Slack Absent from Role Map**
Severity: Significant

Slack is listed in section 5 of the brief but absent from section 4's role map. This is not a formatting issue — it means Slack's processing activity, DPA requirement, and data retention implications have not been fully assessed. Slack retains message history by default, meaning every KAM question containing client names and deal context is retained by a US platform under Slack's own policies, potentially indefinitely.

Recommended action: Add Slack to the role map. Review and configure workspace retention settings. Confirm DPA status. Assess whether Slack message retention can be limited to the minimum necessary period.

Escalation needed: No — this can be addressed by the builder with IT/admin support.

---

**Finding 6 — XLSX Export Retention**
Severity: Significant

XLSX exports accumulate in a local `./exports/` folder with no automated deletion, no documented retention schedule, and no access controls described in the brief. These files contain Salesforce account data and KAM names. This is a data minimisation and storage limitation failure under Art. 5(1)(c) and (e).

Recommended action: Implement an automated deletion schedule for XLSX exports (e.g. 30-day rolling deletion). Document the retention period. Restrict access to the exports folder to authorised users only.

Escalation needed: No — this can be addressed by the builder.

---

**Finding 7 — Data Subject Rights Operationalisation**
Severity: Significant

The brief does not address how the system would respond to a data subject access request or erasure request. KAM question text is retained by Slack; Salesforce data is queried live; OpenAI may retain API inputs per its own policy. A DSAR from a KAM or a client contact would require coordinated responses across at least four platforms, none of which have documented response procedures.

Recommended action: Document a DSAR response procedure covering all four processors. Confirm OpenAI's data retention policy for API inputs and whether deletion can be requested.

Escalation needed: No — can be addressed by the builder with DPO guidance.

---

### Section 4: GDPR Obligations Checklist

| Obligation | Assessment | Note |
|---|---|---|
| Lawful basis identified for each processing purpose | Gap identified | No lawful basis proposed or documented in brief |
| Purpose limitation respected | Appears met | Single coherent use case; no secondary use identified |
| Data minimisation | Gap identified | XLSX exports accumulate with no deletion; Slack retains all message history |
| Controller/processor roles mapped and DPAs in place | Gap identified | Slack absent from role map; no DPA confirmed for any processor |
| International transfer mechanism documented | Gap identified | Four US processors; no transfer mechanism documented for any |
| DPIA conducted if required | Cannot determine from brief | Not mentioned; likely required for production migration |
| Art. 22 safeguard in place if automated decisions affect people | Appears met | System is decision-support only; human always reviews before action |
| Privacy notice covers AI processing | Cannot determine from brief | Not mentioned in brief |
| Data subject rights can be operationalised within deadlines | Gap identified | No DSAR procedure documented; multi-platform coordination required |

---

### Section 5: Overall Recommendation

**Proceed with conditions.**

The system's core design is sound: it is an internal tool, it does not make automated decisions about individuals, and the data it processes is limited in scope and sensitivity. The Art. 22 assessment is well-reasoned and the mock-data approach to Supabase correctly limits current risk. However, three blocking findings must be resolved before the system processes personal data in any session: Data Processing Agreements must be executed with all four external processors, a lawful basis must be documented for each processing purpose, and international transfer mechanisms must be confirmed and documented. The planned production migration must not proceed without a completed DPIA. Two additional significant findings — Slack retention and XLSX export management — should be resolved promptly but do not block current testing provided the above conditions are met.

---

### Section 6: What This Report Is Not

This report is a first-pass independent audit based solely on the data processing brief provided. It is not a legal opinion, not a formal Data Protection Impact Assessment, and not a certification of compliance with GDPR or any other regulation. The builder and their organisation should obtain qualified legal advice and engage a Data Protection Officer before relying on this assessment for any compliance decision. Findings and recommendations reflect the system as described in the brief and may not account for implementation details not captured in that document.

---

## Phase 5: Debrief

*To be completed in person with Dilia after both audits are finalised.*

**Sequence:**
1. Auditor (Olalla) presents this report without interruption
2. Builder (Dilia) responds with any clarifications or context not captured in the brief
3. Compare lawful basis selections for the primary processing purpose
4. Compare DPIA conclusions — do both audits agree a DPIA is required for production?
5. Compare gap lists — what did the self-audit catch that this review missed, and vice versa?

**Joint closing note:** *(to be written together with Dilia after the debrief and added here before submission)*

---

## Reinforce

*Review the clarifying questions Dilia logged about the PMM Market Intelligence Agent. Which of those questions cannot be answered from the current project documentation? What does that reveal about documentation gaps in that system?*

*(To be completed after receiving Dilia's peer review report.)*

---

## Stretch: Remediation Plan — Finding 1 (International Transfer Mechanism)

**Finding:** Four US-based processors (OpenAI, Salesforce, Supabase, Slack) handle personal data on every session. No Chapter V transfer mechanism is documented for any of them.

**What specific artifact closes this gap?**
- For OpenAI and Supabase: executed Standard Contractual Clauses (SCCs), using the European Commission's 2021 standard form, supplemented by a Transfer Impact Assessment (TIA) documenting the risk assessment for each transfer
- For Salesforce and Slack: verification of EU-US Data Privacy Framework certification for the specific legal entities in use, plus executed DPAs incorporating SCCs as a fallback mechanism
- All four transfer mechanisms documented in the Records of Processing Activities (RoPA)

**Who would own it?**
The controller (the organisation deploying the agent), with execution handled by legal counsel or the DPO. The builder can prepare the groundwork by identifying the specific legal entities for each vendor and pulling their standard DPA and SCC documentation.

**Realistic timeline:**
- Week 1: Identify legal entities for all four vendors; pull standard DPA and SCC documents from each vendor's legal portal
- Week 2: Legal counsel reviews and executes SCCs with OpenAI and Supabase; confirms DPF certification for Salesforce and Slack
- Week 3: TIAs drafted for OpenAI and Supabase transfers; RoPA updated with all four transfer mechanisms
- Total: 3 weeks with active legal counsel involvement

**What evidence would confirm to a regulator that the gap is closed?**
- Executed DPA documents with all four vendors, dated before the first session involving personal data
- Signed SCCs with OpenAI and Supabase using the 2021 standard form
- Written confirmation of DPF certification status for Salesforce and Slack (or executed SCCs as fallback)
- TIAs on file for the OpenAI and Supabase relationships
- RoPA entry for each processing activity referencing the applicable transfer mechanism


**Written together with Dilia after the debrief, May 2026.**
### Joint Closing Note
### Where both audits agree

| Finding | Why both audits reached the same conclusion |
|---|---|
| No DPAs with any external processor | Four US-based vendors handle personal data on every session; the absence of Art. 28 agreements is structurally unavoidable regardless of perspective |
| No Chapter V transfer mechanism | All processors are US-based; EEA-to-US transfers are inherent to the architecture |
| DPIA mandatory and not initiated | System meets at least three EDPB criteria; both audits applied the same threshold |
| No internal privacy notice for employees | Both audits identified this as a prerequisite for legitimate interests to hold, not a downstream obligation |
| Overall verdict: proceed with conditions | Internal tool, no autonomous decisions affecting individuals, bounded data scope — both audits weighted these mitigating factors consistently |

### Where they diverge

| Finding | Self-audit | External review | Why they differ |
|---|---|---|---|
| Slack in the role map | Listed in accountability table, not escalated | Standalone Significant finding | External auditor had no prior familiarity with Slack as a workflow component and evaluated it on equal terms with other processors |
| XLSX export retention | Noted as a documentation gap | Standalone Significant finding requiring immediate action | Same reason — builder treated it as a known limitation rather than a compliance risk |
| Betriebsrat / §87 BetrVG | Raised as a residual risk | Not identified | Domain knowledge the builder has that is invisible to an auditor working only from the brief |
| Remediation specificity | Concrete technical fixes: Bearer token auth already in codebase, OpenAI ZDR mode, secrets manager migration | Gap identified, no technical detail | Builder knows the stack; external auditor does not |
| Partial LIA analysis | Drafted, with specific elements flagged for legal review | Identified absence of lawful basis as Blocking, no LIA attempt | Builder had enough context to start the analysis; external auditor could only flag the absence |

### What this comparison reveals

The self-audit was stronger at remediation; the external review was stronger at gap prioritisation and role mapping. The Slack omission is the clearest illustration of the core dynamic: components that feel like standard infrastructure become invisible as compliance relationships to the builder. An external auditor has no such familiarity and therefore applies the same scrutiny to every processor regardless of how embedded it feels in the workflow.