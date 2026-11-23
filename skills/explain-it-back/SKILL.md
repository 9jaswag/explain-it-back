---
name: explain-it-back
description: Use when an engineer is about to merge, approve, or deploy a change they did not write or do not yet understand—especially AI-generated changes reviewed by another agent. In that context, also use when they say they skimmed it, only get the gist, or want to merge because the reviewer approved it or said LGTM. Use it too when they ask to explain a change back, including as practice when no decision is pending.
license: MIT
---

# Explain It Back

## Overview

**A review agent's approval is evidence about the code, not evidence about the engineer.**

The engineer who merges is accountable for behavior they may not be able to explain,
debug, or roll back. Before merge, make them demonstrate understanding in their own
words. Your explanation is not the test. Their explanation is.

The technique is **explain it back**: the engineer explains the change to you, in their
own words, before you explain anything to them. This applies whether you wrote the
change, reviewed it, or were just handed it.

## When to use

Start the check when the engineer is about to merge, approve, or deploy a change they
did not write or do not yet understand. In that context, cues include "I skimmed it",
"I only get the gist", "the reviewer approved it", "LGTM", and "just merge it, I'll
read it later". The engineer can also ask to explain a change back as practice when no
decision is pending.

For ordinary onboarding or code explanations with no merge, approval, or deployment
decision pending, answer the request directly.

For a high-risk change, run the check whenever a merge, approval, or deployment is
pending, even if the engineer has not admitted confusion. High-risk areas include
retries, authentication, payments, data changes, migrations, and concurrency.

**Do not use** for docs, comments, formatting, or a dependency bump with no behavior change. A one-line confirmation is enough there; a quiz would be theatre.

## What "understands" means

The engineer can, without reading the code:

1. **Purpose** — what behavior changed and why this approach.
2. **Main path** — inputs, outputs, state, and side effects.
3. **Failure path** — what happens on error, retry, duplicate delivery, or partial failure.
4. **Risk** — the part most likely to break and the blast radius.
5. **Evidence gap** — what the tests do not prove.
6. **Recovery** — for high-risk changes, how to detect, roll back, or contain it.

Not required: line-by-line recall, variable names, style, or every branch.

## Workflow

Copy this checklist:

```
Explain-it-back check:
- [ ] Map the diff yourself
- [ ] Triage risk and pick 3–5 load-bearing questions
- [ ] Ask cold, one at a time, before explaining
- [ ] Probe with counterfactuals
- [ ] Coach gaps and re-check with a new question
- [ ] Decide ready / not ready
- [ ] Offer a short written record
```

### 1. Map the diff yourself

Identify and pin the candidate revision and its base. For a PR, compare its target
and head commits with `git diff <target>...<candidate>`. For a deployment, compare
the source revision of the artifact with the revision it replaces. For a local
candidate, include staged, unstaged, and untracked work only when it is part of the
change the engineer intends to ship. Read the source and tests from the candidate
and check which revision the review covers. If you cannot identify the candidate,
report what is missing and leave readiness open.

Build a private answer key of 3–5 load-bearing facts covering the entry points, data
flow, side effects, failure paths, and what the tests miss.

### 2. Triage risk

| Risk | Examples | Depth |
|---|---|---|
| **High** | data mutations, migrations, auth/permissions, payments, concurrency, retries, idempotency, external side effects, destructive ops, public API, security | 4+ questions, including failure, duplicate/crash, observability, rollback |
| **Medium** | business logic, error handling, new dependency, config change | 3–4 questions on behavior and failure |
| **Low** | docs, comments, formatting, tests-only, no behavior change | One confirmation, or skip |

Risk sets the depth, not whether you check. A small diff can hide a large behavior change.

### 3. Ask before you tell

Set a time box before you start: "three questions, about three minutes." Keep it. If gaps remain when the box expires, stop and report them; do not keep drilling.

Default opening:

> Before I summarize, walk me through what happens when `<specific scenario>`.

If the engineer asks for help during the check, give a short orientation of up to
five behavior-level bullets, then ask them to walk you through one path. Use their
answer as evidence of comprehension.

Rules for questions:

- One at a time. Wait. Do not fill the silence.
- Open-ended. No yes/no, no multiple choice, no leading the witness.
- Ask about behavior and consequences, not definitions.
- Use their words back to them; do not feed them the answer.
- If they are stuck, give a one-sentence pointer to the relevant file or behavior, not the full answer, and ask again.

### 4. Probe with counterfactuals

Pick from these based on the diff:

- "How many times does the provider see this request if the first two attempts fail?"
- "Where does the attempt count live, and why there rather than in the queue?"
- "What happens if the process dies after the side effect but before the ack?"
- "What if the same message is delivered twice?"
- "Which line makes this idempotent / authorized / bounded?"
- "What would you see in production if this were wrong?"
- "What does the test suite not prove?"
- "If we roll back after ten minutes, what happens to work already in flight?"
- "What is the part you are least confident about?"

After a correct answer, ask "why?" once. A restatement is not transfer.

### 5. Score and coach

Mark each load-bearing fact **solid**, **partial**, or **missing**.

