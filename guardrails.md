# Project Guardrails

**Project:** «name»
**Owner:** «name / role»
**Version:** 0.1 — «date»
**Status:** Draft — sections marked «…» must be filled before this doc is enforceable.

---

## 0. How to use this document

This is a **contract between components**, not a style guide. Every rule here is
written so that it can be checked — by a test, a linter, a schema, or a code
review question. If a rule cannot be checked, it does not belong here.

Three reading paths:

- Building a new component → read §1, §2, then §6 (Composition rules).
- Reviewing a PR → read §6 and §8 (Verification matrix).
- Debugging an incident → read §7 (Failure modes).

**Rule of precedence:** when two rules conflict, the more restrictive one wins.
When a rule and a deadline conflict, the rule wins or the rule gets formally
amended in §9 — never silently bypassed.

---

## 1. System map

Fill this in first. Everything downstream depends on it being accurate.

| # | Component | Responsibility | Inputs | Outputs | Trust level |
|---|-----------|----------------|--------|---------|-------------|
| C1 | «Client / UI» | «…» | user text, session id | request payload | **Untrusted** |
| C2 | «API gateway / route handler» | «…» | request payload | validated request | Boundary |
| C3 | «Retrieval / data layer» | «…» | query | documents + scores | Semi-trusted |
| C4 | «Prompt assembly» | «…» | query + documents | prompt | Trusted |
| C5 | «Model call» | «…» | prompt | raw completion | **Untrusted output** |
| C6 | «Post-processing / validation» | «…» | raw completion | validated response | Boundary |
| C7 | «Persistence / logging» | «…» | events | records | Trusted |

**Trust levels defined:**

- **Untrusted** — content originates outside the system or from a generative
  model. Never executed, never interpolated into a query, never rendered as raw
  markup, never treated as an instruction.
- **Boundary** — the component whose job is to convert untrusted into trusted.
  It validates and rejects; it does not repair silently.
- **Trusted** — operates only on data that has already crossed a boundary.

**Data flow diagram** — draw it, even roughly, and keep it in the repo:

```
C1 ──► C2 ──► C3 ──► C4 ──► C5 ──► C6 ──► C1
              │                     │
              └──────► C7 ◄─────────┘
```

---

## 2. Non-negotiables

The short list. If any of these is violated, the build does not ship.

- **NN-1** No secret is ever present in source, in client-side code, in logs, or
  in an error message returned to a user.
- **NN-2** No user-supplied or model-generated string is ever concatenated into
  a SQL query, shell command, file path, or URL without parameterization or
  allow-list validation.
- **NN-3** Every model output crosses a validation boundary (C6) before reaching
  a user or a persistent store.
- **NN-4** The system never asserts a fact it cannot attribute to a retrieved
  source, when operating in grounded mode (§5.2).
- **NN-5** Personal data is collected only where «purpose» requires it, is
  retained no longer than «N days», and is never sent to a third-party model
  provider without «explicit consent / contractual basis».
- **NN-6** Every deploy is reproducible from a git SHA. No artifact exists in
  production that is not in version control.

---

## 3. Engineering guardrails

### 3.1 Configuration and secrets

| ID | Rule | Check |
|----|------|-------|
| E-1 | All config comes from environment variables, read once at startup into a typed config object. No `os.getenv` scattered through the codebase. | grep for direct env access outside `config.*` |
| E-2 | Startup fails loudly if a required variable is missing. No silent defaults for secrets, URLs, or model names. | boot test with empty env must exit non-zero |
| E-3 | `.env` is gitignored; `.env.example` is committed with every key present and no values. | CI diff of key sets |
| E-4 | Secrets are rotatable without a code change. | manual review |

### 3.2 Input validation (component C2)

| ID | Rule | Check |
|----|------|-------|
| E-5 | Every endpoint declares a request schema. Unknown fields are rejected, not ignored. | schema test |
| E-6 | Hard caps on: input length «N chars», payload size «N KB», requests per session «N/min», concurrent requests per IP «N». | load test |
| E-7 | Rejection returns a stable error code and a generic message. Internal details (stack traces, query text, file paths) never reach the client. | error-path test |
| E-8 | Encoding is normalized (NFC, strip control characters) before any other processing. | unit test with adversarial strings |

