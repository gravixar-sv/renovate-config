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

- **Scheduled** — updates land weekly (early Monday), not as a running stream.
- **Grouped** — all non-major updates batch into a single PR; majors stay separate for attention.
- **Rate-limited** — max 5 concurrent / 2 per hour, so no repo ever floods.
- **Low rebase churn** — branches only rebase when conflicted (`recreateWhen: never`), which stops the "same PR redeploys 5 times" noise on Vercel-connected repos.
- **Lockfile maintenance** — weekly refresh to keep lockfiles healthy.
- **No auto-merge** — every PR waits for a human. (Intentional: several repos deploy `main` straight to production.)
- **`@gravixar-sv/core` disabled** — it lives on a private registry Renovate can't read; its versioning is managed outside Renovate.
- **Override blocks are off-limits** — `overrides` / `pnpm.overrides` / pnpm-workspace `overrides` / `resolutions` are security floors, driven by advisories (Cortex rollups, audit gates), never by version currency. Renovate can only raise the value under a fixed range key, which is a no-op at best and a forced major at worst, and on npm repos the in-range lockfile bump fails with `EOVERRIDE`.
- **Held majors (fleet-wide, by decision)** — `typescript` `<7.0.0` (typescript-eslint refuses the native 7.x compiler at load) and `eslint` / `@eslint/js` `<10.0.0` (eslint-config-next bundles an eslint-plugin-react that crashes under eslint 10). Each rule's `description` in `default.json` names the evidence and the lift condition; lift by deleting the rule. The brain's `stack-drift` report lists both under "Deferred by decision" so they are suppressed from the actionable count, never hidden.

To change behavior for the **whole fleet**, edit `default.json` here — every repo picks it up on its next run.
