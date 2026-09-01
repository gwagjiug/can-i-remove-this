---
name: can-i-remove-this
description: Audit browser-delivered JavaScript dependencies and decide whether they can be removed, partially replaced, or progressively replaced with Baseline web platform features. Use when reviewing package.json dependencies, reducing frontend bundle size, replacing libraries with native APIs, or asking whether a dependency is still needed.
license: MIT
metadata:
  author: gwagjiug
  version: "0.1.0"
---

# Can I Remove This?

Produce an evidence-backed dependency retirement report. Treat a package in
`package.json` as a candidate, never as proof that it is removable.

Do not edit source, install packages in the audited project, or remove a
dependency unless the user separately asks for implementation.

## Audit workflow

Read [references/decision-rules.md](references/decision-rules.md) before the
audit. Read [references/replacements.md](references/replacements.md) only for
candidate discovery and known semantic gaps; it is a starting map, not an
exhaustive or authoritative package list.

1. Identify the package root and package manager from `package.json` and the
   lockfile. List installed production dependencies with the package manager's
   native command, such as `npm ls --omit=dev --depth=0`.

2. Inspect the project's source and build boundaries. Use `rg` or existing
   language tooling to find imports, wrappers, configuration, and every caller.
   Separate browser-delivered dependencies from server-only and build-only code.

3. Read existing build and analyzer scripts from `package.json`. When a safe,
   already-installed analyzer is available, measure post-tree-shaking bundle
   cost. Otherwise label registry or Bundlephobia size as theoretical and actual
   cost as unknown. Never install an analyzer or run deploy/release scripts.

4. For each plausible native replacement, verify current Baseline status with
   official WebDX `web-features`/webstatus.dev or the MDN Baseline badge. Record
   the source and lookup date; do not rely on remembered status.

5. Inspect the real use and answer all three:

   - Is the native replacement safe for this project's Browserslist or audience?
   - Does removal still save bytes after wrappers, polyfills, and fallbacks?
   - Does the native feature cover every capability this project actually uses?

6. Assign exactly one verdict: `REMOVE`, `REPLACE_PARTIALLY`, `PROGRESSIVE`,
   `KEEP`, or `INVESTIGATE`. `REMOVE` requires affirmative evidence for all
   three questions.

7. Write the result using
   [references/report-template.md](references/report-template.md). Include file
   evidence, WebDX feature IDs and status dates, browser targets, actual or
   theoretical bytes, capability gaps, confidence, and required validation.

## Non-negotiable checks

- Resolve Baseline status from a current official source. Do not copy a status
  from the replacement guide or memory.
- If Browserslist and audience data are both absent, mark audience safety as
  unknown. Baseline Widely is a useful default reference, not proof about the
  site's users.
- Bundlephobia or package metadata is theoretical cost. Do not present it as
  actual application savings.
- A partial migration does not save package bytes while any static import keeps
  the dependency in the same bundle.
- Feature detection alone does not ship less JavaScript when the fallback is
  statically imported. Verify dynamic loading and chunk separation before using
  `PROGRESSIVE`.
- Baseline does not prove accessibility, WebView behavior, or non-core browser
  support. Preserve explicit testing requirements.