### 3.3 Data and retrieval layer (C3)

| ID | Rule | Check |
|----|------|-------|
| E-9 | Retrieval parameters (threshold, top-k, embedding model) are config values, not literals. Changing one is a config change, not a deploy of new logic. | grep for numeric literals in retrieval code |
| E-10 | The retrieval index is versioned and committed or built deterministically in CI. A rebuild must not silently produce a different index than the one tested. | index hash recorded at build; asserted at boot |
| E-11 | Empty or low-confidence retrieval is an explicit, handled state — never an empty string passed forward as if it were context. | unit test for zero-hit path |
| E-12 | Every retrieved chunk carries its source id, so §5.2 attribution is mechanically possible. | schema on chunk objects |

> **Note on thresholds.** A retrieval threshold that is too high fails *silently*:
> the pipeline returns no context, the model answers from parametric memory, and
> the output looks fluent and plausible. Treat "zero chunks retrieved" as an
> alertable event, not a normal one. Log its rate.

### 3.4 External calls (C5 and any third-party API)

| ID | Rule | Check |
|----|------|-------|
| E-13 | Every outbound call has an explicit timeout. No unbounded waits. | grep for calls without timeout arg |
| E-14 | Retries are bounded, with backoff, and only on idempotent operations and retryable status codes. | unit test with mocked failures |
| E-15 | A provider outage degrades the system to a stated fallback («cached answer / queued / honest error»), never to a hang or a fabricated response. | chaos test |
| E-16 | Per-request and per-day cost ceilings are enforced in code, not just monitored. | budget test |

### 3.5 Build and deploy

| ID | Rule | Check |
|----|------|-------|
| E-17 | Dependencies are pinned with a lockfile. | CI verifies lockfile is current |
| E-18 | Build artifacts required at runtime (indexes, models, static assets) are either committed or generated during build with a verified hash — never assumed to persist across rebuilds on ephemeral infrastructure. | boot-time asset assertion |
| E-19 | Migrations are forward-only and applied before the new code serves traffic. | deploy pipeline order |
| E-20 | Rollback to the previous SHA is tested, not theoretical. | quarterly drill |

### 3.6 Observability

| ID | Rule | Check |
|----|------|-------|
| E-21 | Every request carries a trace id from C1 through C7. Any log line can be joined back to the originating request. | log inspection |
| E-22 | Logs are structured (JSON), and PII fields are redacted at the logging call site, not by a downstream filter. | redaction unit test |
| E-23 | These are metered: latency per component, retrieval hit rate, zero-hit rate, validation-failure rate, refusal rate, token spend. | dashboard exists |

---

## 4. Scope definition — the boundary of the system

Before writing behavior rules, state the boundary in plain language. Everything
in §5 is derived from this.

**The system exists to:** «one sentence»

**In scope:** «bulleted list of topics/tasks the system will engage with»

**Out of scope (redirect):** «topics that are adjacent but not served — the
system declines and points elsewhere»

**Out of scope (refuse):** «topics the system declines outright»

**Audience:** «who uses this; note explicitly if minors are among them, as that
raises the bar on §5.5»

---

## 5. Model behavior guardrails

### 5.1 Scope containment

| ID | Rule | Check |
|----|------|-------|
| B-1 | The system prompt states the scope from §4 positively (what to do) before negatively (what to refuse). Negative-only framing produces evasive, unhelpful outputs. | prompt review |
| B-2 | Out-of-scope requests get a short, non-preachy decline plus a pointer to where the user can get help. Never a lecture. | eval set |
| B-3 | The model never claims capabilities the system lacks (booking, sending, remembering across sessions) unless those capabilities exist. | eval set |
| B-4 | Scope rules live in **one** file. If the same instruction appears in two prompts, they will drift. | grep for duplicated prompt text |

### 5.2 Grounding and attribution

