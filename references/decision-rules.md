# Decision rules

Use these rules while inspecting the target project. The agent collects the
evidence and owns the final judgment; there is no custom scanner or CLI.

## 1. Inventory production dependencies

Use the package manager that owns the lockfile:

```bash
npm ls --omit=dev --depth=0
pnpm list --prod --depth 0
yarn info --name-only --recursive
bun pm ls
```

Do not equate `dependencies` with browser-delivered code. Inspect imports and
the application's server/client boundaries. In a workspace, audit each shipped
application package, not only the workspace root.

Read `package.json` for production dependencies, Browserslist, workspaces, and
existing build/analyzer scripts. Use `rg` or already-installed language tooling
to find static imports, dynamic imports, CommonJS requires, re-exports, wrappers,
configuration, and callers. No import found is not automatically an unused
dependency verdict because aliases, generated files, framework conventions,
and custom loaders may hide usage.

## 2. Measure actual cost

Prefer evidence in this order:

1. Post-tree-shaking bytes attributed by the project's existing analyzer.
2. Source-map analysis of the project's production build.
3. Package registry or Bundlephobia minified/gzip size.
4. Unknown.

Record whether bytes are parsed, minified, gzip, or Brotli. Compare like with
like. Include wrapper code, polyfills, and fallback chunks in replacement cost.

Inspect the target's `package.json` before running commands. Do not install an
analyzer or run an unpinned `npx` package during an audit. Never run deploy,
publish, or release scripts. If a safe existing build/analyzer is unavailable,
mark actual bundle cost unknown and provide a follow-up command.

## 3. Resolve Baseline and audience safety

Prefer current official sources in this order:

1. The target project's installed `web-features` package, if already present.
2. webstatus.dev or the WebDX feature catalog.
3. The MDN page and its Baseline badge.

WebDX data represents status as:

- `high` becomes `widely`.
- `low` becomes `newly`.
- `false` becomes `limited`.
- missing or undefined becomes `unknown`.

Record Newly/Widely dates, support versions when relevant, source URL, and
lookup date. If using an installed package, also record its version. Do not
hard-code status in this Skill because it changes over time.

Compare support with the project's Browserslist targets. If product analytics
are supplied, use them to quantify the affected audience. Do not invent an
acceptable unsupported-user percentage; report the percentage and let the
project's policy decide.

Baseline does not cover every downstream browser, embedded WebView, or
assistive-technology interaction. Preserve explicit testing when those matter.

## 4. Test actual capability coverage

Trace every caller of the dependency or its shared wrapper. Catalog blocker
patterns are prompts for inspection, not automatic KEEP verdicts.

Answer:

1. Which library methods and configuration are actually used?
2. Which behavior is implicit, such as error normalization, retries, locale
   data, focus restoration, time zones, or plugin registration?
3. What new code would reproduce that behavior?
4. Does the replacement change accessibility, performance, or error semantics?
5. Which existing tests prove equivalence?

If the native API only covers some call sites, use `REPLACE_PARTIALLY`. Do not
claim byte savings while another static import keeps the package in the bundle.

## 5. Choose a verdict

| Verdict | Required evidence |
| --- | --- |
| `REMOVE` | Audience-safe, full capability coverage, positive byte savings, and a validation path. |
| `REPLACE_PARTIALLY` | Some call sites are equivalent, but the package cannot yet leave the shipped graph. |
| `PROGRESSIVE` | Feature detection plus a real lazy fallback or simpler fallback reduces bytes for supported browsers. |
| `KEEP` | A demonstrated capability gap, audience incompatibility, or non-positive replacement cost. |
| `INVESTIGATE` | Missing audience, usage, bundle, or semantic evidence prevents a safe decision. |

### Progressive enhancement rule

Feature detection does not reduce shipped JavaScript if the fallback remains a
static import. Verify that the fallback is a dynamic import in a separate chunk,
or that the fallback needs no library at all. Measure both initial and fallback
chunks before using `PROGRESSIVE`.

### Confidence

- `high`: actual bundle bytes, resolved targets, all callers inspected, and
  relevant tests identified.
- `medium`: callers and targets are known, but byte cost is theoretical or one
  behavior requires validation.
- `low`: manifest or catalog match without complete source, audience, or build
  evidence.

Only a high-confidence finding should normally use `REMOVE`.
