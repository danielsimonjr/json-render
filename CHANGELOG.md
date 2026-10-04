# Changelog

All notable changes to json-render are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Security

- Four overrides added for advisories Dependabot provably could not fix on its own. Its
  updater runs for `brace-expansion`, `js-yaml`, `@ai-sdk/provider-utils` and
  `@humanfs/node` were all failing with `security_update_not_possible` -- for
  brace-expansion it reported latest-resolvable 5.0.9 against lowest-non-vulnerable
  5.0.12 with an EMPTY conflicting-dependencies list, meaning a parent's range caps the
  transitive dep and no in-range update exists. The alerts therefore fired indefinitely
  while no PR could ever open.
- `brace-expansion` 5.0.9 -> 5.0.12, `js-yaml` 5.2.3 -> 5.4.2,
  `@ai-sdk/provider-utils` 4.0.5 -> 4.0.57, `@humanfs/node` 0.16.7 -> 0.16.8.
- Each key is scoped to the range the advisory actually flags, so it goes inert once the
  tree moves past it, and each is capped with `^` rather than `>=`. The cap matters here:
  `@ai-sdk/provider-utils` is at 5.0.53 on latest, so an open-ended bound would have
  pulled a major jump far beyond what the advisory required -- the same trap recorded for
  fast-uri below.
- The lockfile delta is 17 resolutions, every one attributable to these four: the targets
  themselves plus `@ai-sdk/provider`, `eventsource-parser`, `undici`,
  `@humanfs/core` and `@humanfs/types`. No other pinned package moved; sharp, postcss,
  ws, esbuild, ajv, next, flatted, picomatch, minimatch and rollup were each checked and
  are unchanged.
- Verified locally with the same four steps CI runs: lint, type-check, 166 tests across 10
  files, and a full workspace build, all green.
- `next` 16.2.11 -> 16.3.6 in both manifests: `apps/web` via #12 then #14, and
  `examples/dashboard` via #15, which only became visible once #14 landed.
  16.3.6 carries the fix for GHSA-vcvr-r3jv-pc5j, a remote code execution in the
  `next/og` `ImageResponse` handler. 16.3.4 and 16.3.5 are backported bug fixes,
  including two `next/image` disk-cache fixes and a CSP-nonce fix for loading and
  template files.
- The bump moves two pinned-by-override packages transitively, and both stay inside
  their override: `sharp` 0.35.3 -> 0.35.5 (override `sharp@<0.35.0`) and
  `postcss` 8.5.24 -> 8.5.28 (override `^8.5.10`). The rest of the lockfile delta is
  `caniuse-lite`, `baseline-browser-mapping`, `nanoid` and `source-map-js`.
  No override goes inert and nothing is downgraded.
- `fast-uri` 3.1.4 -> 3.1.5 via a pnpm override, clearing the last open advisory. It is
  transitive and its parent pins it, so `pnpm update` could not move it.
- The override is keyed `fast-uri@<3.1.5` -> `^3.1.5`, deliberately: keyed to the range
  actually flagged so it goes inert once the tree moves past it, and capped to `^3.1.5`
  rather than `>=3.1.5` because the open-ended form resolved to **4.1.2**, a major jump far
  beyond what the advisory required.


### Security — all 6 open Dependabot alerts resolved (2026-08-03)

- `brace-expansion` 5.0.6 -> 5.0.9 (two high alerts, needing 5.0.7 and 5.0.8)
- `js-yaml` 5.1.0 -> 5.2.3 (one high needing 5.2.2, two medium needing
  5.2.0 / 5.2.1)
- `sharp` 0.34.5 -> 0.35.3 (high, needs 0.35.0)

brace-expansion and js-yaml moved with a plain in-range `pnpm update -r`; no
manifest change was required.

`sharp` needed a new override. It is an optional dependency of `next`, which
declares `^0.34.5` — and still does at `next@latest` (16.2.12), so no Next
version in existence resolves to a patched sharp. Added
`"sharp@<0.35.0": ">=0.35.0"` to `pnpm.overrides`.

Note on the pre-existing overrides: several are dead. They are version-selector
keyed against major lines the tree has since left behind — `brace-expansion@<1.1.13`,
`brace-expansion@>=2.0.0 <2.0.3` and `js-yaml@<=4.1.1` cannot match the
5.x resolutions now in the lock. They are harmless (they still guard against an
old version reappearing) and were left in place, but they do not and did not fix
these alerts. The new sharp override is keyed to the range actually installed.

Verified: `pnpm build` succeeds across the workspace, including
`apps/web`'s full Next production build against the overridden sharp;
`pnpm test` passes 166/166 across 10 files.
