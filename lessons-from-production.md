# Lessons from production

Fifteen things that broke in systems I run, what I changed, and the rule I keep because of it. Details are sanitized; the lessons are not.

---

### 1. The bot answered inside a WhatsApp group

**What happened.** The operations agent replied in a work group chat and exposed internal tool activity in front of colleagues. The rule "never write in groups" lived in the prompt. The WhatsApp bridge only filtered the bot's own echoes.

**Fix.** The bridge now drops every group message before the allowlist, returns 403 for any send to a group, and a preflight check stops the service from starting if an update removes the guard.

**Principle.** If a rule matters, enforce it below the model. A prompt is a request; the transport layer is a wall. Prefer a visible outage to a public incident.

---

### 2. A message went to the wrong chat

**What happened.** A recipient ID was inferred from logs instead of taken from an explicit list.

**Fix.** Only explicit, validated recipient IDs. Nothing inferred.

**Principle.** Agents may suggest who to contact. They never guess an address.

---

### 3. A schema change put a dashboard at zero for two weeks, with no error

**What happened.** The BI source behind a team podium changed from a current-state table to a six-column event log. Dates and amounts disappeared. Every builder ran successfully and published zeros.

**Fix.** Builders read the current-state source, check required columns and freshness, and refuse to publish on a mismatch.

**Principle.** Treat every external source as a contract. Know whether you are reading events or state. A dashboard showing zero should be an error, not a number.

---

### 4. A bonus was calculated against last month's target

**What happened.** The server sync preserved a folder of runtime outputs on purpose. A config file lived in that folder, so a target change merged in Git never reached the server. Someone's bonus ran against the old target for weeks.

**Fix.** Config follows `main`; runtime outputs are preserved; the deploy wrapper re-installs itself when it changes; agents and people change the server only through pull requests.

**Principle.** Separate configuration from generated state, and make the server a mirror of the repo, not a second source of truth.

---

### 5. The job said "10 emails sent". None were.

**What happened.** A daily email job stored the provider's return ID and marked each message as sent without checking. All ten IDs returned 404. It was also connected to a coworker's mailbox by default, not the agent's.

**Fix.** The job re-reads each message from the provider's sent folder, and aborts in production if the connected account is not the expected sender.

**Principle.** Verify the side effect, not the return code. And check whose identity your integration is using.

---

### 6. The agent was down for three hours and could not tell anyone

**What happened.** A manual restart failed because a local bridge didn't bind its port in time. The process supervisor auto-restarts crashes, not restarts a human commanded, so the service stayed dead. The agent was also the alerting channel.

**Fix.** An external heartbeat every five minutes on a separate bot channel, with no LLM in the path. It also detects "alive but mute" (repeated rate-limit errors from the model provider) and only writes when the state changes.

**Principle.** Monitoring must not depend on the thing it monitors.

---

### 7. A watchdog turned a small failure into a big one

**What happened.** A watchdog restarted a service in a loop over a recoverable error, and each restart knocked the WhatsApp session over.

**Fix.** Watchdogs act only on specific fatal signatures (for example, "gave up after five reconnection attempts"), not on any error.

**Principle.** Automatic recovery needs the same care as any other automation. A restart is not free.

---

### 8. The disk filled up

**What happened.** Old backups filled the server's disk. The agent couldn't write its state database, scheduled jobs timed out, and the WhatsApp session had to be re-paired by hand.

**Fix.** Freed space from dated backups only, kept the two newest, and never touched the live state.

**Principle.** Every backup needs a retention policy. Disk space is part of uptime.

---

### 9. A platform update wiped the secrets

**What happened.** Updating a managed app's spec without the secret values replaced them with nothing. The app returned 500, and the front end, which didn't handle errors, showed a spinner forever.

**Fix.** Always apply a complete spec, smoke-test the public URL after every deploy, and show errors in the UI.

**Principle.** A deploy is not done when the platform says "active". It is done when a real request works.

---

### 10. "Connected" but dead

**What happened.** An integration's status said connected while its key had been revoked. Separately, a hosted embeddings key was revoked and the agent lost its memory.

**Fix.** Health checks run a real operation, not a status call. Embeddings moved to a local model. When two processes corrupted an embedded database, memory moved to Postgres with a lock on the refresh job.

**Principle.** Test the capability, not the connection. Remove external dependencies from anything the agent can't work without.

---

### 11. The login protected the domain, not the app

**What happened.** Two separate surprises. A zero-trust login placed over a whole domain also blocked a vendor's webhook. And apps behind that login were still reachable at the hosting platform's default URL.

**Fix.** Access rules are scoped by path. Apps restrict ingress or validate the access token themselves.

**Principle.** Map every way in before you call something protected.

---

### 12. The model recommended a lender we don't work with

**What happened.** A broker reported that an AI assistant using our pre-qualification tool recommended a state lender that is not one of our 7 partners.

**Fix.** Eligibility moved into code the same day. The model receives the evaluated list and is told to repeat it as returned.

**Principle.** The model explains; the code decides. Anything that can create a commercial or legal promise is computed, not generated.

---

### 13. The voice agent heard a language nobody was speaking

**What happened.** On real calls, echo tripped voice activity detection and the agent interrupted itself. Background chatter was transcribed as fragments of Hindi and English, each one opened a new turn, she went silent and the call dropped.

**Fix.** Higher detection thresholds, language pinned to Colombian Spanish, at least three words before a caller can interrupt, and a clean shutdown when the caller's audio ends.

**Principle.** Voice agents fail in the room, not in the lab. Test with real calls in real noise.

---

### 14. Paying people on a number that was mostly noise

**What happened.** A 1-to-100 individual score was about to decide bonuses. Its monthly reliability (ICC) was about 0.30. The top performer of the year was first in only 3 of 12 months.

**Fix.** A binary floor per job, judged by its false-negative rate (1.4% to 4.5%), shared cases split 1/n, "not calculable" when data is missing (paid by default), and the pay cut frozen with a hash so it can be reproduced.

**Principle.** Before a metric moves money, measure how reliable it is. Rank people only when the data can carry the weight.

---

### 15. WhatsApp's rules are product requirements

**What happened.** Meta rejected free-text replies outside the 24-hour window. It reclassified our alert templates from Utility to Marketing because the copy was persuasive. Alerts arrived after the window had already closed, and some went out at night.

**Fix.** The window is computed from the lead's last inbound message and fails closed with a template offer. Alert copy is neutral and report-like. Alerts run only during business hours, and the threshold for worked leads dropped from 72 hours to 4.

**Principle.** Read the platform's rules as part of the spec. They decide what you can send, when, and at what price.

---

*Context for each lesson is in the [case studies](README.md#systems).*
