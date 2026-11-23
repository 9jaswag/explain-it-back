# explain-it-back

You are about to merge, approve, or deploy a change you did not write or do not yet understand. Maybe an AI agent built it and another agent approved it. The tests pass, but if someone asked you to explain what actually changed, you would be improvising.

That is the gap this skill closes. Before the decision, the agent walks the candidate diff with you — the exact version you intend to ship — and asks you to explain the behavior, failure paths, and risks back in your own words. Where the explanation is thin, it coaches the gap and asks again. It only recommends proceeding once you can stand behind the change, not just the reviewer's approval. You can also ask for a practice check when no decision is pending.

It is not a second code review, and it is not a syntax quiz. It is a short, focused check that you can actually own the change: debug it, extend it, and roll it back if it breaks.

## When it activates

This is for the moment you are about to approve, merge, or deploy a change you did not write or do not yet understand. It also applies to a high-risk change even if you have not said you are unsure. You can ask to explain a change back as practice when no decision is pending.

You will recognize it from phrases like:

- "I skimmed it."
- "I only get the gist."
- "The reviewer already signed off."
- "LGTM."
- "Just merge it, I'll read it later."
- "Don't quiz me."

Retries, authentication, payments, data changes, migrations, concurrency, and anything else hard to reverse call for a deeper check. It is also for the quieter version: you can describe the feature, but not what happens when it fails, retries, or runs twice.

For ordinary onboarding or code explanations with no merge, approval, or deployment decision on the table, the agent answers directly. Docs, comments, formatting, and dependency bumps that change no behavior can get a quick sanity check and move on.

## What the check covers

This is not a memory test. You will not be asked to recite variable names or walk through every line. The bar is whether you can explain the change well enough to own it. The depth depends on risk: retries, auth, money, and data get more questions than a copy edit.

By the end, you should be able to explain:

- What behavior changed, and why it was done this way.
- What happens on the normal path, and what happens when it fails, retries, or runs twice.
- Where the risky state lives, and what could go wrong around it.
- What the tests prove, and what they do not.
- How you would notice the change breaking, contain it, and roll it back.

If you can explain those, you can stand behind the merge. If you cannot, that is the gap the check is there to close.

## How it works

The agent reads the diff for the version you intend to ship. Then it asks you one question at a time about the normal path, failure behavior, and risky state. If your answer is thin, it points you to the relevant code, explains the gap, and asks again.

When you can explain the critical parts in your own words, it says you're ready to merge and offers a short comprehension note for the PR. If you can't, it tells you what is still unverified and the fastest way to close it.

## Install

```bash
# Global install (recommended)
npx skills add 9jaswag/explain-it-back --skill explain-it-back -g

# Project-only install
npx skills add 9jaswag/explain-it-back --skill explain-it-back

# Local development (from this repo)
npx skills add . --skill explain-it-back
```

## What it is not

A quick reminder, because this can sound like more process than it is:

- **Not a second code review.** Correctness is the reviewer's job. This is about whether you can own the change.
- **Not a line-by-line quiz.** Behavior, failure modes, and risk matter; variable names and syntax do not.
- **Not a blocker for small changes.** Docs, formatting, and dependency bumps that change no behavior get a quick sanity check, not a quiz.

## The skill

The instructions live in [`skills/explain-it-back/SKILL.md`](skills/explain-it-back/SKILL.md). The skill ships with its [MIT license](skills/explain-it-back/LICENSE) and requires no scripts or dependencies.

## License

MIT. The root [LICENSE](LICENSE) is canonical. The skill includes an identical full copy at [`skills/explain-it-back/LICENSE`](skills/explain-it-back/LICENSE) so standalone installs include the license text.
