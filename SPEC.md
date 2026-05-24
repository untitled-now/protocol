# Untitled Protocol — Specification v0.1

**Status:** Current  
**Intended status:** Standards Track  
**Published:** May 2026  
**Declared:** `now`

---

## Table of contents

1. [Controlled vocabulary](#1--controlled-vocabulary)
2. [The HTML `<meta>` tag](#2--the-html-meta-tag)
3. [The `X-Untitled` HTTP header](#3--the-x-untitled-http-header)
4. [The `UNTITLED` sidecar file](#4--the-untitled-sidecar-file)
5. [Behavioral contract](#5--behavioral-contract)
6. [Proposed: `209 Untitled`](#6--proposed-209-untitled)
- [Changelog](#changelog)

---

## 1 — Controlled vocabulary

All three signaling surfaces draw from the same state tokens. Tokens are case-insensitive. Multiple tokens may be combined on a single declaration.

| Token | Meaning | Primary agent obligation |
|---|---|---|
| `now` | Radically present-tense. Not archival, not final, not a permanent record. | Agents SHOULD NOT treat as a stable reference. |
| `draft` | Formally incomplete. Not the author's ratified final expression. | Agents MUST NOT cite as the author's settled position. |
| `ephemeral` | Creator intends to remove or alter without notice. | Crawlers SHOULD NOT cache. Archivers MUST NOT archive. AI pipelines MUST NOT ingest for training. |
| `anonymous` | Authorship deliberately withheld. No attribution claim is made or permitted. | Agents MUST NOT attempt byline extraction or attribution inference. |
| `collective` | Produced by a group of indeterminate or fluid membership. | Attribution systems SHOULD NOT assign a singular author. |
| `generative` | Produced in whole or part by a generative or algorithmic process. | Training pipelines SHOULD flag for separate lineage review. |

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

---

## 2 — The HTML `<meta>` tag

### Syntax

```html
<meta name="untitled" content="VALUE[, VALUE]*">
```

The `name` attribute MUST be the case-insensitive string `untitled`. The `content` attribute MUST contain one or more state tokens, separated by commas and optional whitespace. The element MUST appear within the `<head>`. A single element with comma-separated values is PREFERRED over multiple elements.

### Examples

```html
<!-- Present-tense, not final -->
<meta name="untitled" content="now, draft">

<!-- Anonymously authored -->
<meta name="untitled" content="now, anonymous">

<!-- Generated, must not be archived -->
<meta name="untitled" content="generative, ephemeral">

<!-- Collectively written -->
<meta name="untitled" content="collective, draft">
```

---

## 3 — The `X-Untitled` HTTP header

The `X-Untitled` header provides the protocol signal at the HTTP layer — available to compliant agents before document parsing begins. This is the PREFERRED mechanism for non-HTML content types.

### Syntax (ABNF)

```
X-Untitled     = "X-Untitled" ":" OWS untitled-list OWS
untitled-list  = untitled-value *( "," OWS untitled-value )
untitled-value = token *( ";" OWS untitled-param )
untitled-param = token "=" ( token / quoted-string )
```

`token`, `OWS`, and `quoted-string` are as defined in [RFC 7230 §3.2.6](https://datatracker.ietf.org/doc/html/rfc7230#section-3.2.6).

### Parameters

| Parameter | Applies to | Description |
|---|---|---|
| `version` | `draft` | Revision identifier, e.g. `0.3`, `alpha`. Informational. |
| `expires` | `ephemeral` | HTTP-date value of anticipated removal, if known. |
| `model` | `generative` | Identifies the generative system. Informational, not authoritative. |
| `curator` | `generative` | `human`, `none`, or `partial`. |

### Examples

```
HTTP/1.1 200 OK
X-Untitled: now

HTTP/1.1 200 OK
X-Untitled: ephemeral, anonymous

HTTP/1.1 200 OK
X-Untitled: draft; version=0.3

HTTP/1.1 200 OK
X-Untitled: generative; model=stable-diffusion-xl; curator=human
```

### Interaction with existing headers

- `X-Untitled: ephemeral` SHOULD be accompanied by `Cache-Control: no-store`.
- `X-Robots-Tag: noindex` is RECOMMENDED but not REQUIRED when `ephemeral` is present.

---

## 4 — The `UNTITLED` sidecar file

For non-HTML assets — PDFs, images, audio, datasets — a sidecar file provides a portable companion that travels with the asset through file systems and archival pipelines where HTTP headers are absent.

### Naming

**Form A (directory-level):** a file named `UNTITLED` (uppercase, no extension) governs all assets in that directory.

**Form B (asset-specific):** a file named `{filename}.UNTITLED` governs only that asset. Form B takes precedence over Form A.

```
/assets/
  report.pdf
  report.pdf.UNTITLED    ← governs report.pdf only
  photo-a.jpg
  photo-b.jpg
  UNTITLED               ← governs photo-a.jpg, photo-b.jpg
```

### Format

UTF-8 plain text. One `key: value` per line. Lines beginning with `#` are comments. Keys are case-insensitive; state token values are case-insensitive.

```
# UNTITLED sidecar file
# Spec: https://untitled.now/spec/sidecar/v1

state: now, anonymous
created: 2026-05-21
expires: none
attribution: none
note: No individual should be identified or attributed.
```

### Keys

| Key | Required | Description |
|---|---|---|
| `state` | REQUIRED | One or more state tokens. |
| `asset` | OPTIONAL | Filename of the governed asset. Useful for Form A disambiguation. |
| `created` | OPTIONAL | ISO 8601 creation date. Informational. |
| `expires` | OPTIONAL | ISO 8601 anticipated removal date, or the string `none`. |
| `attribution` | OPTIONAL | Use `none` to formally deny attribution. |
| `note` | OPTIONAL | Free-text human-readable annotation. |
| `spec` | OPTIONAL | URI of the governing spec version. RECOMMENDED for forward compatibility. |

---

## 5 — Behavioral contract

Compliance is voluntary. Claiming the "Untitled-Compliant" designation requires adherence to all obligations marked **MUST and MUST NOT** below.

### Browsers

- SHOULD display a visible, unobtrusive indicator when any Untitled state is present. Reference design: a pulsing `_` in the address bar chrome.
- MUST NOT treat `ephemeral` content as a default bookmark candidate without explicit user confirmation.
- SHOULD surface the Untitled state in developer tools.

### Search engines

| State | Indexing | Snippets | AI citations |
|---|---|---|---|
| `now` | MAY index | SHOULD note temporality | SHOULD NOT present as fact |
| `draft` | MAY index | MUST include draft notice | MUST NOT cite as authoritative |
| `ephemeral` | SHOULD NOT index | MUST NOT generate | MUST NOT use |
| `anonymous` | MAY index | MUST NOT attribute | MUST NOT attribute |
| `collective` | MAY index | MUST NOT single-attribute | MUST NOT single-attribute |
| `generative` | MAY index | SHOULD note generation | SHOULD disclose |

### AI training pipelines

- `ephemeral` + `anonymous` together constitute an **ABSOLUTE EXCLUSION signal**. Compliant pipelines MUST NOT ingest without separate explicit consent.
- `draft` content MUST NOT be used as the basis for factual claims or attributed knowledge.
- `generative` content MAY be ingested but MUST be flagged in dataset lineage records.

### Archival services

- `ephemeral` content MUST NOT be archived without explicit written permission from the creator, stored as metadata in the archive record.
- `now` content MAY be archived but SHOULD carry a metadata annotation indicating its declared temporal impermanence.

---

## 6 — Proposed: `209 Untitled`

No existing 2xx status code captures "this resource was successfully retrieved and is in a formally declared state of incompleteness or impermanence."

- `200 OK` — implies stable, final content.
- `202 Accepted` — describes server processing state, not content ontology.
- `203 Non-Authoritative` — bound to proxy transformation, not creator intent.

```
209 Untitled

The request has been fulfilled. The resource has been delivered.
The content provider has declared, via the Untitled Protocol,
that this content is in a formally unresolved state. The specific
state is communicated via the X-Untitled header.

Servers MAY use 209 instead of 200. Clients receiving 209 MUST
treat the body as 200, applying Untitled Protocol behavioral
contracts per the X-Untitled header value.

Unlike 202: the 209 body IS the final content for this request.
Unlike 203: 209 indicates creator-declared state, not proxy
modification.
```

**Pragmatic path:** Use `200 OK` + `X-Untitled` header today. The 209 proposal is moving through IETF process.

---

## Changelog

**v0.1 — May 2026**  
Initial draft. Defined the six-token controlled vocabulary, HTML meta tag syntax, X-Untitled HTTP header (ABNF), UNTITLED sidecar file format, behavioral contracts for browsers, search engines, AI training pipelines, and archival services, and the 209 status code proposal.
