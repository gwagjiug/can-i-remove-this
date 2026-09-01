# can-i-remove-this

[English](./README.md) | [한국어](./README.ko.md)

> Can this JavaScript dependency leave the bundle?

`can-i-remove-this` is an Agent Skill that reviews browser-delivered JavaScript
dependencies from a removal-first, evidence-based perspective. It starts with
the target project's `package.json`, but never treats the manifest alone as
proof that a dependency is used, shipped, expensive, or replaceable.

## Operating perspective

### `package.json` is a map, not the application

A production dependency is only a candidate. The Skill traces imports,
re-exports, shared wrappers, callers, workspaces, and browser/server boundaries
before deciding whether the dependency reaches users.

No import found is also not automatic proof that a dependency is unused.
Generated code, aliases, framework conventions, and runtime loading may hide
the real path.

### Shipped bytes matter more than package size

The Skill prefers post-tree-shaking production bundle evidence. Bundlephobia or
registry size is useful for discovery, but remains theoretical until confirmed
against the project's actual build.

A partial migration has zero package-level savings while any shipped static
import keeps the dependency in the bundle.

### The audience matters more than an abstract Baseline badge

Baseline is used as current interoperability evidence, not as a universal
permission to replace code. The Skill compares the native feature with the
project's Browserslist targets or supplied audience analytics.

Baseline Widely is usually strong evidence. Baseline Newly may still be safe for
a modern internal application. Neither replaces testing for important WebViews,
downstream browsers, or assistive technology.

### Actual usage matters more than API resemblance

Two APIs looking similar is not enough. The Skill examines behavior the project
actually depends on: interceptors, retries, parsing, time zones, locale data,
focus restoration, upload progress, error semantics, and plugins.

For example, `fetch` being Widely available does not make Axios removable when
an interceptor performs token refresh and request replay.

### Replacement cost includes the replacement

The comparison includes wrapper code, polyfills, fallbacks, additional tests,
and any behavior that must be rebuilt. If a Temporal polyfill is larger than the
date library being considered, keeping the library can be the smaller choice.

### Progressive enhancement must reduce delivery

Feature detection alone does not ship less JavaScript when the fallback library
is statically imported. The Skill uses `PROGRESSIVE` only when the fallback is
unneeded or loaded as a separate lazy chunk, and reports initial and fallback
costs separately.

### Unknown remains unknown

Missing bundle data is not zero bytes. Missing audience data is not universal
support. A catalog match is not removal approval. When evidence is insufficient,
the verdict is `INVESTIGATE` with the exact missing evidence.

### Audits are read-only by default

The Skill reports findings and recommended validation. It does not modify code,
install analyzers, remove packages, or run deploy/release scripts unless the
user separately requests an authorized implementation.

## The three decision questions

Every candidate is evaluated through the same lens:

1. Is the native replacement safe for this project's actual audience?
2. Does removal still save bytes after wrappers, polyfills, and fallbacks?
3. Does the platform feature cover every capability this project actually uses?

`REMOVE` requires affirmative evidence for all three.

## Verdicts

- `REMOVE`: the dependency can leave the shipped graph now.
- `REPLACE_PARTIALLY`: some callers can migrate, but the dependency remains.
- `PROGRESSIVE`: native support plus a lazy or simpler fallback reduces delivery.
- `KEEP`: a demonstrated compatibility, capability, or cost reason remains.
- `INVESTIGATE`: evidence is missing.

## Run the audit from your `package.json`

The groups in the replacement map are only a starting point. Your dependencies
belong to your project, so the Skill follows the same repeatable five-step audit
for the full production dependency set.

### Step 1. List production dependencies

Start with what may actually ship to users. For npm projects:

```bash
npm ls --omit=dev --depth=0
```

Then read `package.json`, the lockfile, and workspaces, and trace imports,
wrappers, callers, and browser/server build boundaries. A production dependency
is not assumed to be browser-delivered merely because it appears in the list.

### Step 2. Measure each dependency's cost

[Bundlephobia](https://bundlephobia.com/) provides a quick minified and gzip
estimate for published packages. Treat that as theoretical. For the real cost
after tree-shaking and deduplication, prefer the project's production build and
an already-installed analyzer such as `source-map-explorer` or, for Vite,
`vite-bundle-visualizer`.

The Skill does not silently download an unpinned analyzer. If the project has no
safe existing measurement path, it reports actual bundle cost as unknown and
identifies the command or setup needed to measure it.

### Step 3. Check each replacement's current Baseline status

For every candidate, identify the platform feature that could replace it and
verify the current status through [webstatus.dev](https://webstatus.dev/), the
official `web-features` data, or the feature's MDN Baseline badge. Record the
source and lookup date rather than relying on a status copied into this Skill.

### Step 4. Run the three decision questions

For every library with a plausible platform replacement, ask:

1. Is it safe for this project's actual audience when compared with
   Browserslist or supplied analytics?
2. What does replacement really cost after wrappers, polyfills, fallbacks, and
   rebuilt behavior?
3. Does the platform feature cover how this project actually uses the library?

Most decisions are resolved by audience support and actual-use inspection, not
by the package name alone.

### Step 5. Replace behind progressive enhancement where needed

Widely available features are strong replacement candidates only after the
other two questions also pass. For Newly available features, verify that the
audience is sufficiently modern or keep a feature-detected fallback.

The fallback must be absent, simpler, or lazy-loaded in a separate chunk to ship
less JavaScript. A statically imported fallback does not qualify as
`PROGRESSIVE` because supported browsers still receive the library.

The final report records one verdict, confidence, expected savings, and required
validation for each candidate. Dependencies outside the initial map are also
researched when their role overlaps a web platform feature.

## Use

Clone or copy the repository into a skills directory supported by your agent.
For Codex, for example:

```bash
git clone https://github.com/gwagjiug/can-i-remove-this.git ~/.codex/skills/can-i-remove-this
```

There is no install or build step. Invoke it with a request such as:

```text
Use $can-i-remove-this to audit this project. Start from package.json and report
which browser JavaScript dependencies can be removed, what can replace them,
and the actual or currently measurable savings.
```

## Repository structure

```text
can-i-remove-this/
├── SKILL.md
├── references/
│   ├── decision-rules.md
│   ├── replacements.md
│   └── report-template.md
├── README.md
├── README.ko.md
└── LICENSE
```

- `SKILL.md`: entrypoint, workflow, and non-negotiable checks.
- `decision-rules.md`: evidence hierarchy and verdict rules.
- `replacements.md`: candidate native replacements and semantic gaps.
- `report-template.md`: required shape of the final audit report.

## Sources

- [WebDX Baseline](https://web-platform-dx.github.io/)
- [`web-features`](https://github.com/web-platform-dx/web-features)
- [`browserslist-config-baseline`](https://github.com/web-platform-dx/browserslist-config-baseline)
- [`baseline-browser-mapping`](https://github.com/web-platform-dx/baseline-browser-mapping)
- [How Baseline Can Help You Ship Less JavaScript](https://www.smashingmagazine.com/2026/08/how-baseline-can-help-ship-less-javascript/)

## License

MIT
