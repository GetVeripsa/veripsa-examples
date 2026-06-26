# veripsa-examples

Worked walkthroughs for using **Veripsa Core** — the GitHub App that flags
pre-merge cross-PR collisions for teams running parallel AI coding agents.

Each example is a short, copy-pasteable narrative for one concrete thing
you might do once Veripsa is installed on your repository. The examples
deliberately stay on the **outside** of the App: what you click in
GitHub, what Veripsa posts back, what an integrator can branch on.

> If you want to try the App on a sandbox before installing it on your
> own repo, the live walkthrough lives at
> [veripsa.com/try](https://veripsa.com/try).

## What's here

- [`examples/01-minimal-install/`](./examples/01-minimal-install/) —
  Day 0: install the App on a repository and confirm Veripsa is live.
- [`examples/02-first-collision/`](./examples/02-first-collision/) —
  what the verdict comment looks like the first time two open PRs
  collide.
- [`examples/03-ack-workflow/`](./examples/03-ack-workflow/) —
  the soft-pause flow: snapshot-bound `veripsa-ack` walkthrough.
- [`examples/04-branch-protection/`](./examples/04-branch-protection/) —
  when (and when **not**) to make Veripsa a required check.

Each example is self-contained. Read them in order if you are new to
Veripsa; jump straight to the one you need if you are not.

## What this repo is NOT

- **Not the Veripsa engine.** The engine is closed-source. These
  examples describe what Veripsa shows on your PRs and how to react to
  it — not how it decides anything.
- **Not the integration contract.** That lives at
  [`GetVeripsa/veripsa-webhook-spec`](https://github.com/GetVeripsa/veripsa-webhook-spec)
  — verdict-ladder shape, GitHub `conclusion` mapping, comment marker,
  `veripsa-ack` semantics.
- **Not the changelog.** Behaviour changes are announced at
  [`GetVeripsa/veripsa-changelog`](https://github.com/GetVeripsa/veripsa-changelog).

## Honest scope

- Veripsa is **content-free**: it never reads, stores, or transmits file
  bodies. Only paths, symbol names, line ranges, structural relationships,
  and GitHub identifiers cross the wire.
- Veripsa is **advisory by default**. The check it posts is non-blocking
  unless your branch-protection policy makes it required (see
  [example 04](./examples/04-branch-protection/)).
- Veripsa runs **same-owner per-repo** today. Cross-owner / cross-repo
  collision surfacing is on the roadmap, not something these examples
  promise.

## Cross-links

- Product home — [veripsa.com](https://veripsa.com)
- Try a live walkthrough — [veripsa.com/try](https://veripsa.com/try)
- Integration contract — [`veripsa-webhook-spec`](https://github.com/GetVeripsa/veripsa-webhook-spec)
- What shipped, by date — [`veripsa-changelog`](https://github.com/GetVeripsa/veripsa-changelog)

## Reporting a bad example

If an example is inaccurate, out-of-date, or hard to follow, please open
an issue on this repo. The App's actual behaviour wins, and the example
gets corrected.

We do **not** accept code contributions here — this repo is
documentation only. The Veripsa Core engine is not open-source.

## License

The contents of this examples repo are licensed under
[Creative Commons Attribution 4.0 (CC-BY 4.0)](./LICENSE). The Veripsa
Core engine itself is **not** open-source and is **not** distributed
under this license — only the example text in this repo is.
