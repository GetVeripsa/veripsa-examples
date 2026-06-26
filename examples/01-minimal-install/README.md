# Example 01 — Day 0: minimal install

The shortest path from "I have not heard of Veripsa" to "Veripsa is
posting verdicts on my PRs". This example assumes you have a GitHub
repository you can install an App on.

## What you will end up with

- The **Veripsa GitHub App** installed on one repository.
- One initial check run on the repository's most recent open PR (or,
  if no PR is open, on the next PR you open).
- Confidence that the App is alive end-to-end, without changing any
  branch-protection rule yet.

## Prerequisites

- A GitHub repository where you have **admin** permission (App
  installs require admin on the target repo or organization).
- The repository's default branch is one of the supported languages
  in the
  [extraction-depth table](https://github.com/GetVeripsa/veripsa-webhook-spec/blob/main/EVENTS.md).
  Languages outside that table install fine — Veripsa just has less
  to say on PRs that touch only unsupported file types.

## Steps

### 1. Install the App

1. Visit [veripsa.com](https://veripsa.com) and follow the
   **Install on GitHub** link, or go directly to the App's GitHub
   install page from the marketing site.
2. Pick the GitHub account or organization that owns the repository.
3. Choose **Only select repositories** and select the one repository
   you want to start with. You can broaden the install later.
4. Approve the requested GitHub permissions. The full permission list
   is documented in
   [`veripsa-webhook-spec/EVENTS.md`](https://github.com/GetVeripsa/veripsa-webhook-spec/blob/main/EVENTS.md).

The App is now installed. Veripsa starts a one-time base analysis of
the default branch in the background. No PR-side output appears yet.

### 2. Confirm Veripsa is watching

The first signal you get from the App is a **"Veripsa is now watching"**
check posted on the next PR event in the installed repository — either
the next PR you open, or, if you already had an open PR, the next push
to it.

To trigger it deliberately:

```bash
# from a fresh branch on the installed repo
git switch -c veripsa-smoke
echo "# veripsa smoke" >> README.md
git commit -am "smoke test for veripsa install"
git push -u origin veripsa-smoke
gh pr create --title "smoke: confirm Veripsa is installed" \
             --body  "Throwaway PR to confirm the Veripsa App posts a check."
```

Within a minute, the PR's **Checks** tab should show a check run named
`Veripsa — watching` (or similar). That confirms the webhook is
flowing end-to-end. You can close the PR without merging.

> If no check appears after a few minutes, see
> [Troubleshooting](#troubleshooting) below.

### 3. (Optional) Wait for the first verdict

If you let the base analysis finish (typically minutes for a small
repo, longer for a larger one), the **next** PR you open against the
default branch will get a real verdict instead of the "watching"
state. The verdict is one of:

- `Veripsa — Clear` (`conclusion: success`),
- `Veripsa — Heads up` (`conclusion: neutral`),
- `Veripsa — Wait in line` (`conclusion: neutral`),
- `Veripsa — Unknown` (`conclusion: neutral`).

The full ladder, including the `pause-and-acknowledge` tier, is
specified in
[`veripsa-webhook-spec/OUTPUT.md`](https://github.com/GetVeripsa/veripsa-webhook-spec/blob/main/OUTPUT.md).

## What you have NOT done yet

This example installs the App. It deliberately does **not**:

- Make the Veripsa check **required** to merge — see
  [example 04](../04-branch-protection/).
- Opt the installation into the **pause-and-acknowledge** tier — see
  [example 03](../03-ack-workflow/).
- Add Veripsa to additional repositories.

Each of those is a separate, reversible step.

## Troubleshooting

- **No check appears on a new PR.**
  Check the App's install on **Settings → Integrations → Applications**
  and confirm the target repository is listed under the install. If
  the repository was not selected during install, the App is silent
  on it by design.

- **The check appears but stays on "Veripsa is now watching".**
  The base analysis of the default branch is still running. The next
  PR event after it finishes gets a real verdict.

- **The check appears but says "Veripsa paused — free tier reached".**
  The installation is beyond its plan's coverage. The PR is not being
  analyzed, but the App is alive. See
  [veripsa.com/pricing](https://veripsa.com/pricing) for the tier
  details.

## Uninstalling

Visit **Settings → Integrations → Applications**, find the Veripsa
install, and click **Uninstall**. The App stops posting immediately and
GitHub revokes the webhook subscription. The base analysis Veripsa
holds for that install is purged on the standard data-retention
schedule documented at
[veripsa.com/trust](https://veripsa.com/trust).

## Next

- [Example 02 — first collision](../02-first-collision/) — what the
  verdict comment looks like the first time two open PRs touch the
  same surface.
