# Report template

Use this structure. Omit empty sections, but do not omit unknown evidence.

```markdown
# Can I Remove This? Report

Audited: <project and commit if available>
Generated: <date>
WebDX data: web-features <version>
Audience source: <Browserslist / analytics / unknown>

## Summary

| Verdict | Count | Measured potential |
| --- | ---: | ---: |
| REMOVE | 0 | 0 kB gzip |
| REPLACE_PARTIALLY | 0 | no immediate package savings |
| PROGRESSIVE | 0 | 0 kB initial / 0 kB fallback |
| KEEP | 0 | — |
| INVESTIGATE | 0 | unknown |

State which dependencies were outside the browser bundle or had no catalog
candidate. Do not silently call them safe or necessary.

## Findings

### <package>@<version> — <VERDICT> (<confidence>)

- Current role: <what the project uses it for>
- Import evidence: `<file:line>`
- Actual usage: <methods, options, wrappers, implicit behavior>
- Current cost: <actual/theoretical/unknown and compression unit>
- Native replacement: <API and web-feature IDs>
- Baseline: <widely/newly/limited/unknown with dates>
- Audience: <resolved targets or missing evidence>
- Capability gaps: <none demonstrated / concrete gaps>
- Replacement cost: <wrapper, polyfill, fallback, and chunk behavior>
- Expected saving: <measured range or unknown>
- Required validation: <tests and browser checks>

Decision: <one concise evidence-backed explanation>

## Missing evidence

- <exact missing fact>
- <safe command or artifact that would resolve it>

## Suggested order

1. <high-confidence complete removal>
2. <measurement or experiment>
3. <keep/watch item>
```

## Reporting rules

- Link every source finding to a file and line when possible.
- Keep theoretical and actual byte measurements separate.
- State that partial migration has zero package-level savings until the last
  shipped static import disappears.
- For `PROGRESSIVE`, report initial and fallback chunks separately.
- Never convert `unknown` to zero, safe, unused, or unsupported.
