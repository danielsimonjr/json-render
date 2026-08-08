# Changelog

All notable changes to json-render are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Security

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
