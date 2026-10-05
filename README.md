# AI systems in production

**Federico Garzón Rodríguez.** Former fintech founder and CEO (DORA, acquired). Now Head of AI & Growth at LQN Hipotecas.

I build and run the AI agents a VC-backed mortgage company uses every day, and I hold them to loan volume, cost and response-time targets. This repo documents ten of those systems: what each one does, how it is built, what broke, and what it changed.

> **One minute?** Read [Manolito](case-studies/01-manolito-ops-agent.md), [Ianna](case-studies/04-ianna-broker-copilot.md) and [Lessons from production](lessons-from-production.md).

---

## Numbers

| Result | What it measures | Type |
|---|---|---|
| **5,100+ sessions** | Internal operations agent in production since May 2026: 43 users, 75K tool calls | Measured |
| **USD 51K / year** | Operations coordinator role absorbed by the agent (about USD 4.3K/month), at a run cost of USD 274–400/month | Measured |
| **18 days, 34 PRs** | Time to ship a WhatsApp, voice and document copilot for mortgage brokers. Cost per turn cut 10x with prompt caching, about USD 0.41 per broker | Measured |
| **181 leads, 78%** | Leads in the first week of an in-house WhatsApp CRM, and the share with an identified source. Now used by 3 brands, with card payments live | Measured |
| **8–9x** | Daily Google impressions in three weeks (about 150 to 1,000–1,400). Indexed pages went from 7 of 34 to 30 of 41 | Measured |
| **286 PRs, ~1,000 commits** | Merged across 7 repositories in six months, working with coding agents | Measured |
| **111.9%** | July disbursement target reached, single-day record of USD 4.6M, first positive EBITDA month of 2026, while LQN World was rolled out | Company result |
| **USD 8.4M** | Annualized rent guaranteed at DORA, with a 0.8% default rate | Company result (as CEO) |

"Measured" means I can point to the log, the counter or the dashboard. "Company result" means it happened while the system was live, but I do not claim the system caused it.

---

## Systems

| # | System | What it does | Stack |
|---|---|---|---|
| 01 | [Manolito, operations agent](case-studies/01-manolito-ops-agent.md) | Company-wide AI agent on WhatsApp and Telegram. Watches the loan pipeline, drafts follow-ups, answers ops questions with per-user permissions | Agent runtime, MCP, Postgres memory, systemd |
| 02 | [LQN World, gamified operations](case-studies/02-lqn-world-gamified-ops.md) | Turns each team's loan pipeline into daily goals, queues and team podiums for 30+ employees | Python, Cloudflare Pages, WhatsApp |
| 03 | [WhatsApp CRM + voice agent](case-studies/03-whatsapp-crm-voice-agent.md) | Shared inbox and Kanban for leads, unanswered-lead alerts, a voice agent that answers and transfers calls, ad attribution, card payments | TypeScript, XState, Postgres, Pipecat |
| 04 | [Ianna, broker copilot](case-studies/04-ianna-broker-copilot.md) | Pre-qualifies a mortgage against 7 partner banks, reads borrower documents, takes calls. Shipped in 18 days | TypeScript, typed-judgment router, OCR, voice |
| 05 | [MCP plugin for ChatGPT and Claude](case-studies/05-mcp-plugin-chatgpt-claude.md) | Remote MCP server: pre-qualify, estimate payments and submit a case from inside ChatGPT or Claude. Submitted to both app directories | MCP, MCP Apps, Cloudflare |
| 06 | [Document-review agents](case-studies/06-document-review-agents.md) | Four services that check mortgage files before they go to the bank. I took them to production and secured them | FastAPI, Claude and Gemini vision |
| 07 | [Origination engine](case-studies/07-origination-engine.md) | Credit rules engine with an LLM judgment layer for an in-house lender. 40/40 on validation, 19 tests | Python, typed judgments, CI |
| 08 | [SEO, GEO and AI visibility](case-studies/08-seo-geo-ai-visibility.md) | Indexing fix, LLM-assisted prioritization, and a tracker of how ChatGPT, Claude, Gemini and Perplexity mention the brand | Python, Search Console, LLM APIs |
| 09 | [Knowledge system (LLM wiki)](case-studies/09-knowledge-system-llm-wiki.md) | 400+ page company wiki in Git that humans read and agents maintain, synced into the agent's memory | Obsidian, Git, embeddings |
| 10 | [DORA, rental guarantees](case-studies/10-dora-rental-guarantees.md) | The company I co-founded and ran before AI: replaced the co-signer in Colombian leases. Acquired | Underwriting, risk, partnerships |

---

## How I work

- **I run coding agents in parallel**, one task per git worktree, with `AGENTS.md` as the contract, CI quality gates as the definition of done, and tests as the safety net for code I did not type.
- **I govern agents like new employees with too much access.** Least privilege, read-only wrappers, permission checks in code (not in the prompt), drafts before sends, and a written ladder for when an agent earns the right to act alone.
- **I keep a decision log and label every number.** Measured, estimated, company result or not proven. When my own analysis didn't hold up, I said so.

More in [how-i-work.md](how-i-work.md) and [lessons-from-production.md](lessons-from-production.md).

---

## Before AI: DORA

From 2022 to 2025 I was CEO and co-founder of DORA, a rental-guarantee company that replaced the co-signer requirement in Colombian leases. We reached 2,000 tenants in 15 cities, USD 380K ARR and USD 700K a month in guaranteed rent, kept defaults at 0.8%, cut CAC from USD 130 to USD 40, raised USD 264K in equity and USD 101K in debt, reached profitability and were acquired. Underwriting and portfolio risk are why I build AI systems the way I do: rules in code, judgment measured, and every decision reproducible. [Read the DORA case](case-studies/10-dora-rental-guarantees.md).

---

## Contact

- Email: fechegarzon@gmail.com
- LinkedIn: [linkedin.com/in/feche1101](https://linkedin.com/in/feche1101)
- Web: [feche.xyz](https://feche.xyz)
- Based in Bogotá. Open to remote work or relocation (US, Mexico).

---

*Code for these systems belongs to LQN and is private; this repo documents architecture, decisions and results. Public code samples: [mcp-readonly-gateway](https://github.com/fechegarzon/mcp-readonly-gateway), [whatsapp-agent-kit](https://github.com/fechegarzon/whatsapp-agent-kit), [agent-evals](https://github.com/fechegarzon/agent-evals). Also public: [InmoLawyer](https://github.com/fechegarzon/inmolawyer), a lease-risk analyzer I built as fractional Chief AI Officer at Feche.xyz.*

*Docs licensed under [CC BY 4.0](LICENSE).*
