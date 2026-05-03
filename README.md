# Multi-Domain Support Triage Agent
### HackerRank Orchestrate Competition

A terminal-based AI support triage agent for **HackerRank**, **Claude**, and **Visa**.

---

## Quick Start

```bash
# 1. Install dependencies
pip install requests

# 2. Set your Anthropic API key
export ANTHROPIC_API_KEY=sk-ant-...

# 3. Run the agent
python3 agent.py
```

Output is written to `support_tickets/output.csv`.

---

## Architecture

```
support_tickets.csv  ─→  agent.py  ─→  Claude API (claude-sonnet-4)  ─→  output.csv
                            │
                     [System Prompt]
                            │
                     ┌──────────────────┐
                     │  Support Corpus  │
                     │  • HackerRank    │
                     │  • Claude        │
                     │  • Visa          │
                     └──────────────────┘
                            │
                     ┌──────────────────┐
                     │  Few-Shot        │
                     │  Examples (5)    │
                     └──────────────────┘
```

## Key Design Decisions

### 1. Grounded Corpus (No Hallucination)
The agent uses an inline support corpus scraped from official support sites. Claude is instructed to use **only this corpus** and never fabricate policies, phone numbers, or URLs.

### 2. Smart Escalation Logic
The agent escalates when:
- Billing/refund/subscription issues (require human access to billing systems)
- Platform-wide outages (require engineering)
- Fraud, identity theft (require immediate human intervention)
- Security vulnerabilities (must go through proper disclosure channels)
- Account access issues requiring identity verification
- Ambiguous tickets with high-risk mis-routing potential

### 3. Prompt Injection Detection
The agent identifies and safely handles prompt injection attempts embedded in tickets (e.g., Ticket #25 in French asking for internal system rules).

### 4. Multi-language Support
The agent processes tickets in any language (demonstrated by French ticket #25).

### 5. Safety Over Completeness
When company is "None" and context is ambiguous, the agent escalates rather than risk wrong routing.

## Output Schema

| Field | Values |
|-------|--------|
| `status` | `replied` \| `escalated` |
| `product_area` | screen, billing, privacy, etc. |
| `request_type` | `product_issue` \| `feature_request` \| `bug` \| `invalid` |
| `response` | User-facing message grounded in corpus |
| `justification` | Internal triage reasoning |

## Files

```
triage_agent/hackerrank-orchestratte-may26/
├── code/
    ├── main.py                       # Main terminal agent                                                  # This file
└── support_tickets/
    ├── support_tickets.csv           # Input (29 tickets)
    ├── sample_support_tickets.csv    # Reference examples
    └── output.csv                    # Generated output
```