- **Solid** — explained in their own words and survived one follow-up.
- **Partial** — needed a prompt or only restated your words. Re-ask with a different scenario.
- **Missing** — they did not know. Point to the exact file and behavior, explain briefly, then ask a new question that requires applying it.

Do not move on because they said "makes sense". Do not accept one correct answer as mastery. If two coaching attempts do not land, read the relevant file together and re-ask.

### 6. Decide

Before giving a readiness verdict, verify that the candidate and base still match
what you assessed. If either changed, inspect the new diff and recheck the affected
answers.

**Ready to merge, approve, or deploy** only when every critical fact is solid, the engineer names at least one real risk or uncertainty, and they can state what the tests do not prove.

**Not ready** if any critical fact is missing or partial. Say what is unverified and offer the fastest way to close it: read two files together, run a scenario, add a test, reduce scope, or revert.

If the engineer chooses to proceed anyway, state the unresolved gaps, get explicit confirmation, and call it **risk accepted**, not **ready**. Do not execute the merge, approval, or deploy while calling it safe.

**If the change is reverted or abandoned:** the merge gate no longer applies, but do not let "revert" end the conversation. Confirm the engineer can explain what is shipping instead, why the change failed, and what the next attempt must do differently.

**If you find a defect while mapping:** report the defect, its impact, and whether it
blocks the intended action. Make edits only when the user has authorized a fix. If
the candidate changes, update the answer key and recheck the affected behavior.

**If you implemented the change:** your memory is not evidence. Use the diff, and require the same answers.

### 7. Offer a short record

Only after the check. Keep it in the PR or merge note, not a new document graveyard:

```markdown
### Change comprehension
- Behavior: ...
- Failure path: ...
- Riskiest part: ...
- What tests do not prove: ...
- If it breaks / rollback: ...
- Verified by: engineer explained ... in their own words
```

## Example

> **Engineer:** "Reviewer approved it, tests pass. Just merge it."
>
> **You:** "Let's take three minutes for four questions. What happens if the email
> provider rejects every attempt?"
>
> **Engineer:** "The worker retries with backoff. After three failed sends, it moves
> the job to the dead-letter queue."
>
> **You:** "How does the worker know it is handling the third attempt?"
>
> **Engineer:** "It reads the attempt count from the job body. Each retry creates a
> new message, so the queue's delivery count starts over."
>
> **You:** "What happens if the process dies after the send succeeds but before the
> worker acknowledges the message?"
>
> **Engineer:** "The queue can deliver the same message again and the worker could
> send a duplicate email."
>
> **You:** "What does the test suite leave unverified about that crash?"
>
> **Engineer:** "It tests retries after provider failures. There's no test for a
> successful send followed by a crash before acknowledgement. I'm out of time."
>
> **You:** "We're at the time box. We covered retry behavior, duplicate sends, and
> what the tests miss. How you'd spot and contain duplicates after deployment is
> still open, so you're not ready yet. Close that gap before deciding: identify how
> duplicate sends would be detected and contained."

## Red flags — stop and ask

- You are about to summarize the diff before asking anything.
- You are about to merge because the reviewer or tests approved it.
- The engineer said "I get the gist" and you moved on.
- You explained it, they said "makes sense", and you treated that as proof.
- You are calling it "ready" while a critical area is still unverified.
- You decided not to ask because they are senior, tired, or in a hurry.
- You found a bug and stopped there, without checking they understand it.
- You are about to write the PR description in your words and call it done.

## Rationalizations

| Excuse | Reality |
|---|---|
| "The reviewer approved it and tests pass" | That is evidence about the code. It says nothing about the engineer. Both can be true while they cannot explain the change. |
| "I explained it clearly" | Explanation is input, not evidence. The engineer must produce the explanation. |
| "They said they understand" | "Yes" is not a demonstration. Ask them to trace a path or predict a failure. |
| "They're senior; this is condescending" | Seniority does not transfer across an unfamiliar AI diff. Frame it as ownership of this change, not competence. |
| "We're out of time" | A five-minute check is cheaper than debugging a change nobody owns. If there is truly no time, do not call it safe; record the gaps. |
| "They explicitly declined" | Respect the decision without calling it safe. State the unresolved risk and label it risk accepted. |
| "I found bugs, so the merge is blocked anyway" | Report the blockers. If a revised candidate is prepared, check that the engineer understands the affected behavior. |
| "I'll fix it myself" | Report the defect. If the user authorizes a fix, verify comprehension on the revised candidate. |
| "The diff is small" | Small diffs hide large behavior changes: retry counts, auth checks, off-by-one errors. |
| "I'll write the PR description" | A description in your words does not prove they can answer a question. |
| "Comprehension can happen after merge" | Then it is not a pre-merge gate. Debugging under pressure is the worst time to learn the change. |
| "They wrote part of it" | Authorship of part does not imply understanding of the AI-generated parts. |

## Common mistakes

- Lecturing for ten minutes, then asking "does that make sense?"
- Quizzing trivia instead of behavior, risk, and failure modes.
- Turning the check into a second code review.
- Requiring line-by-line knowledge and blocking on style.
- Asking five questions at once.
- Letting one correct answer end the check.
- Skipping the check for low-risk changes, then also skipping it for high-risk ones.
- Assessing working-tree edits that are outside the intended candidate.
