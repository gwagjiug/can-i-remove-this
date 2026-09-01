# Native replacement map

This is a candidate-discovery guide, not a removal allowlist. Confirm current
Baseline status from WebDX and inspect actual project usage before deciding.
Research dependencies not listed here when their role overlaps a web platform
feature.

## HTTP clients

`axios`, `superagent` → Fetch API and AbortController

Inspect interceptors, automatic retries, authentication refresh, upload
progress, timeout/cancellation semantics, request transforms, error handling,
and browser/server dual use. Fetch rejecting only on network failures commonly
changes application error behavior.

Web features: `fetch`, `aborting`

## Relative time

`timeago.js` → `Intl.RelativeTimeFormat`

Inspect the logic that selects unit and numeric value, custom locales and
formatters, and automatic refresh scheduling. `Intl.RelativeTimeFormat` formats
a supplied value; it does not decide when “59 minutes” becomes “1 hour.”

Web feature: `intl-relative-time-format`

## Plurals

`pluralize` → potentially `Intl.PluralRules`

This is often not equivalent. `Intl.PluralRules` selects a locale-specific
plural category; it does not turn an English word into its plural form. Keep or
replace the inflection data when the project calls `pluralize()`, `.plural()`,
`.singular()`, or registers irregular/uncountable rules.

Web feature: `intl-plural-rules`

## Number formatting

`numeral` → potentially `Intl.NumberFormat`

Inspect format tokens, parsing/unformatting, rounding, custom locales, and
abbreviations. `Intl.NumberFormat` does not parse formatted input.

Web feature: `intl`

## Duration formatting

`humanize-duration` → potentially `Intl.DurationFormat`

Inspect millisecond-to-duration conversion, rounding, unit selection, and
custom language tables. For Newly available support, include fallback or
polyfill cost and verify whether it is lazy-loaded.

Web feature: `intl-duration-format`

## Deep cloning

`lodash.clonedeep` → potentially `structuredClone`

Inspect cloned values for functions, symbols, DOM nodes, class instances,
property descriptors, prototypes, and transferable objects. Verify expected
error behavior for unsupported values.

Web feature: `structured-clone`

## Grouping

`lodash.groupby` → potentially `Object.groupBy`

Inspect iteratee shorthand, key coercion, and assumptions about the returned
object's prototype. A partial migration has no package-level savings while
another shipped import remains.

Web feature: `array-group`

## Dialog and focus helpers

`a11y-dialog`, `focus-trap`, `body-scroll-lock` → potentially `<dialog>`

Inspect focus placement/restoration, nested overlays, scroll locking, Escape and
backdrop behavior, and assistive-technology behavior. Native availability does
not remove the need for accessibility testing.

Web feature: `dialog`

## Tooltips and popovers

`tippy.js`, `@popperjs/core` → potentially Popover API and CSS anchor positioning

Inspect collision handling, flipping, virtual anchors, cursor following, rich
triggers, keyboard behavior, and interactive content. Check Popover and anchor
positioning separately because their Baseline states may differ.

Web features: `popover`, `anchor-positioning`

## Date libraries

`moment`, `dayjs`, `date-fns` → Temporal is a watch candidate, not a default
removal recommendation.

Inspect current Baseline status, polyfill cost, time zones, daylight-saving
semantics, plugins, parsing, formatting, locale behavior, and mutable versus
immutable operations. A Temporal polyfill may be larger than the library being
considered for removal.

Web feature: `temporal`

## Official research entry points

- WebDX Baseline: <https://web-platform-dx.github.io/>
- Web Platform Status: <https://webstatus.dev/>
- MDN Web Docs: <https://developer.mozilla.org/>
- web-features: <https://github.com/web-platform-dx/web-features>