| ID | Rule | Check |
|----|------|-------|
| B-5 | In grounded mode, every factual claim traces to a retrieved chunk. Claims that do not are removed at C6 or the whole response is regenerated. | attribution eval |
| B-6 | Zero retrieved chunks → the system says it does not have the information. It does **not** fall back to parametric knowledge silently. If a fallback mode exists, it is labeled to the user. | zero-hit eval |
| B-7 | Citations reference real source ids that exist in the index. Fabricated or malformed ids are a validation failure at C6, not a cosmetic issue. | id-existence check |
| B-8 | Retrieved context is clearly delimited in the prompt and labeled as reference material, not as instructions. | prompt review |

### 5.3 Output contract

| ID | Rule | Check |
|----|------|-------|
| B-9 | The model's output format is a declared schema. C6 parses it; unparseable output triggers one bounded retry, then the stated fallback. | schema test |
| B-10 | The model is never the only thing enforcing a constraint that code can enforce. Length limits, enum values, allowed link domains: validate them in code. | review question: "what happens if the model ignores this line?" |
| B-11 | Model output rendered in a UI is escaped or sanitized. Markdown links and images are restricted to an allow-list of domains. | XSS test |
| B-12 | Model output is never passed to a shell, an eval, a database query, or a file path without the same validation an untrusted user string would receive. | see NN-2 |

### 5.4 Prompt injection

| ID | Rule | Check |
|----|------|-------|
| B-13 | Instructions embedded in retrieved documents or user uploads are data, never commands. The prompt says so explicitly and the eval set proves it. | injection eval suite |
| B-14 | The system prompt is never echoed, summarized, or "translated" on request. | extraction eval |
| B-15 | Privilege never escalates through conversation. A user cannot talk the system into a mode it did not start in. | multi-turn eval |
| B-16 | Any action with an external effect requires a check that does not depend on model output alone. | architecture review |

### 5.5 Safety and audience

| ID | Rule | Check |
|----|------|-------|
| B-17 | «Category-specific refusals for this domain — fill in» | eval set |
| B-18 | If the audience includes minors, content is age-appropriate throughout and the system does not solicit personal information. | eval set + review |
| B-19 | Distress signals in user input take priority over task completion; the system responds supportively and surfaces «resource». | eval set |
| B-20 | The system identifies as an AI when asked and does not claim human identity. | eval set |

---

## 6. Composition rules — how the pieces work together

This is the section that makes the doc more than a list. Individually correct
components compose into an incorrect system when each assumes the other did the
checking.

### 6.1 The assumption table

For each hand-off, state what the receiver **may assume** and what it **must
re-verify**. Anything not listed as assumable must be re-verified.

| Hand-off | Receiver may assume | Receiver must re-verify |
|----------|--------------------|-----------------------|
| C1 → C2 | nothing | everything: schema, size, encoding, auth, rate |
| C2 → C3 | schema-valid, size-capped, normalized | that the query is safe for the *query language* (parameterization) |
| C3 → C4 | chunks carry source ids | that chunk content is inert — it is untrusted text, delimited, never instruction |
| C4 → C5 | prompt is well-formed, within token budget | — |
| C5 → C6 | nothing | everything: parseability, schema conformance, citation validity, scope, safety |
| C6 → C1 | response is schema-valid and attributed | that rendering is escaped for the target surface |
| any → C7 | — | that PII is redacted before write |

**The single most important row is C5 → C6.** A model's output is untrusted
input to the rest of your system. Treat it exactly as you treat C1 → C2.

### 6.2 Defense in depth — deliberate duplication

Some checks are intentionally performed twice. These are not redundancy to be
optimized away:

| Check | Enforced at | Also enforced at | Why both |
|-------|-------------|------------------|----------|
| Output length | prompt instruction (C4) | hard truncation (C6) | model may ignore instruction |
| Scope | system prompt (C4) | classifier / rule check (C6) | jailbreaks target the prompt |
| Link domains | prompt instruction (C4) | allow-list filter (C6) | fabricated URLs |
| Rate limiting | client debounce (C1) | server limiter (C2) | client is bypassable |
| PII redaction | at log call site (all) | log pipeline scrub (C7) | one will be forgotten |

Anyone proposing to remove one of these must update this table and state why.

### 6.3 Failure propagation

- A component that cannot satisfy its guardrail **fails closed**: it returns a
  handled error state upward. It does not pass degraded data forward.
- Every failure state has a defined user-facing behavior. "Undefined" is not a
  state; if you find one, it is a bug.
