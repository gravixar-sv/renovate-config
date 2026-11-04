# renovate-config

Shared **Renovate** preset for the Gravixar fleet (`gravixar-sv`). One dial for dependency automation across every repo.

## Why this repo is public

Renovate cannot resolve a preset hosted in a **private** repo when the **consuming** repo is public — it fails with `Cannot find preset's package` and halts all updates. The fleet preset used to live in the private `gravixar-core`, which silently broke Renovate on the public `gravixar-marketing` and `gravixar-demo`. Hosting the preset in this public repo lets **both** public and private repos resolve it.

## How to consume it

Put this in a repo's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>gravixar-sv/renovate-config"]
}
```

That's the whole per-repo config. Everything else is tuned here in [`default.json`](default.json).

## Policy (as of this preset)

- **Mondays only, Pakistan time** — Renovate opens, updates, rebases and auto-merges its branches only on Mondays (`timezone: Asia/Karachi`, cron `* * * * 1`), and `updateNotScheduled: false` stops it re-pushing open PRs the rest of the week (HQ S296; that churn was 42% of October's GitHub Actions bill). The window is the whole day because Renovate also skips auto-merge outside it, and the Mend app visits each repo only about every 4 hours. Security PRs ignore the window.
- **3-day release age (npm)** — a version must be 3 days old before Renovate proposes it, so a burst of releases becomes one push. Security PRs and the lockfile refresh are not delayed.
- **Majors on the Dependency Dashboard** — a major shows as a checkbox on the repo's Dependency Dashboard issue and gets a branch and PR only once someone ticks it. Security majors still open at once.
- **Grouped** — all non-major updates batch into a single PR.
- **Rate-limited** — max 5 concurrent / 2 per hour, so no repo ever floods.
- **Low rebase churn** — branches only rebase when conflicted (`recreateWhen: never`), which stops the "same PR redeploys 5 times" noise on Vercel-connected repos.
- **Lockfile maintenance** — weekly refresh (Mondays) to keep lockfiles healthy.
- **Auto-merge on green, majors excepted** — security PRs and the Monday lockfile refresh (HQ S250), and the weekly all-non-major group (HQ S254), merge themselves once every status check on the PR is green. Renovate checks the status itself (`platformAutomerge: false`), because GitHub's native auto-merge merges at once on a repo with no required checks. Majors always wait for a person. A repo where every merge needs a person opts out in its own `renovate.json` (bs-hub does).
- **`@gravixar-sv/core` disabled** — it lives on a private registry Renovate can't read; its versioning is managed outside Renovate.
- **Override blocks are off-limits** — `overrides` / `pnpm.overrides` / pnpm-workspace `overrides` / `resolutions` are security floors, driven by advisories (Cortex rollups, audit gates), never by version currency. Renovate can only raise the value under a fixed range key, which is a no-op at best and a forced major at worst, and on npm repos the in-range lockfile bump fails with `EOVERRIDE`.
- **Held majors (fleet-wide, by decision)** — `typescript` `<7.0.0` (typescript-eslint refuses the native 7.x compiler at load) and `eslint` / `@eslint/js` `<10.0.0` (eslint-config-next bundles an eslint-plugin-react that crashes under eslint 10). Each rule's `description` in `default.json` names the evidence and the lift condition; lift by deleting the rule. The brain's `stack-drift` report lists both under "Deferred by decision" so they are suppressed from the actionable count, never hidden.

To change behavior for the **whole fleet**, edit `default.json` here — every repo picks it up on its next run.
