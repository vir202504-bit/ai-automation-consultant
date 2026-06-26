# Workflow Overview: AI Automation Consultant

A node-by-node breakdown of both workflows in this repo.

---

## 1. Main Workflow — `AI_Automation_Consultant.json`

| Node | Type | Purpose |
|---|---|---|
| **Webhook** | Webhook Trigger | Receives the Tally form `FORM_RESPONSE` payload (POST `/Lead_capture`) |
| **Edit Fields** | Set | Extracts `Website`, `Industry`, and formats the submission date |
| **Manus Research** | HTTP Request → `api.manus.ai/v2/task.create` | Sends the website + a structured-output schema to Manus AI, asking it to identify business model, maturity, automation-readiness, and ranked automation opportunities |
| **Wait** | Wait | Pauses before checking task status (Manus research runs asynchronously) |
| **HTTP Request** | HTTP Request → `api.manus.ai/v2/task.listMessages` | Polls the Manus task for its latest messages/status |
| **Switch** | Switch | Branches on `agent_status`: `running` → loop back to Wait; `stopped` → continue to results |
| **Aggregate** | Aggregate | Collects `automation_opportunities` from the Manus structured output into a single array |
| **Call 'Research Agent for Tool with tavily'** | Tool Workflow (sub-workflow call) | For each opportunity, calls the Tavily research sub-workflow to gather pricing, tools, integrations, alternatives, and case studies |
| **AI Agent** | LangChain Agent | Acting as a "Senior AI Automation Architect," synthesizes the Manus profile + Tavily research into an implementation-ready blueprint |
| **OpenAI Chat Model** | LangChain LM (OpenAI) | The underlying model powering the AI Agent |
| **Markdown** | Markdown → HTML | Converts the AI Agent's Markdown output into styled HTML |
| **Convert HTML to PDF** | HTML/CSS to PDF | Renders the final HTML (with a custom CSS theme — headings, tables, page breaks) into a polished PDF report |

### Structured Output Schema (Manus AI)

The Manus research call requests a strict JSON schema in return, including:

- `company_name`, `industry`, `business_model`
- `business_maturity_score`, `automation_readiness_score`
- `automation_opportunities[]` — each with `name`, `impact`, `difficulty`, `roi`, `n8n_workflow`
- `recommended_next_step`

This is what makes the downstream steps reliable — every automation opportunity arrives in a predictable shape that the Aggregate and AI Agent nodes can consume directly.

### AI Agent System Prompt (summary)

The AI Agent is instructed to act as a Senior AI Automation Architect with the following capabilities:

- Design complete automation workflows from trigger to final output
- Research software tools, APIs, SaaS platforms, and AI solutions
- Create detailed implementation plans suitable for n8n
- Recommend the most cost-effective and scalable technology stack
- Estimate implementation complexity and ROI
- Compare tools and provide alternatives

---

## 2. Sub-workflow — `Research_Agent_Tavily.json`

A small, reusable research workflow designed to be called from any parent workflow.

| Node | Type | Purpose |
|---|---|---|
| **When Executed by Another Workflow** | Execute Workflow Trigger | Accepts a single `input` string — the research question/topic |
| **Create research task** | Tavily node (`resource: research`) | Kicks off an asynchronous Tavily research task using the `mini` model |
| **Wait** | Wait (1 minute) | Gives Tavily time to complete the research |
| **Get research status** | Tavily node (`operation: status`) | Polls the task using the returned `request_id` |
| **Edit Fields** | Set | Extracts the final `content` field to return to the calling workflow |

**Example input** (used during testing):
> "Provide tool recommendations, pricing details, API availability, n8n integration, alternatives, and case studies for Dynamic Pricing and Route Optimization with AI-Driven Insights in logistics."

Because this sub-workflow only takes a plain-text `input` and returns plain-text `content`, it can be reused for *any* research task, in *any* workflow — not just this pipeline.

---

## Data & Privacy Notes

The original workflow exports contained test data captured during development, including:

- A live Manus AI API key (left in a sticky note)
- A real Tally form submission (a logistics company's website + industry, plus access tokens and request metadata)
- The developer's personal n8n Cloud instance subdomain

All of the above have been **removed or redacted** in the versions published in this repo. If you import these workflows, you'll need to add your own credentials before they will run.
