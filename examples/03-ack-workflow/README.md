# Example 03 — The `veripsa-ack` workflow

Walkthrough of the **pause-and-acknowledge** flow: how a `Wait in line`
verdict turns into a soft pause, how an author or agent records an
explicit acknowledgement with the `veripsa-ack` label, and how Veripsa
keeps that acknowledgement honest by tying it to a snapshot of the
coupling.

## Why this exists

The advisory verdict (`conclusion: neutral`) is the right default — it
surfaces overlap without taking control of when you ship.

But some teams want one extra step: when Veripsa says
**Wait in line**, they want the PR to **visibly pause** until somebody
explicitly says "I have seen this and I am proceeding anyway". That is
what the `veripsa-ack` label is for.

Ack is **not**:

- a code-review approval (it is independent of GitHub reviews),
- a claim that the collision is resolved (the structural coupling may
  still be real),
- a permanent opt-out (it is bound to a specific snapshot — see
  [stale acks](#stale-acks) below).

The semantics are specified in
[`veripsa-webhook-spec/ACK_LABEL.md`](https://github.com/GetVeripsa/veripsa-webhook-spec/blob/main/ACK_LABEL.md);
this example shows the day-to-day use.

## Prerequisites

- The Veripsa App is installed on the repo — see
  [example 01](../01-minimal-install/).
- The installation is on the **pause-and-acknowledge** tier. This is
  an opt-in tier; the default tier is advisory-only and does not
  produce `action_required` checks. See
  [veripsa.com/pricing](https://veripsa.com/pricing) for which tiers
  include it.
- You have an open PR that Veripsa has flagged with **Wait in line**
  (see [example 02](../02-first-collision/) for how to produce one).

## What the PR looks like before ack

On the pause-and-acknowledge tier, a `Wait in line` verdict posts:

- a check named `Veripsa — Wait in line (direct collision ahead)`
  with `conclusion: action_required`,
- a PR comment opening with `**Wait in line.**`, naming the partner
  PR by its `PR-<number>` reference, and ending with a short ack
  instruction line.

`action_required` is the **only** non-advisory conclusion Veripsa
ever posts, and it is posted only in this tier and only for a verdict
that already says **Wait in line**. In every other case the check is
advisory (`success` or `neutral`).

A required branch-protection rule on the Veripsa check would
therefore hold this PR until ack is applied — see
[example 04](../04-branch-protection/).

## Adding the ack — agent flow

The standard agent flow uses `gh pr edit` with a token that has
`pull_requests: write` on the base repo:

```bash
# replace 123 with the PR number Veripsa flagged
gh pr edit 123 --add-label veripsa-ack
```

That single label change is the entire contract on the agent side.

Within a few seconds of GitHub firing the `pull_request.labeled` event:

- the check's `conclusion` flips from `action_required` to `neutral`,
- the Veripsa comment is re-rendered to show that the coupling has
  been explicitly acknowledged,
- the comment carries an invisible snapshot marker
  (`<!-- veripsa-ack-snap:<12-hex> -->`) that binds this ack to
  exactly this coupling.

The PR is now mergeable on the Veripsa side. (Your other required
checks and review approvals are independent.)

## Adding the ack — human flow

From the PR sidebar, open **Labels**, search for `veripsa-ack`, and
apply it. The label is created lazily on first need — it does not
need to exist on the repository ahead of time.

Veripsa will re-analyse on the `labeled` event and update the check
and comment as in the agent flow above.

## Removing the ack

Removing the `veripsa-ack` label re-raises the soft pause on the next
event that re-analyses the PR. There is no separate "un-ack" command —
removing the label is the un-ack.

This is intentional: ack is **a fresh decision per coupling**, not a
permanent opt-out for the lifetime of the PR.

## Stale acks

An ack is bound to the **specific coupling** Veripsa described at the
moment the ack was applied — which partner PR, which files, which
symbol set. If that coupling materially changes between events — a
different partner PR comes into the picture, a different set of
colliding files emerges — the prior ack is considered **stale**:

- the pause re-raises on the next analysis,
- the `veripsa-ack` label is slated for removal so the PR visibly
  reads un-acked again,
- the re-rendered comment names the new coupling and asks for a fresh
  ack.

> The previous ack is **not** retroactively voided — the historical
> record stays. What changes is that this PR, as it stands right now,
> is asking for ack against a coupling nobody has acknowledged yet.

The framing matters: ack is a **snapshot acknowledgement**, not a
pardon. The collision may still be real after ack; ack only records
that someone took responsibility for proceeding.

## Common questions

**Q. Can I require ack via branch protection?**
Yes — make the Veripsa check required (see
[example 04](../04-branch-protection/)). On the pause-and-acknowledge
tier, the check stays `action_required` until ack is applied, which
GitHub treats as not-yet-passing.

**Q. Can fork contributors ack their own PR?**
No. Fork PRs cannot carry trusted labels by GitHub's permission model.
Veripsa treats fork PRs as non-material for ack purposes.

**Q. Does Veripsa ever apply `veripsa-ack` itself?**
No. The label is always human-or-agent-applied. Veripsa only **reads**
the label state.

**Q. Does Veripsa change any other label?**
No. Veripsa does not add or remove any label other than `veripsa-ack`,
and only in the stale-ack case where it removes its own previous ack.

## Next

- [Example 04 — branch protection](../04-branch-protection/) — when
  to make the Veripsa check required (and the honest caveats while
  the App is pre-1.0).
