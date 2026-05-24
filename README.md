# Untitled Protocol

A lightweight, machine-readable standard for declaring the ontological status of web content — whether a page is radically present-tense, formally incomplete, intended to disappear, anonymously authored, collectively produced, or algorithmically generated. The protocol defines three signaling surfaces: an HTML `<meta>` element, an HTTP response header (`X-Untitled`), and a plain-text sidecar file (`UNTITLED`) for non-HTML assets.

**→ [untitled.now](https://untitled.now)**

## Specification

The current specification is [SPEC.md](SPEC.md) — also rendered at [untitled.now/spec/](https://untitled.now/spec/).

## Quick start

```html
<!-- Present-tense content that is not a permanent record -->
<meta name="untitled" content="now">

<!-- Formally incomplete -->
<meta name="untitled" content="draft">

<!-- Creator intends to remove without notice -->
<meta name="untitled" content="ephemeral">

<!-- Authorship deliberately withheld -->
<meta name="untitled" content="anonymous">

<!-- Produced by a group of indeterminate membership -->
<meta name="untitled" content="collective">

<!-- Produced in whole or part by a generative process -->
<meta name="untitled" content="generative">
```

Or via HTTP header:

```
X-Untitled: now
X-Untitled: ephemeral, anonymous
X-Untitled: draft; version=0.3
```

## Tools

- **[untitled.js](https://untitled.now/js/untitled.js)** — drop-in visual indicator (1 kb, no dependencies)
- **[Validator](https://untitled.now/validate/)** — check any URL for an Untitled declaration
- **[Suggest](https://untitled.now/suggest/)** — AI-powered state recommendation
- **[Tools registry](https://untitled.now/tools/)** — implementations and integrations

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[CC0 1.0 Universal](LICENSE) — public domain dedication.
