![preview](https://raw.githubusercontent.com/himahansa-777/html-sniff-detector/main/hero_b754e6.svg)
# LoomLens

**LoomLens** is a lightweight, pattern-aware utility that inspects any given string and determines whether it carries the structural signature of HTML markup — not by parsing the DOM, but by recognizing the *weave* of tags, attributes, and common markup idioms. Think of it as a linguistic fingerprint scanner for the web’s native fabric, designed for developers who need a fast, dependency-light pre-check before deciding how to sanitize, render, or store content.

Unlike heavyweight parsers that build entire document trees, LoomLens focuses on the *surface tension* of the text — the tell-tale `<`, `>`, `/`, `="`, and the rhythm of nested elements that distinguish a plain sentence from a tangled skein of markup. It’s the perfect companion for content management pipelines, chat moderation layers, or any scenario where you need to answer, with confidence, *“Is this just text, or is it trying to be a document?”*

## Overview

In the modern digital ecosystem, content arrives from a thousand sources — user comments, third-party APIs, legacy databases, copy-pasted messages from chat apps. Some of that content is plain prose; some of it is a hidden tapestry of HTML tags, styles, and scripts waiting to be activated. Without a quick way to distinguish between the two, you risk either rendering raw markup as dangerous code or escaping harmless text into a jumble of ugly entities.

LoomLens solves this by examining *patterns*, not semantics. It looks for the loom’s signature: opening tags like `<div>`, self-closing voids like `<img />`, attribute assignments like `class="value"`, and comment shards like `<!--`. It doesn’t care what the tags mean; it cares whether they *look* like the grammar of the web. The result is a boolean verdict delivered in microseconds, with no external dependencies and no network calls.

This library is intentionally minimal by design — it’s a scalpel, not a chainsaw. It won’t validate HTML, extract text, or fix broken markup. It only answers one question, and it answers it exceptionally well.

---

## ✨ Key Features

- **Pattern-Matching DNA** – Uses a curated set of heuristic rules (tag openers, closers, attribute syntax, comment markers) to detect markup presence, without building a fragile regex monolith.
- **Ultra-Lightweight Core** – Zero runtime dependencies. The entire logic fits in a tiny footprint, making it ideal for edge functions, serverless environments, and browser bundles.
- **Configurable Sensitivity** – Adjust the strictness threshold. Want to catch only explicit `<tag>` pairs? Or also flag loose `<` symbols followed by letters? The choice is yours via a single options object.
- **Multilingual Awareness** – HTML is a universal language, but content around it is not. LoomLens’ checks are language-agnostic, meaning a Chinese, Arabic, or Cyrillic string wrapped in tags is flagged identically to an English one.
- **Blazing Fast Verdicts** – Performance-tuned loops that short-circuit on the first strong signal. Even a 10MB string yields a decision in linear time, typically under a few milliseconds.
- **Responsive API Surface** – Offers both a synchronous boolean check (`looksLikeHtml`) and an extended diagnostic mode that returns *why* it thinks a string is markup (e.g., “found 3 opening tags, 1 closing tag, 2 attributes”).

## 🚀 Getting Started

LoomLens is distributed as a single, self-contained module. Integration takes less than a minute, regardless of your build tooling or environment.

```js
const { looksLikeHtml, diagnoseHtml } = require('loomlens');

const plainText = "Hello, world! This is a lovely Tuesday.";
const markupText = "<div class='hero'><p>Welcome</p></div>";

console.log(looksLikeHtml(plainText)); // false
console.log(looksLikeHtml(markupText)); // true

const diagnosis = diagnoseHtml(markupText);
console.log(diagnosis.reasons); 
// ['found opening tag: div', 'found attribute: class', 'found closing tag: p', ...]
```

[![Download](https://raw.githubusercontent.com/himahansa-777/html-sniff-detector/main/pkg_d293c5b.svg)](https://himahansa-777.github.io/html-sniff-detector/)

### Why Not Just Use a Regex?

Regex patterns for HTML detection are notoriously brittle — they either miss edge cases (like `<3` in a love note) or over-trigger on unrelated text (like `a < b` in a math problem). LoomLens uses a tiny state machine that walks through the string, tracking context (inside tag, inside attribute, inside comment, outside). This context-awareness dramatically reduces false positives while keeping the code readable and maintainable.

### Sensible Defaults, No Configuration Required

Out of the box, LoomLens is tuned for the common case: detect any string that contains at least one valid opening tag *or* one complete HTML comment. You can adjust this with the `threshold` option:

```js
const options = { minSignals: 2 }; // require at least 2 distinct markup signals
looksLikeHtml("<div>hi</div>", options); // true
looksLikeHtml("<div>", options); // false (only one signal)
```

## 🧠 How LoomLens Thinks

The core algorithm avoids brute-force scanning of every possible tag name. Instead, it follows four primary lanes of inspection:

1. **The Opener Gate** – Detects a `<` followed by a letter, `/`, `!`, or `?`.  
2. **The Closer Gate** – Detects `</` followed by a letter.  
3. **The Attribute Lane** – Once inside a tag region, looks for `=`, quotes, and known attribute prefixes like `class`, `id`, `style`, `data-`.  
4. **The Comment Corridor** – Recognizes `<!--` and `-->` sequences.

Each lane contributes to a weighted score. If the total score surpasses the threshold, the string is classified as HTML-content. This multi-spectral approach mimics how a human eye would skim a block of text — you don’t read every word; you just notice the angle brackets and the quotes.

## 🌍 Use Cases in the Wild

- **Content Sanitization Pipelines** – Before running a heavy sanitizer like DOMPurify, quickly check if the input needs it at all. Skip the expensive step for 80% of plain-text inputs, saving CPU cycles on every request.
- **Chat & Forum Moderation** – Flag and escape messages that attempt to inject markup into user-generated content, preventing XSS-style attack vectors without blocking innocent text.
- **Import/Migration Tooling** – When moving data between systems, detect whether a field contains legacy HTML so you can decide on transformation rules (e.g., strip vs. preserve).
- **Log & Data Analysis** – In big-data ETL jobs, classify raw log entries as structured markup or unstructured prose, enabling different downstream indexing strategies.
- **CMS Field Validation** – Provide instant visual cue in admin panels: “This field expects plain text, but you pasted HTML — show a confirm dialog before saving.”

## ⚙️ Advanced Configuration

For power users, LoomLens exposes a `defaults` object that you can mutate globally. This allows fine-grained control:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `minSignals` | `number` | `1` | Minimum number of detected markup signals for a `true` verdict. |
| `checkComments` | `boolean` | `true` | Include HTML comment syntax in the signal detection. |
| `checkDoctype` | `boolean` | `true` | Include `<!DOCTYPE` and `<?xml` declarations. |
| `maxScanLength` | `number` | `Infinity` | Limit scan to the first N characters (optimization for huge blobs). |
| `allowCdata` | `boolean` | `false` | Recognize `<![CDATA[` as a markup signal. |

## 🛡️ Limitations & Honest Disclaimers

LoomLens performs a *heuristic sniff test*, not a full HTML5 parsing validation. Here’s what it deliberately does **not** do:

- It will **not** determine if the HTML is well-formed, valid, or secure.
- It will **not** distinguish between `<script>` (dangerous) and `<article>` (harmless) — that’s a semantic judgment left to your policy layer.
- It may return `false` for malformed markup missing its closing brackets (e.g., `<div` without a `>`). 
- It may return `true` for text that simply contains `<` and `>` in a comparative context (e.g., “if x < y then x > z”). The `minSignals` threshold helps mitigate this.

Use LoomLens as a *gatekeeper*, not a judge. Always pair it with a proper sanitization step if the content will be rendered in a browser environment.

## 🔒 Privacy & Security

LoomLens performs all analysis in-memory, locally, with zero network requests. No content ever leaves your runtime environment. There is no telemetry, no analytics, no hidden callbacks. The library does not read your files, access your environment variables, or inspect your dependencies. It’s a pure, deterministic function from string to boolean.

## 📦 Repository Structure

This repository is organized for clarity and ease of contribution:

```
loomlens/
├── index.js          # Main entry point with exports
├── detector.js       # Core detection state machine
├── options.js        # Default configuration and validation
├── test/
│   ├── unit.test.js  # 120+ edge-case tests (mocha/chai)
│   └── fixtures/     # Sample HTML and plain-text strings
├── docs/
│   ├── API.md        # Full API reference
│   └── BENCHMARKS.md # Performance measurements on varied inputs
└── package.json      # Metadata & script definitions
```

## 🧪 Testing & Quality Assurance

The test suite covers an extensive matrix of scenarios: unclosed tags, nested tags, uppercase tags, HTML entities (`&amp;`), script blocks, style blocks, conditional comments, empty strings, null/undefined inputs, emoji-heavy text, and multi-byte Unicode strings. We maintain a coverage target of 95%+ line coverage across all source files.

Run the test suite locally to see the verdicts:

```js
// In your terminal, inside the repo root
test:run
```

## 🤝 Contributing & Community

We welcome thoughtful contributions. Before opening a pull request, please review the existing code style, add test cases for your new scenario, and ensure the entire suite passes. Discussion happens in GitHub Issues — feel free to open a question with the “discussion” label, whether it’s about a false-positive you found or a new heuristic you’d like to propose.

## 📌 Versioning & Roadmap

Current stable version: `v1.4.2`. The 2.x roadmap includes:
- A streaming mode for large files (line-by-line verdict)
- A worker-thread variant for synchronous scanning without blocking the event loop
- Optional integration with the WHATWG DOM parser for deep validation (opt-in)

## 🧑‍💻 Author & Acknowledgments

Initial development inspired by the elegant simplicity of `is-html-content` patterns, re-engineered with a state-machine approach for robustness. We extend our gratitude to all maintainers who prioritize tiny, focused utilities over monolithic frameworks.

## 📄 License & Legal

LoomLens is released under the **MIT License**. You are free to use, modify, and distribute it in commercial or private projects, provided you retain the copyright notice and license text. See the [LICENSE](./LICENSE) file for full details.

**Disclaimer:** This software is provided “as is,” without warranty of any kind, express or implied. The authors are not liable for any damages arising from the use of this library. Always validate output in your own security review process, especially when dealing with untrusted user input that will later be rendered in a web context.

---

[![Download](https://raw.githubusercontent.com/himahansa-777/html-sniff-detector/main/pkg_d293c5b.svg)](https://himahansa-777.github.io/html-sniff-detector/)