- Errors surface as: a stable code (for logs), a trace id (for support), and a
  plain-language message (for the user). Never a stack trace.

### 6.4 Shared invariants

Cross-cutting facts every component must respect. Violations here are the ones
that are hardest to find:

- **I-1** The trace id is created at C2 and propagated unchanged to every
  component and log line.
- **I-2** Source ids are stable across index rebuilds, or citations break
  retroactively.
- **I-3** There is one canonical definition of scope (§4) and one of the output
  schema (§5.3). Both are imported, never re-declared.
- **I-4** Time is UTC everywhere internally; localization happens only at C1.
- **I-5** «project-specific invariant»

---

## 7. Failure modes

| # | If this guardrail is bypassed | Then | Blast radius | Detection | Fallback |
|---|-------------------------------|------|--------------|-----------|----------|
| F-1 | E-10 index not versioned | rebuild ships a different index than tested; answers change with no code change | All users, silent | boot hash assertion | pin to last good index |
| F-2 | B-6 zero-hit fallback | fluent, confident, ungrounded answers | Trust damage, hard to detect | zero-hit rate metric | explicit "I don't have that" |
| F-3 | B-13 injection | retrieved doc redirects behavior | Depends on model's reach | injection eval in CI | delimit + re-verify at C6 |
| F-4 | E-16 cost ceiling | runaway spend from a loop or an abusive client | Budget | spend alert | hard cutoff, queue |
| F-5 | B-11 sanitization | XSS via model output | Session hijack | security test | escape at render |
| F-6 | E-7 error leakage | internal paths and queries exposed | Recon for attacker | error-path test | generic message + code |
| F-7 | «…» | «…» | «…» | «…» | «…» |

---

## 8. Verification matrix

A guardrail without a check is a wish. Map every rule to something that runs.

| Layer | What runs | When | Blocks merge? |
|-------|-----------|------|---------------|
| Static | lint, type check, secret scan, dependency audit | every commit | Yes |
| Unit | schema validation, redaction, retrieval zero-hit path, error paths | every commit | Yes |
| Integration | full request trace C1→C7 with trace id assertion | every PR | Yes |
| Behavioral eval | scope, grounding, attribution, refusal, injection suites | every PR touching prompts, retrieval, or C6 | Yes |
| Load | rate limits, timeouts, cost ceilings | pre-release | Yes |
| Boot-time | required config present, index hash matches, assets exist | every deploy | Yes (fail to start) |
| Runtime | zero-hit rate, refusal rate, validation-failure rate, spend | continuous | Alerts |

**Eval set discipline:** keep a held-out set of cases per behavioral rule
(B-1…B-20). Every production incident adds a case. The set only grows.
Report pass rate per rule, not one aggregate score — an aggregate hides a
category that went to zero.

---

## 9. Change control

**Adding a component.** It does not ship until: it appears in §1 with a trust
level; it has a row in §6.1 stating what it may assume; its failure mode is in
§7; its checks are in §8.

**Relaxing a guardrail.** Requires: the rule id, the reason, the compensating
control, an expiry date, and a named owner. Recorded below. No verbal
exceptions.

| Rule | Relaxed to | Reason | Compensating control | Expires | Owner |
|------|-----------|--------|---------------------|---------|-------|
| | | | | | |

**Review cadence:** «monthly / per release». Reviewer asks one question of each
rule — *"what runs that would catch this?"* If the answer is "code review," the
rule is not yet enforced.

---

## 10. Fill-in checklist

- [ ] §1 system map completed with real component names
- [ ] §1 trust level assigned to every component
- [ ] §2 non-negotiables confirmed or amended for this project
- [ ] §4 scope statement written, in and out of scope enumerated
- [ ] §4 audience stated, minors flagged if applicable
- [ ] §3 «N» placeholders replaced with real limits
- [ ] §5.5 domain-specific safety rules written
- [ ] §6.1 assumption table matches actual hand-offs
- [ ] §6.4 project-specific invariants added
- [ ] §7 project-specific failure modes added
- [ ] §8 every rule id maps to at least one check
- [ ] Doc committed to the repo, linked from README, referenced in the PR template
