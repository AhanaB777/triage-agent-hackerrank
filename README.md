# Multi-Domain Support Triage Agent

> HackerRank Orchestrate — May 2026

A terminal-based AI support triage agent that classifies and responds to customer support tickets across **HackerRank**, **Claude (Anthropic)**, and **Visa** using a semantic RAG pipeline. No external API required — runs fully locally.

---

## Demo

```
╔══════════════════════════════════════════════════════════╗
║       Multi-Domain Support Triage Agent  v1.0            ║
║       Architecture: Semantic RAG + Rule-based Escalation ║
║       Model: all-MiniLM-L6-v2 (local, no API required)  ║
╚══════════════════════════════════════════════════════════╝

Loaded 29 tickets from support_tickets/support_tickets.csv

─────────────────────────────────────────────────────────
[1/29]  Claude  │  Claude access lost
       I lost access to my Claude team workspace...
       Status:  ⚠ escalated
       Area:    account_access  │  Type: product_issue
       Reply:   Claude support cannot restore workspace seats on behalf of non-admin users...
```

---

## Architecture

```
Ticket Input (issue + subject)
        │
        ▼
┌─────────────────────┐
│  Injection Check    │ ──── TRUE ──→ Reply: out of scope  [EXIT]
│  (regex, 3 langs)   │
└─────────────────────┘
        │ FALSE
        ▼
┌─────────────────────┐
│  Semantic Retrieval │
│  all-MiniLM-L6-v2   │
│  cosine similarity  │
│  vs 35 corpus chunks│
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Similarity < 0.30? │ ──── TRUE ──→ Escalate: low confidence  [EXIT]
└─────────────────────┘
        │ FALSE
        ▼
┌─────────────────────┐
│  Hard Keyword Rules │ ──── TRUE ──→ force_escalate = True (continues)
│  refund, fraud...   │
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Corpus Escalation  │
│  Flag Check         │
│  (OR logic)         │
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Structured Output  │
│  → output.csv       │
└─────────────────────┘
```

**Key design choice:** This is a retrieve-and-return architecture rather than retrieve-and-generate. Responses come directly from pre-written grounded corpus chunks — there is no LLM generation step, making hallucination architecturally impossible.

---

## Features

- **Semantic RAG pipeline** — sentence-transformers for contextual matching, not keyword rules
- **No external API** — runs fully locally after first model download, zero cost, no rate limits
- **Dual-layer escalation** — hard keyword rules + corpus escalation flags work together
- **Multilingual injection detection** — regex patterns for English, French, and Spanish
- **Deterministic output** — numpy seed fixed, same input always produces same output
- **Graceful degradation** — similarity below 0.30 escalates instead of guessing
- **Colored terminal UI** — per-ticket status, area, type, and response preview

---

## Project Structure

```
hackerrank-orchestrate-may26/
├── AGENTS.md
├── README.md                        ← you are here
├── .env.example
├── code/
│   ├── main.py                      ← agent entry point
│   └── README.md                    ← setup instructions
└── support_tickets/
    ├── support_tickets.csv          ← input (29 tickets)
    ├── sample_support_tickets.csv   ← labeled examples
    └── output.csv                   ← agent predictions
```

---

## Setup

**Requirements:**
- Python 3.8+
- No API key needed

**Install dependencies:**
```bash
pip install sentence-transformers numpy
```

**Run the agent:**
```bash
cd code
python main.py
```

Output is saved to `support_tickets/output.csv`.

On first run, the sentence-transformers model (~90MB) will download automatically from Hugging Face. Subsequent runs use the cached model.

---

## Output Schema

Each row in `output.csv` contains five fields:

| Field | Description | Values |
|---|---|---|
| `status` | Triage decision | `replied` or `escalated` |
| `product_area` | Support category | `screen`, `billing`, `privacy`, `fraud_security`, etc. |
| `request_type` | Issue classification | `product_issue`, `feature_request`, `bug`, `invalid` |
| `response` | User-facing answer grounded in corpus | Full text |
| `justification` | Internal reasoning with similarity score | Full text |

---

## Results

```
Total tickets : 29
Replied       : 18  (62%)
Escalated     : 11  (38%)
Errors        : 0
```

**Escalation triggers used:**
- Billing, refunds, subscription changes
- Fraud, identity theft, unauthorized transactions
- Platform-wide outages
- Security vulnerability reports
- Account access requiring admin verification
- Ambiguous tickets below similarity threshold
- Prompt injection attempts

---

## Supported Domains

| Company | Coverage |
|---|---|
| HackerRank | Tests, assessments, interviews, billing, user management, community, infosec |
| Claude | Account access, privacy, outages, security, web crawling, API/developer, education |
| Visa | Disputes, lost/stolen cards, identity theft, emergency cash, travel, merchant policy |

---

## Design Decisions

**Why RAG over a pure LLM call?**
RAG forces the agent to use only what is in the corpus. A plain LLM call risks hallucinating policies, phone numbers, or steps that do not exist in the actual support documentation.

**Why sentence-transformers / all-MiniLM-L6-v2?**
Lightweight (90MB), fast on CPU, no API dependency. Designed specifically for semantic similarity tasks. Sufficient for matching support tickets to known documentation patterns.

**Why a 0.30 similarity threshold?**
Conservative by design — in a support context, a wrong answer is worse than escalating. Below 0.30 the match is too weak to trust.

**Why retrieve-and-return instead of retrieve-and-generate?**
Determinism and grounding. Every response is exactly what the corpus says, traceable to the source documentation, with zero risk of the model paraphrasing a policy incorrectly.

---

## Known Limitations

- Corpus is English-only — legitimate non-English tickets may score low similarity and escalate unnecessarily
- Static similarity threshold — optimal value may vary for different ticket distributions
- No cross-ticket memory — each ticket is processed independently
- Novel ticket types with no close corpus match fall back to escalation (safe default, but increases human workload)

---

## Chat Transcript

Built during the HackerRank Orchestrate 24-hour hackathon (May 1–2, 2026).
Full AI collaboration log: `%USERPROFILE%\hackerrank_orchestrate\log.txt`
