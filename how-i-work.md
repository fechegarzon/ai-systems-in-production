# How I work

I came to engineering from the business side: credit, operations, growth, fundraising. Since late 2025 I ship software myself by directing coding agents. At LQN, in six months, that added up to about 1,000 commits and 286 merged pull requests across 7 repositories, plus the systems in this repo running in production.

What makes that safe is not the agents. It is the structure around them. This page describes it.

---

## 1. Running coding agents in parallel

**One task, one worktree.** I use Conductor to run several Claude Code and Codex sessions at once. Each session gets its own git worktree and branch. Two sessions in the same folder once overwrote each other and a fix was lost, so the rule is strict: never share a working directory.

**`AGENTS.md` is the contract.** Every repo has an entry file that tells an agent what to read first, what it may not touch, which commands to run and what "done" means. `CLAUDE.md` holds the long domain context. Agents read these before writing a line. When I give a non-programmer a repo ([case 07](case-studies/07-origination-engine.md)), the same files are what keep their assistant inside the lines.

**Quality gates are the definition of done.** A short table maps each kind of task to its gate:

| Task | Gate |
|---|---|
| New wiki page or decision | `make wiki-lint` |
| TypeScript change | typecheck, tests, build |
| Codebase-wide QA pass | `make verify` plus a feature contract test |
| Infra change | validated config, updated runbook, explicit rollback |
| Automation that messages people | dry run or limited send, with a log |
| Browser extension that can act | tests, build, demo check, dry run before the real system |

If a gate can't run, the agent must say why, state the residual risk and leave the exact command for someone who can. If it fails, it does not get dressed up.

**Tests are the safety net for code I didn't type.** I can't review every line an agent writes as carefully as a senior engineer would. So I insist on tests that would fail if the behavior I care about broke: an isolated cache database for smoke tests, a fake WhatsApp provider for end-to-end runs, a guard test that fails if a confidential number appears in public text, a privacy gate over published datasets. One QA pass inventoried 72 features with user stories, edge cases and test cases, then ran the gates and fixed what failed, including a vulnerable transitive dependency.

**Handoffs are written.** When a session ends mid-work, it leaves a handoff file: context, files touched, how to test, what is pending. After a handoff I check parity, because fixes have been lost in transfers between agents.

**Production can drift from `main`.** Hotfixes happen. Before deploying, I diff the files on the server against `main` so I don't overwrite a fix that never got merged. Long-term, every server mirrors `main` and changes go through pull requests, including changes the operations agent itself wants to make.

**Repeatable work becomes a skill or a subagent.** Deploying a new document agent is a Claude Code subagent with a step-by-step playbook. Updating the bank-policy knowledge base is a skill that goes PR, merge, deploy. The second time I do something by hand, I write it down for an agent.

**Red CI is never normal.** A check that stayed red for weeks taught people to merge on red. Fix the check or delete it on purpose.

---

## 2. Governing agents

I treat a new agent like a new employee who was handed every password on day one. The job is to take most of them back.

**Least privilege, through my own wrappers.** A connector that exposes 112 tools does not get passed to an agent. I write a small MCP server per system with an allowlist of operations and hard limits: email only inside one label, with no delete; calendar and drive read-only; a scraper capped at 100 items and USD 5 per run; writes only with an explicit confirmation flag that the model cannot set by itself.

**Permissions in code, not in the prompt.** A hook runs before every LLM call, identifies the sender and injects what that person may ask about. The prompt can be talked around; the hook can't. Access is closed by default.

**Contracts, not mailboxes.** Agents read anonymized event contracts produced by deterministic collectors. They never hold live credentials to other people's accounts. Compromising the agent should not compromise anyone's inbox.

**Block at the transport layer.** When an agent spoke in a WhatsApp group despite a prompt rule, the fix went into the bridge: drop all group traffic, return 403 on group sends, and refuse to start if the guard disappears.

**Drafts first, then a ladder.** Agents draft; people send. A category of message can graduate to autonomous sending only with evidence: 200+ reviewed drafts, 90%+ approved without real edits, zero privacy incidents, its own sender identity, sign-off from two of three owners, an append-only log and a pause button.

**Two brakes on anything that messages people.** An `ENABLED` flag and a `DRY_RUN` flag, both required. Production activation is a signed record only the owner can create. Without it, the job runs and sends nothing.

**Verify the side effect, not the return code.** An email job once reported "10 sent" while nothing left the mailbox, and it was connected to a coworker's account by default. Now it re-reads each message from the provider and aborts if the connected mailbox is not the expected sender.

**Watch the watcher from outside.** An agent can't report its own death. A heartbeat on a separate channel, with no LLM in the path, checks the agent every five minutes.

---

## 3. Code decides, the model judges

For decisions, I prefer typed judgments over generated text. A typed-judgment model (I use TypeSafe Jev) answers a narrow question with a label and a probability. Then:

- **code owns the thresholds and the math** (payments, ratios, eligibility, scores);
- **uncertain cases go to a human**, with the band defined in code;
- **the judge version is pinned** so a metric change means the world changed, not the judge;
- **answers are cached by input hash**, so changing weights or thresholds costs nothing;
- **the model selects from candidates code extracted**; it doesn't invent options.

I use this pattern for routing broker messages ([case 04](case-studies/04-ianna-broker-copilot.md)), prioritizing search queries and judging AI answers ([case 08](case-studies/08-seo-geo-ai-visibility.md)), triaging blog comments into leads, and credit decisions ([case 07](case-studies/07-origination-engine.md)). When an LLM recommended a lender that is not one of our partners, eligibility moved into code the same day ([case 05](case-studies/05-mcp-plugin-chatgpt-claude.md)).

---

## 4. The decision log

Every decision that changes architecture, money or access gets a record: context, options with pros and cons, the decision, expected consequences, owner and how reversible it is. There are 60+ of them. Reversals are recorded too, linked to the record that replaced them.

For the big calls I run an "LLM council": five advisers prompted to disagree with each other, an anonymous peer review of their arguments, and a synthesis. It doesn't decide for me. It makes me argue with the strongest version of each option before I commit. The "agent drafts, human sends" rule came out of one of those sessions.

---

## 5. Measured versus estimated

Every number I report carries a label: **measured**, **modeled**, **company result** or **not proven**.

Three examples of what that looks like in practice:

- **Savings.** The operations agent absorbed the routine layer of a coordinator role. I report it as that, not as "a bot replaced a manager", and I report net savings after counting the leadership work that went back to humans.
- **An effect that didn't hold up.** I expected the agent's work queues to improve approval rates. The naive comparison said +3.9 points. Controlling for bank mix and for how the queue was ordered, it shrank to about +2 points, not significant. I reported it as not proven and proposed a randomized holdout.
- **Correcting our own estimate.** My area's manual once projected that automating notary-status checks would add 18 disbursements a month, assuming the manual check was weekly. The documented baseline showed it was twice a day. We rewrote the estimate to about 5 a month (range 1 to 11) and said the real value was instrumentation, not speed.

The operations agent runs under the same rule. It has to separate fact, signal, hypothesis and missing data, and it is not allowed to call activity a result.

---

## What I bring to a team

- I can sit with operators, find the real bottleneck in the data, and ship the fix in days with coding agents.
- I put guardrails in before the incident, and when an incident happens anyway, I fix the layer that allowed it.
- I report results the way an investor or a credit committee would want them: with the denominator, the label and the caveat.
