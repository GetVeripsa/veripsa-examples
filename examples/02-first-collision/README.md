# Example 02 — Your first collision verdict

What Veripsa posts back the first time two of your open PRs structurally
overlap. This example walks through the **Wait in line** shape because
it is the loudest and easiest to recognise; the **Heads up** and
**Clear** shapes are noted at the end.

## Prerequisites

- The Veripsa App is installed on your repository — see
  [example 01](../01-minimal-install/).
- The default-branch base analysis has finished (so you are past the
  "Veripsa is now watching" state and getting real verdicts).
- You can open two PRs at the same time on the same repository.

## The setup

You will open two PRs that both touch the same surface — for example,
the same exported function in the same module — without coordinating
first. This mirrors what happens when two parallel agents (or two
people) pick adjacent work without knowing about each other.

> The examples-repo intentionally does not show real diffs. The
> [veripsa.com/try](https://veripsa.com/try) walkthrough uses a public
> sandbox to show the verdict on a concrete PR. This page focuses on
> **what the verdict looks like** so you can recognise it on your own
> repo.

### Step 1 — open PR A

On a branch like `feat/rename-export`, change the **signature** of an
exported function (rename it, add a required argument, change the
return type). Open the PR.

Within a minute, Veripsa posts:

- a check run named `Veripsa — Clear` with
  `conclusion: success` (because PR A is, so far, the only open PR
  touching that surface);
- a PR comment opening with `**Clear to land.**`.

### Step 2 — open PR B

On a different branch like `feat/use-export`, change a **caller** of
that same function in a way that depends on the **old** signature.
Open the PR while PR A is still open.

Within a minute of PR B opening, Veripsa **re-analyses both PRs** and
upserts new output on each:

- on **PR A**: the check title updates to
  `Veripsa — Wait in line (direct collision ahead)` with
  `conclusion: neutral`, and the existing comment is edited in place
  to open with `**Wait in line.**` and reference `PR-<B>`.
- on **PR B**: the check posts as
  `Veripsa — Wait in line (direct collision ahead)` with
  `conclusion: neutral`, and a comment is posted opening with
  `**Wait in line.**` and referencing `PR-<A>`.

You did not refresh anything — Veripsa edits its own check and comment
in place on re-analysis.

## What the verdict comment looks like

The exact wording is allowed to evolve; the **shape** is stable. A
**Wait in line** comment carries:

- a leading bold line such as `**Wait in line.**` that names the
  verdict,
- a one-sentence reason in plain English (paths and symbol names only —
  no code body),
- one or more `PR-<number>` references to the partner PR(s),
- a `Details ↗` link that lands on the comment on the PR that received
  it (not on a bare commit page),
- an invisible HTML marker `<!-- veripsa:PR-<number> -->` at the very
  start of the comment that integrators use to find it.

The full machine-readable shape — check title prefixes, GitHub
`conclusion` mapping, comment marker, idempotent upsert behaviour — is
specified in
[`veripsa-webhook-spec/OUTPUT.md`](https://github.com/GetVeripsa/veripsa-webhook-spec/blob/main/OUTPUT.md).

## What the verdict comment does NOT contain

- **No file contents.** Veripsa is content-free. Even when the verdict
  is about a specific symbol on a specific line, the comment names the
  path, the symbol, and the line range — never the line's text.
- **No "approved" / "rejected" judgement.** Veripsa is not a code
  reviewer. The verdict says structural overlap exists — it does not
  say whose change is correct.
- **No autonomous label changes.** Veripsa does not add or remove any
  GitHub label except in response to a human or agent applying
  `veripsa-ack` (see [example 03](../03-ack-workflow/)).

## Recognising the other verdicts

| You see                                  | It means                                                                                              | Default conclusion |
|------------------------------------------|-------------------------------------------------------------------------------------------------------|--------------------|
| `Veripsa — Clear`                        | No other open PR touches the surface this PR edits.                                                   | `success`          |
| `Veripsa — Heads up` (cross-PR overlap)  | Another open PR touches an adjacent surface. Worth coordinating; not a direct collision.              | `neutral`          |
| `Veripsa — Heads up` (minor overlap on a build / list file) | Two open PRs touch the same lockfile / manifest, but not the same lines.                               | `neutral`          |
| `Veripsa — Wait in line`                 | Two open PRs touch the **same** surface in a way that will conflict structurally — land them in order. | `neutral`          |
| `Veripsa — Unknown`                      | Veripsa could not gather enough signal to call it Clear. Treat as "look manually".                    | `neutral`          |

The `conclusion` column is the default — see
[example 04](../04-branch-protection/) for when to make this required
and [example 03](../03-ack-workflow/) for the `action_required` tier.

## What to do when you see Wait in line

The advisory check does **not** block merge. The right move is the one
that fits your team's process — these are common patterns, not
prescriptions:

- **Land them in order.** Merge PR A first; let PR B rebase onto the
  new surface; let Veripsa re-analyse PR B (the verdict should drop to
  Clear or Heads up).
- **Coordinate the agents.** If both PRs are agent-authored, route the
  Veripsa comment text back to whichever agent has the cheaper rebase.
- **Acknowledge and proceed.** If you have opted in to the
  pause-and-acknowledge tier, apply `veripsa-ack` to explicitly
  record "I have seen this and I am proceeding anyway". See
  [example 03](../03-ack-workflow/).

## Next

- [Example 03 — ack workflow](../03-ack-workflow/) — the explicit
  acknowledgement flow for the soft-pause tier.
- [Example 04 — branch protection](../04-branch-protection/) — when
  to make the Veripsa check a required status check.
