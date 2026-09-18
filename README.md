CargoLinq Freight Prompt Library (Portfolio)

This is the version history for my BUS4005-T5-W Assessment 1 prompt library. It shows how each prompt was tested and improved, using a real LLM — not simulated.

## What's in this portfolio

- **This file** — the full v1 → v2 test history for all 10 prompts
- **/screenshots** — the real screenshot of every test referenced below (name each file clearly, e.g. `prompt1-v1.png`, `prompt1-v2.png`, `prompt4-v2.png`, so a marker can match a screenshot to its result in seconds)

## How to read this log

Each prompt below shows: the rough first attempt (**v1**), what went wrong, the improved version (**v2**, the one submitted in the Report), and why it changed. Two entries (Prompts 3 and 9) also note a real limitation found during testing — these are kept honest, not smoothed over, because naming real limitations is part of what the assessment rewards.

---

## Requirement I — Lane & Quote Recommendation

### Prompt 1 — Lane & price lookup (tested together with Prompt 2's capacity rule)

**v1 (tested):** "What's a good freight option for moving cargo to Perth?" *Result:* generic road/rail/sea advice, invented competitor names, no CargoLinq data, no structured output, offered to research further instead of answering. *Why changed:* no role, no data constraint, no output format — unusable for automation.

**v2 (tested, submitted):** role-framed prompt with the 4-lane table and fixed output fields (Lane / Transit / Price / Capacity flag). *Result:* correctly matched Melbourne–Perth, correct price ($1.60/kg), correctly computed the capacity shortfall (45‒40 = 5 tonnes). *Why changed:* role framing (the Persona pattern; White et al., 2023), a hard data constraint, and a fixed output format fixed all three v1 failures at once.

### Prompt 3 — Fallback recommendation

**v1 (tested):** "Can you help move something to Perth by tomorrow?" *Result:* no attempt at an answer — the model asked clarifying questions about package type instead of recognising a deadline conflict. *Why changed:* without the lane data, the model can't even detect the problem it's meant to solve.

**v2 (tested, submitted):** same 4-lane table plus an explicit fallback instruction. *Result:* correctly refused the deadline, gave the real 72-hour Perth transit time, named the trade-off, and proactively offered faster lanes as alternatives. *Why changed:* giving the model the actual constraint, and telling it what to do when nothing fits, turned a dead end into a useful answer. *Limitation found:* the reply ran longer than the "one sentence" instruction — a real length-compliance gap, noted in the Report rather than hidden.

## Requirement II — Delay-Risk Triage

### Prompts 4 & 5 — Delay-risk triage + capacity escalation (tested together)

**v1 (tested):** "Is this shipment going to be on time? 45 tonnes, Melbourne to Perth, needed by Friday." *Result:* refused to answer, asked three clarifying questions back, gave no decision at all. *Why changed:* a triage step that bounces every enquiry back to a human hasn't automated anything.

**v2 (tested, submitted):** lane table plus a forced JSON schema and an explicit escalation rule. *Result:* `{"status": "escalate", "reason": "Requested 45 tonnes exceeds the Melbourne-Perth weekly capacity of 40 tonnes."}` — clean JSON, no extra prose, correctly fired on the capacity rule alone. *Why changed:* a machine-readable schema and a deterministic rule turned a conversation into a decision.

### Prompt 6 — Repeat-delay flag

**v1 (tested):** "Has this customer had delivery problems before?" (3 shipments given) *Result:* correct arithmetic, but hedged — offered discussion points instead of a decision. *Why changed:* automation needs a flag, not a discussion.

**v2 (tested, submitted):** forced JSON with an explicit true/false rule. *Result:* `{"repeat_delay": true, "late_count": 2}` — correct and unambiguous. *Why changed:* naming the exact rule (2 of 3 late = true) removed the hedge entirely.

### Prompt 7 — Consolidated triage record

No naive v1 tested — this step is pure merge logic, not a judgement call, so testing went straight to the working version. **v2 (tested, submitted):** merges quote, delay_risk and repeat_delay into one record with a `next_action` field. *Result:* correctly returned `capacity_flag: "yes"`, `repeat_delay: true`, and — since both were true — `next_action: "escalate_to_ops"`.

