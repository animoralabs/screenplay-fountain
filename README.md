# Fountain

> Fountain parser & serializer for TypeScript. Zero dependencies, lossless round-trip, semantic AST.

[![CI](https://github.com/animoralabs/fountain/actions/workflows/ci.yml/badge.svg)](https://github.com/animoralabs/fountain/actions)
[![npm](https://img.shields.io/npm/v/@animora/fountain)](https://www.npmjs.com/package/@animora/fountain)
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](./LICENSE)

**[Fountain](https://fountain.io)** is the plain-text markup standard for screenplays, used across the industry since 2012. This is a clean-room TypeScript implementation of the spec — built for correctness, type safety, and round-trip fidelity.

> **Status: in active development.** v0.1 targets reliable round-trip on real-world screenplays. See [Roadmap](#roadmap).

## Why another Fountain parser?

Existing JavaScript parsers were built a decade ago: regex-driven, untyped, and lossy — they extract text but discard structure, positions, and formatting nuance. This implementation takes a different approach:

- **Semantic AST, not string soup.** Parsing emits [`@animora/screenplay-schema`](./packages/screenplay-schema) — a typed, versioned document model where dialogue blocks are real structures, inline emphasis is a proper mark tree, and element sequence rules are expressed as data.
- **Lossless round-trip.** `serialize(parse(text))` preserves your document. Notes, boneyard content, forced elements, and scene numbers survive the trip.
- **Never throws.** Fountain is designed so that any text is a valid document. Ambiguous input degrades gracefully to action, and the parser reports structured warnings instead of failing.
- **Zero runtime dependencies.** Both packages. Auditable in an afternoon.
- **Source positions on every node.** Line, column, and offset — so you can build editors, linters, and tooling on top.
- **Locale-aware.** First-class support for non-English screenplay conventions (currently `en` and `it`), including localized scene heading prefixes, times of day, and transitions.

## Packages

| Package | Description |
| --- | --- |
| [`@animora/fountain`](./packages/fountain) | Parser and serializer for the Fountain format |
| [`@animora/screenplay-schema`](./packages/screenplay-schema) | Semantic screenplay document model: types, validators, sequence constraints as data |

The schema is deliberately a separate package: it has no opinion about Fountain. Any tool can adopt it as a screenplay interchange model, with Fountain as one of its serialization formats.

## Installation

```bash
npm install @animora/fountain
# or
pnpm add @animora/fountain
```

## Quick start

```ts
import { parse, serialize } from '@animora/fountain';

const { document, warnings } = parse(`
INT. COFFEE SHOP - DAY

MARGO sits alone, rewriting the same line for the tenth time.

MARGO
(without looking up)
It was never about the formatting.
`);

document.elements[0];
// → { type: 'scene_heading', parsed: { prefix: 'INT.', location: 'COFFEE SHOP', timeOfDay: 'DAY' }, ... }

document.elements[2];
// → { type: 'dialogue_block', character: { name: 'MARGO' }, parts: [Parenthetical, DialogueLine] }

const text = serialize(document);
// → round-trips back to Fountain
```

Working with the schema directly:

```ts
import { validate, isDialogueBlock, SEQUENCE_CONSTRAINTS } from '@animora/screenplay-schema';

const result = validate(document);        // structural + sequence validation, never throws
const lines = document.elements.filter(isDialogueBlock);
SEQUENCE_CONSTRAINTS['transition'];       // → ['scene_heading'] — the rules are data, not code
```

## Fountain support

**v0.1 scope:**

- Title page (key/value, well-known keys normalized)
- Scene headings, with structured breakdown and scene numbers (`#1A#`)
- Action, character, dialogue, parentheticals, transitions
- Forced elements (`.` `!` `@` `>`) and centered text (`> ... <`)
- Inline emphasis: `*italic*`, `**bold**`, `_underline_`, and combinations
- Notes `[[...]]` and boneyard `/* ... */` — preserved verbatim, never interpreted
- Page breaks (`===`)

**Not yet supported** (see [Roadmap](#roadmap)): dual dialogue (`^`), sections (`#`), synopses (`=`), lyrics (`~`).

## Locale support

Screenplays are written outside Hollywood too. The parser accepts a `locale` option and the schema ships locale token tables:

```ts
import { parse } from '@animora/fountain';

const { document } = parse(source, { locale: 'it' });
// EST. CASA DI NONNA - NOTTE → recognized as a scene heading
```

A detail we enjoy: `EST.` means *establishing shot* in English and *esterno* (exterior) in Italian. Same token, different semantics — which is exactly why the document model carries a `locale`.

## Scope

This project ends where products begin. It will always be a **format layer**: parsing, validation, serialization, and the document model.

It will never include rendering, pagination, PDF export, or editor UI. If you need a full writing environment on top of this model, that's [Typewrite](https://typewrite.dev) — the AI-native screenwriting platform this project was extracted from.

## Roadmap

- **v0.1** — the scope above, with golden-file tests against real screenplays and round-trip property tests
- **v0.2** — dual dialogue, sections, synopses, lyrics; full spec coverage
- **v1.0** — schema freeze with semver guarantees, expanded test corpus, fuzzing, `fountain` CLI (`check`, `convert`)
- **Later** — FDX (Final Draft) conversion layer

The Fountain spec has been stable since 2012. This is one of the rare open source projects that can genuinely reach *done*.

## Contributing

Issues and PRs are welcome — especially real-world Fountain files that break the parser, and screenplay conventions from locales we don't cover yet.

Before opening a feature PR, please check [Scope](#scope): requests for rendering or export features will be (politely) declined.

## Who's behind this

Built by [Animora](https://animora.it), makers of [Typewrite](https://typewrite.dev). We believe the screenplay format should be open infrastructure — the value is in what you build on top of it.

## License

[MIT](./LICENSE) © Animora Inc.
