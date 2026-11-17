# Sample run

Four questions covering the four behaviours the assistant needs: reuse an approved answer, combine policy and evidence for a written reply, admit a gap, and escalate a restricted request. Outputs are copied from n8n executions (Claude Sonnet 5, November 2026).

| # | Question | Tools the agent chose | Confidence | Routed to |
|---|---|---|---|---|
| 1 | Quick one from a call: do you encrypt customer data at rest? | `find_approved_answer("encrypt")` | High | Not needed (verbal, high confidence) |
| 2 | For an RFP: when was your last penetration test, and can we send them the full report? | `get_compliance_status`, `find_approved_answer("pen test")` → no match, `search_policies("penetration")`, `submit_for_review` | High facts, restricted ask | Head of Security (CISO) |
| 3 | A prospect asks: are you PCI DSS compliant? | `find_approved_answer("PCI")`, `search_policies("PCI")`, `search_policies("payment")`, `find_approved_answer("payment")` → nothing, `submit_for_review` | Low | Head of Security (CISO) |
| 4 | Legal at a prospect wants us to commit to breach notification within 24 hours in the contract. Can we say yes? | `find_approved_answer("breach notification")` → no match, `search_policies("notif")` → IR-4, `submit_for_review` | Medium, restricted | Head of Security (CISO) |

## Replies

**1. Encryption at rest**
> **Answer:** Yes. Customer data at rest is encrypted using AES-256 via AWS Key Management Service, with keys managed by Acme and rotated annually. Data in transit is encrypted with TLS 1.2 or higher (TLS 1.3 preferred). [Approved answer, Head of Security, 2026-09-10 / ISP-6]
> **Confidence:** High – directly confirmed by an approved answer and policy citation.
> **Review:** Not needed (quick verbal question, high confidence).

**2. Pen test for an RFP**
> **Answer:** Acme's most recent independent third-party penetration test of the application and infrastructure was completed on **12 June 2026**, with all findings remediated within our standard ISP-7 targets (critical: 7 days, high: 30 days, medium: 90 days) [ISP-7, Compliance status]. The full report is **not** shared externally; an executive summary of the latest test is available to customers under NDA [ISP-7].
> **Confidence:** High – directly confirmed by policy and current compliance status.
> **Review:** Logged for Head of Security (CISO) — required because this goes to a customer in writing (RFP) and touches the restricted topic of sharing the full pen test report.

**3. PCI DSS (deliberate gap in the policy pack)**
> **Answer:** Not covered by Acme policy – escalated to the Head of Security. I could not find any approved answer or policy documentation confirming PCI DSS compliance status, so I can't confirm this to the prospect at this time.
> **Confidence:** low – no evidence found in approved answers or policy documents.
> **Review:** Logged for Head of Security (CISO).

**4. 24-hour breach notification in the contract (restricted)**
> **Answer:** Acme's standard policy commits to notifying affected customers of a confirmed personal data breach without undue delay, and in any case within 48 hours of confirmation [IR-4] — not 24 hours. We cannot commit to a 24-hour contractual SLA as this falls outside our documented policy and is a contractual term that must be reviewed by the Head of Security/Legal before agreeing with the prospect.
> **Confidence:** medium – policy states 48 hours, not 24, and this is a contractual commitment (restricted topic) requiring sign-off.
> **Review:** Logged for Head of Security (CISO).

## Review queue after the run

| question | confidence | sources | reviewer | status |
|---|---|---|---|---|
| Are you PCI DSS compliant? | low | – | Head of Security (CISO) | Pending review |
| When was your last penetration test, and can we send them the full report? | high | ISP-7 | Head of Security (CISO) | Pending review |
| Legal at a prospect wants us to commit to breach notification within 24 hours… | medium | IR-4 | Head of Security (CISO) | Pending review |

## Observations
- The agent picked different tool combinations per question; nothing in the workflow fixes the order.
- It never invented a PCI answer: two keyword searches in each source came back empty, so it escalated.
- Multi-word keywords (`"breach notification"`) miss the approved-answer table because search is a simple substring match. The agent recovered by searching policies with a stem. Semantic search is the obvious upgrade (see README).