## Requirement III — Customer-Response Drafting

### Prompt 8 — On-track status email

**v1 (tested):** "Write an email to a customer telling them their freight shipment is on track." *Result:* generic template full of bracket placeholders ([Carrier Name], [ETA]) — invented fields never given, no length control. *Why changed:* a real automation can't hand a customer a fill-in-the-blanks form.

**v2 (tested, submitted):** grounded in an actual triage record, 110–130 word limit specified. *Result:* 130 words exactly, entirely grounded in the record, no invented details. *Why changed:* grounding the prompt in the actual JSON record eliminated the placeholders entirely.

### Prompt 9 — At-risk status email

**v2 (tested, submitted):** grounded in an at-risk triage record, instructed not to confirm the deadline. *Result:* named the real conflict (no scheduled departure on the requested day), offered the mitigation (earlier pickup), and explicitly declined to confirm a delivery date — but ran to 145 words against a 130-word ceiling. *Limitation found:* content constraints were followed correctly; the length constraint was not — output should be checked before sending, not assumed compliant.

### Prompt 10 — Escalation status email

**v2 (tested, submitted):** grounded in an escalate triage record, instructed to give no delivery promise. *Result:* 117 words, correctly stated the capacity conflict, correctly named the pending ops-manager review, and gave no delivery date or price anywhere in the text. *Why tested this way:* this is the highest-stakes prompt in the library, so it was tested directly against its one hard requirement (no promise) rather than iterated from a naive version.

---

## Cross-cutting finding

Across the ten tested prompts, the most consistent v1 failure mode was the model inventing details that were never in the input, rather than acknowledging missing information. Prompt 1's v1 invented named freight carriers (Toll, Linfox, TNT) that CargoLinq does not use, and Prompt 8's v1 generated placeholder fields (carrier name, ETA) implying data that was never supplied. Both were corrected in v2 by grounding the prompt strictly in CargoLinq's own lane table or triage record and explicitly forbidding invented detail — the same fix, applied twice.

## Responsible AI considerations

This library was designed against five risk categories, drawing on NIST's Generative AI Risk Management Profile (NIST, 2024):

- **Hallucination / confabulation** — the primary risk found through testing (see Cross-cutting finding above). Mitigated through explicit "use only case-record facts" instructions in every response-drafting prompt, verified by manual testing on two of the ten prompts (1 and 8).
- **Bias** — the prompts that make an automated decision (4, 5 and 6 — delay-risk triage, capacity escalation and the repeat-delay flag) rely only on objective, quantifiable criteria — requested weight against stated capacity, and a count of late shipments — rather than any subjective or customer-specific judgement, limiting the surface area for a model's training biases to affect a real outcome. This follows the general caution in Bender et al. (2021) that large language models can reproduce biases present in their training data when used for consequential judgements.
- **Privacy** — prompts handling customer data (6, 8, 9 and 10 — the repeat-delay flag and the three status emails) are scoped to use only shipment and triage data already held in CargoLinq's own systems, and never request or output customer data beyond what each task needs.
- **Security** — no prompt stores, logs, or transmits data outside the immediate task; every drafted email is reviewed before sending, limiting the impact of any single incorrect or manipulated output.
- **Governance** — automated confirmation is permitted only on the on_track path, where the facts are already verified against CargoLinq's own data; any at_risk or escalate outcome (Prompts 9 and 10) is deliberately kept low-automation and human-reviewed before a customer sees it — concentrating oversight at the workflow's highest-risk points, as NIST (2024) recommends.

## References

Bender, E. M., Gebru, T., McMillan-Major, A., & Shmitchell, S. (2021). On the dangers of stochastic parrots: Can language models be too big? *Proceedings of the 2021 ACM Conference on Fairness, Accountability, and Transparency*, 610–623.

National Institute of Standards and Technology. (2024). *Artificial intelligence risk management framework: Generative artificial intelligence profile* (NIST-AI-600-1). https://doi.org/10.6028/NIST.AI.600-1

---


