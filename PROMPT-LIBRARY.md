# Report — Structured Prompt Library: CargoLinq Freight

Business field: road-freight brokerage. CargoLinq Freight's manual quoting and delay communication are causing missed sailing windows. Customer operations is one of the four functions capturing roughly 75% of generative AI's economic value (Chui et al., 2023).

| # | Req | Prompt | Task | Problem solved | Automation potential | Risks / limitations |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | I | Act as a freight quoting assistant for CargoLinq. Match origin, destination, weight and deadline to the approved lane table; return lane, transit time and price. | Lane and price lookup | Manual quoting is slow | High, deterministic for listed lanes | Fails for routes outside the four lanes |
| 2 | I | If weight exceeds the lane's weekly capacity, return capacity\_flag with the shortfall instead of quoting (role framing; White et al., 2023). | Capacity-constrained quoting | Reps over-promised capacity | High, deterministic threshold | Needs daily capacity updates |
| 3 | I | If no lane meets the deadline, suggest the nearest feasible lane and date, and state the trade-off in one line. | Fallback recommendation | "Not possible" with no alternative | Medium, needs judgement | Tested: ran longer than the one-sentence limit |
| 4 | II | Extract weight, lane and deadline from the enquiry; compare to capacity and transit time; return JSON status (on\_track, at\_risk or escalate) and reason. | Delay-risk triage | Risk assessed inconsistently from memory | High, core automation step | Wrong status is high-impact, hard to audit |
| 5 | II | Set status to escalate whenever weight exceeds the lane's weekly capacity, regardless of deadline. | Capacity escalation rule | Overbooking found only after pickup | High, deterministic | Ignores multi-week partial shipments |
| 6 | II | From the customer's last 3 shipments, set repeat\_delay to true if 2 or more were late; add to escalation reasons. | Repeat-delay flag | Chronic delays not prioritised | Medium, needs clean history data | Small sample can misclassify a customer |
| 7 | II | Merge quote, capacity\_flag and repeat\_delay into one triage record with a single next\_action for operations. | Consolidated triage record | Checks were kept separate | High, simple merge logic | Fails silently if a field is missing |
| 8 | III | If status is on\_track, write a 110–130 word email confirming pickup and deadline. | On-track status email | Confirmations delayed behind review | High, low-risk restatement | Must not over-reassure beyond the record |
| 9 | III | If status is at\_risk, write a 110–130 word email naming the conflict and one mitigation, without confirming the deadline. | At-risk status email | No warning before missed deadlines | Medium, tone needs judgement | Tested: ran to 145 words vs the 130-word limit |
| 10 | III | If status is escalate, write a 110–130 word email naming the operations-manager contact; state a decision is pending, no delivery promise (guards against unconstrained reassurance; Ji et al., 2023). | Escalation status email | Reps promised delivery pre-approval | Low, deliberately human-reviewed | Highest-stakes prompt; any drift is a liability risk (NIST, 2024) |

## Iterative development

Iterative development: each prompt was tested and refined through at least one revision cycle. The full version history, test outputs and lessons learned are logged at: https://github.com/dudnikmail-netizen/bus4005-prompt-library

## References

Chui, M., Hazan, E., Roberts, R., Singla, A., Smaje, K., Sukharevsky, A., Yee, L., & Zemmel, R. (2023). *The economic potential of generative AI: The next productivity frontier*. McKinsey & Company. https://www.mckinsey.com/capabilities/mckinsey-digital/our-insights/the-economic-potential-of-generative-ai-the-next-productivity-frontier

Ji, Z., Lee, N., Frieske, R., Yu, T., Su, D., Xu, Y., Ishii, E., Bang, Y. J., Madotto, A., & Fung, P. (2023). Survey of hallucination in natural language generation. *ACM Computing Surveys*, *55*(12), Article 248. https://doi.org/10.1145/3571730

National Institute of Standards and Technology. (2024). *Artificial intelligence risk management framework: Generative artificial intelligence profile* (NIST-AI-600-1). https://doi.org/10.6028/NIST.AI.600-1

White, J., Fu, Q., Hays, S., Sandborn, M., Olea, C., Gilbert, H., Elnashar, A., Spencer-Smith, J., & Schmidt, D. C. (2023). *A prompt pattern catalog to enhance prompt engineering with ChatGPT*. arXiv. https://doi.org/10.48550/arXiv.2302.11382

*Note: the reference list above is not counted toward the report's word limit, per the assessment brief.*
