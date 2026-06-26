# Example 04 — Making Veripsa a required check

When (and when **not**) to make the Veripsa check required to merge,
and how to do it without surprising your team.

## TL;DR

- **Default posture: do not require Veripsa yet.** The advisory
  `neutral` check is enough to get the value while you build trust in
  it.
- **Once you have run Veripsa on your real PR stream for a couple of
  weeks**, and once you have opted into the **pause-and-acknowledge**
  tier (see [example 03](../03-ack-workflow/)), making the check
  required is a reasonable next step.
- **If you require Veripsa, you must also accept** that the App going
  down means PRs queue. That trade-off is yours to make.

## Why the default is "advisory only"

Two reasons.

1. **Veripsa is content-free and structural.** It surfaces overlap
   from paths, symbols, line ranges, and structural relationships —
   it does not read your code body. This is intentional (it is what
   makes Veripsa safe to install on private repos) but it also means
   a small fraction of verdicts are noisier than a body-reading
   reviewer would be. The advisory tier lets you calibrate your own
   "do I trust this signal?" before letting it block.

2. **Veripsa is pre-1.0.** The
   [changelog](https://github.com/GetVeripsa/veripsa-changelog) and
   the [webhook-spec status](https://github.com/GetVeripsa/veripsa-webhook-spec)
   are both clearly labelled as such. Wording of verdicts can still
   evolve. Pre-1.0 software is fine to **rely on** for signal; making
   it **gate** your merges is a stronger commitment.

The advisory default is not a hedge — it is the right posture for
most teams, most of the time.

## When you should require Veripsa

These three conditions, all true:

1. **You have run Veripsa on your real PR stream for at least two
   weeks** without observing a verdict you strongly disagree with on
   reflection. Two weeks is a rule of thumb — what matters is that
   you have seen the verdict on enough real PRs to have an opinion.
2. **You are on the pause-and-acknowledge tier** so that a `Wait in
   line` verdict produces an `action_required` check that has a
   well-defined "what do I click to proceed" answer (the
   `veripsa-ack` label) — see [example 03](../03-ack-workflow/).
3. **You accept the failure mode.** If the Veripsa App is degraded —
   stalled webhook delivery, an outage on our side — required-mode
   means your PRs sit in `action_required` until Veripsa posts a
   verdict. The advisory default does not have this failure mode.

If any one of the three is not yet true, leave Veripsa advisory.

## How to make the check required

GitHub configures required checks on the **branch-protection rule for
the target branch** (usually `main` or `master`).

1. **Open the protection rule.** Repository → **Settings** →
   **Branches** → the rule covering your default branch (create one
   if none exists).
2. **Enable "Require status checks to pass before merging".** Inside
   that, enable "Require branches to be up to date before merging"
   if you want a strict serialised merge order.
3. **Add the Veripsa check by its title prefix.** GitHub's
   required-check picker matches on the check name. Pick the check
   that comes from `Veripsa` — its title starts with `Veripsa — `.
   (If multiple Veripsa checks appear, pick the one named after the
   verdict you care about; the verdict ladder is documented in
   [`webhook-spec/OUTPUT.md`](https://github.com/GetVeripsa/veripsa-webhook-spec/blob/main/OUTPUT.md).)
4. **Save the rule.**

> **Heads up.** GitHub's required-check picker only shows checks it
> has seen post recently in this repository. If Veripsa has not posted
> on this repo yet, install the App first (see
> [example 01](../01-minimal-install/)) and let one PR cycle so the
> check name appears in the picker.

The PR side is now: a Veripsa verdict of `Clear` or `neutral`
satisfies the required check; a verdict of `action_required` (only
possible on the pause-and-acknowledge tier) blocks until ack.

## Backing off

If you make Veripsa required and decide you want to step back:

1. Re-open the branch-protection rule.
2. Remove the Veripsa check from the required list.
3. Save.

The App stays installed and keeps posting verdicts — it just goes
back to advisory. No re-install is needed.

## Honest pre-launch caveats

Read these once before flipping the required toggle.

- **Veripsa is pre-1.0.** Behaviour visibly changes from time to
  time. We announce changes in the
  [changelog](https://github.com/GetVeripsa/veripsa-changelog) and
  freeze the parts integrators branch on (verdict-prefix tokens, the
  comment marker, the `veripsa-ack` label name). Wording of human
  copy may still drift.
- **There is no SLA today.** If Veripsa's webhook processing is
  delayed by an incident, a required check holds PRs until the App
  posts. This is the central trade-off of moving off advisory mode.
- **Cross-owner / cross-repo is out of scope.** Veripsa runs
  same-owner per-repo today. Required-mode does not magically extend
  Veripsa's coverage; it just makes the existing same-owner coverage
  gate your merges.
- **Veripsa is not a code reviewer.** It does not assert correctness.
  A required Veripsa check is a signal that "no structural overlap
  blocks this merge", not "this code is correct". Keep code review.

## What this example deliberately does NOT do

- It does not recommend specific timeouts or retry knobs on the
  GitHub side — those are repo-specific.
- It does not recommend making Veripsa required across an entire
  organisation in one step. Roll it out repo-by-repo so you can
  observe the failure mode on one repo before extending.

## Next

If you have read all four examples and the App is installed and
posting verdicts the way you want, the next things to look at are:

- The integration contract at
  [`veripsa-webhook-spec`](https://github.com/GetVeripsa/veripsa-webhook-spec)
  if you want to build dashboards, agent rules, or log shippers on
  top of Veripsa output.
- The
  [changelog](https://github.com/GetVeripsa/veripsa-changelog) to
  follow behaviour changes.
- [veripsa.com](https://veripsa.com) for pricing and the
  pause-and-acknowledge tier.
