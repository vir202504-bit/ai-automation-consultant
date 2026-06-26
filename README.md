# 🤖 AI Automation Consultant — Lead-to-Report Pipeline

**Submit a company website → get back a fully branded PDF report identifying automation opportunities, researched tools, pricing, and an implementation blueprint. Zero human involvement.**

Built with [n8n](https://n8n.io/), [Manus AI](https://manus.im/), [Tavily](https://tavily.com/), OpenAI, and Tally Forms.

[![n8n](https://img.shields.io/badge/built%20with-n8n-EA4B71?logo=n8n&logoColor=white)](https://n8n.io/)
[![Status](https://img.shields.io/badge/status-live-2EA44F)]()
[![Type](https://img.shields.io/badge/type-lead%20magnet%20%2F%20AI%20research%20pipeline-blue)]()

> 📄 This repo includes the **actual n8n workflow exports** (sanitized — all API keys, tokens, and personal data removed). Import them straight into your own n8n instance, drop in your own credentials, and it runs.

---

## 📌 The Problem

Automation consultants and agencies get asked the same question constantly: *"What could I automate in my business?"*

Answering that well requires:
- Researching the prospect's business model, industry, and maturity
- Spotting genuine automation opportunities (not generic ones)
- Researching real tools, pricing, and integrations for each opportunity
- Writing it all up as something a prospect can actually read and act on

Doing this by hand for every inbound lead doesn't scale — it either gets skipped, or it eats hours per lead.

## 💡 The Solution

A prospect fills out a short form (website + industry). From there, the system runs unattended:

1. **Capture** — A Tally form submission triggers an n8n webhook with the company's website and industry
2. **Business Research** — Manus AI analyzes the website and returns a structured profile: business model, maturity score, automation-readiness score, and the top automation opportunities — each with impact, difficulty, and ROI
3. **Tool Research (sub-workflow)** — For each opportunity, a second workflow uses **Tavily's research API** to find real tools, pricing, n8n integration options, alternatives, and case studies
4. **Blueprint Generation** — An OpenAI-powered AI Agent turns the research into an implementation-ready automation blueprint, written as a senior automation architect would
5. **Report Delivery** — The blueprint is converted from Markdown → styled HTML → a branded PDF report, ready to send to the prospect

No human touches a single step between form submission and finished report.

---

## 🏗️ Architecture

```mermaid
flowchart TD
    A["📝 Tally Form Submission<br/>(Website + Industry)"] --> B["🪝 Webhook Trigger"]
    B --> C["🧹 Edit Fields — extract Website / Industry"]
    C --> D["🔬 Manus AI — analyze business<br/>(model, maturity, automation-readiness,<br/>top opportunities)"]
    D --> E{"Polling Loop<br/>Manus task status"}
    E -- running --> F["⏳ Wait"] --> E
    E -- completed --> G["📦 Aggregate automation opportunities"]
    G --> H["📞 Call: Research Agent Sub-workflow"]

    subgraph SUB ["Research Agent Sub-workflow (Tavily)"]
        H1["Create research task"] --> H2["⏳ Wait 1 min"] --> H3["Get research status"] --> H4["Extract content"]
    end

    H --> H1
    H4 --> I["🧠 AI Agent (OpenAI)<br/>builds implementation blueprint"]
    I --> J["📝 Markdown → HTML"]
    J --> K["📄 Convert HTML to PDF"]
    K --> L["✅ Branded PDF Report"]
```

The system is split into **two workflows**:

| Workflow | Responsibility |
|---|---|
| **AI Automation Consultant** (main) | Receives the form submission, runs the Manus business analysis, polls until complete, hands each automation opportunity to the research sub-workflow, then builds and renders the final PDF report |
| **Research Agent for Tool (Tavily)** | Reusable sub-workflow — given any topic, kicks off a Tavily research task, polls for completion, and returns the synthesized research content |

The Tavily sub-workflow is intentionally generic — it's called once per automation opportunity, but it's reusable for *any* research task, not just this pipeline.

---

## 🔄 Step-by-Step Flow

| Step | What Happens |
|---|---|
| 1. Form Submission | Prospect submits their website + industry via a Tally form |
| 2. Webhook Trigger | n8n receives the submission instantly |
| 3. Field Extraction | Website, industry, and submission date are pulled out of the payload |
| 4. Manus Business Analysis | Manus AI visits the website and returns a structured JSON profile: company name, industry, business model, maturity score, automation-readiness score, and a ranked list of automation opportunities |
| 5. Status Polling | The workflow waits and polls until the Manus task is complete (handles long-running research gracefully) |
| 6. Aggregate Opportunities | All identified automation opportunities are collected into a single list |
| 7. Per-Opportunity Research | The Tavily sub-workflow is called to research real tools, pricing, APIs, n8n integrations, alternatives, and case studies for each opportunity |
| 8. Blueprint Generation | An AI Agent (acting as a "Senior AI Automation Architect") synthesizes everything into an implementation-ready blueprint |
| 9. Markdown → HTML | The blueprint is converted to styled HTML using a custom CSS theme |
| 10. PDF Generation | The styled HTML is rendered into a polished, brand-ready PDF report |

---

## 🧰 Tech Stack

| Tool | Role in the System |
|---|---|
| **n8n** | Orchestration engine — webhook handling, polling logic, sub-workflow calls |
| **Tally Forms** | Front-end lead capture form |
| **Manus AI** | Analyzes the prospect's website and surfaces automation opportunities with structured output |
| **Tavily** | Researches real tools, pricing, and case studies for each opportunity |
| **OpenAI (via n8n LangChain nodes)** | Powers the AI Agent that writes the final implementation blueprint |
| **HTML/CSS → PDF** | Converts the finished blueprint into a branded, presentation-ready PDF |

---

## 📁 Repo Structure

```
.
├── README.md                          → you are here
├── docs/
│   └── WORKFLOW_OVERVIEW.md           → detailed breakdown of every node and decision point
└── workflows/
    ├── AI_Automation_Consultant.json   → main n8n workflow (import directly into n8n)
    └── Research_Agent_Tavily.json      → reusable Tavily research sub-workflow
```

---

## ⚙️ Setting It Up Yourself

1. Import both files in `workflows/` into your n8n instance (`Workflows → Import from File`)
2. Create credentials for:
   - **HTTP Header Auth** — for Manus AI (`Authorization: Bearer YOUR_MANUS_API_KEY`)
   - **Tavily API** — your Tavily account key
   - **OpenAI API** — for the blueprint-writing AI Agent
   - **HTML/CSS to PDF API** — for final report rendering
3. Point the `Webhook` node's path at your Tally (or any) form
4. In the **Main Workflow**, set the *Call sub-workflow* node to point at your imported Tavily research workflow
5. Activate both workflows

> All API keys, account tokens, and the original test submission data have been removed from these exports — you'll need to wire in your own credentials.

---

## 🎯 Who This Is Built For

- Automation agencies and freelance consultants who want an instant, high-quality lead magnet
- Anyone fielding "what should I automate?" questions and wanting a consistent, well-researched answer every time
- Teams that want a repeatable way to scope automation opportunities for a new client *before* the first sales call

---

## 📬 About / Contact

Built by **VIRSpace AI** — automation systems for sales, lead-generation, and operations.

- 🌐 [virspaceai.com](https://virspaceai.com/)
- 📧 viraj.bhapkar@virspaceai.com
- 📱 +91 99871 42405

> Workflow exports in this repo have had all credentials, API keys, and real lead/test data removed. They're shared as working reference implementations, not a hosted/managed product.
