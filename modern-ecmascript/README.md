# Modern ECMAScript Skill for Claude Code

> Stop your AI from writing JavaScript like it's 2020.

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) that ensures Claude uses modern ES2023–ES2026 patterns instead of outdated JavaScript idioms. Built from the official [TC39 proposals](https://github.com/tc39/proposals), [MDN Web Docs](https://developer.mozilla.org/), and engine blogs.

## The Problem

LLMs default to old JavaScript patterns because their training data is heavily skewed toward pre-2023 code. There are years of blog posts and Stack Overflow answers about `[...arr].sort()`, `reduce`-based grouping, and `let resolve; new Promise(...)` — but very little about their modern, cleaner replacements.

This skill fixes that by teaching Claude about the ES2023–ES2026 features that replace those patterns, so you get modern code by default instead of manually correcting it every time.

## What Old Patterns Does This Replace?

| Old Pattern | Modern Replacement | ES Version |
|---|---|---|
| `[...arr].sort(fn)` | `arr.toSorted(fn)` | ES2023 |
| `[...arr].reverse()` | `arr.toReversed()` | ES2023 |
| `arr[i] = val` (mutating) | `arr.with(i, val)` | ES2023 |
| `[...arr].reverse().find(fn)` | `arr.findLast(fn)` | ES2023 |
| `reduce` accumulator for grouping | `Object.groupBy(arr, fn)` | ES2024 |
| `new Set([...a, ...b])` | `a.union(b)` | ES2025 |
| `[...a].filter(x => b.has(x))` | `a.intersection(b)` | ES2025 |
| Manual `for...of` + accumulate | `iter.map().filter().take().toArray()` | ES2025 |
| `if (!map.has(k)) map.set(k, [])` | `map.getOrInsertComputed(k, () => [])` | ES2026 |
| `let resolve; new Promise(r => resolve = r)` | `Promise.withResolvers()` | ES2024 |
| `Promise.resolve().then(fn)` | `Promise.try(fn)` | ES2025 |
| Manual `for await` collection | `Array.fromAsync(iter)` | ES2026 |
| Lone-surrogate regex detection | `str.isWellFormed()` / `str.toWellFormed()` | ES2024 |
| Custom `escapeRegExp()` function | `RegExp.escape(str)` | ES2025 |
| `new Error(msg); e.original = cause` | `new Error(msg, { cause })` | ES2022 |
| `err instanceof Error` (cross-realm) | `Error.isError(err)` | ES2026 |
| `try/finally` for cleanup | `using resource = acquire()` | ES2026 |
| `btoa(String.fromCharCode(...bytes))` | `uint8arr.toBase64()` | ES2026 |
| Manual hex loop | `uint8arr.toHex()` | ES2026 |
| `new ArrayBuffer` + copy to resize | `new ArrayBuffer(n, { maxByteLength })` | ES2024 |
| Kahan summation / naive reduce | `Math.sumPrecise(arr)` | ES2026 |
| `require('./data.json')` / fetch + .json() | `import data from './data.json' with { type: 'json' }` | ES2025 |
| `_.groupBy()`, `_.sortBy()` (lodash) | Native equivalents above | — |

## Features Covered

### ES2023
Array immutable operations (`toSorted`, `toReversed`, `toSpliced`, `with`), `findLast`/`findLastIndex`, Symbols as WeakMap keys, Hashbang grammar.

### ES2024
`Object.groupBy`/`Map.groupBy`, `Promise.withResolvers`, well-formed Unicode strings, RegExp `v` flag (Unicode Sets), `Atomics.waitAsync`, Resizable ArrayBuffer, `ArrayBuffer.transfer`.

### ES2025
Set methods (7 operations), Iterator helpers, `Promise.try`, `RegExp.escape`, duplicate named capture groups, RegExp pattern modifiers, Import attributes, JSON modules, `Float16Array`/`Math.f16round`.

### ES2026
`Math.sumPrecise`, Uint8Array base64/hex methods, `Error.isError`, Explicit Resource Management (`using`/`await using`), `Array.fromAsync`, `Iterator.concat`, JSON.parse source text, `Map.getOrInsert`/`getOrInsertComputed`.

### Stage 3 Watchlist
Temporal (ES2027), Decorators, Source Phase Imports, Deferred Module Evaluation, Joint Iteration, and more — documented as future-aware, not suggested prematurely.

## Why Not Just Search the Web?

Web search results for JavaScript are proportionally dominated by older content. A search for "JavaScript group array by property" returns dozens of `reduce`-based solutions and maybe one mention of `Object.groupBy`. This skill eliminates that bias by providing the modern pattern directly, with engine support notes, so Claude doesn't need to search at all.

## Skill Structure

```
modern-ecmascript/
├── SKILL.md                          ← Router (intent-based lookup table + gotchas)
└── references/
    ├── collections.md                ← Arrays, Sets, Maps, iterators, grouping
    ├── async.md                      ← Promises, async iteration, async collection
    ├── strings-regex.md              ← Unicode, regex, text processing
    ├── errors-resources.md           ← Error.cause, Error.isError, using declarations
    ├── binary-encoding.md            ← TypedArrays, base64/hex, ArrayBuffer
    ├── modules.md                    ← Import attributes, JSON modules
    ├── numbers-dates.md              ← Math.sumPrecise, Temporal
    └── future.md                     ← Stage 3 proposals watchlist
```

The skill uses **progressive disclosure** — Claude loads only the reference file relevant to the current task, not all 1,400+ lines. The SKILL.md router is 140 lines and includes a quick-reference table and a gotchas section targeting Claude's most common old-pattern defaults.

## Installation

### Claude Code (CLI)

Drop the `modern-ecmascript/` folder into your skills directory:

```bash
# Personal (all projects)
cp -r modern-ecmascript ~/.claude/skills/

# Project-specific
cp -r modern-ecmascript .claude/skills/
```

### From GitHub

```bash
# Clone and copy
git clone https://github.com/YOUR_USERNAME/modern-ecmascript-skill.git
cp -r modern-ecmascript-skill/modern-ecmascript ~/.claude/skills/
```

## How It Works

The skill activates automatically when Claude is writing, reviewing, or refactoring JavaScript/TypeScript code. It works in two modes:

**Passive** — Claude silently uses modern patterns instead of old ones when writing new code. Instead of generating `[...arr].sort()`, it writes `arr.toSorted()`.

**Active** — When reviewing or refactoring existing code, Claude identifies old patterns and suggests modern replacements with explanations of why the new approach is better.

The skill includes **engine support awareness**, so Claude will note when a feature requires a minimum Node.js or browser version, and won't suggest ES2026 features without mentioning their current support status.

## Sources

All content is sourced exclusively from official channels:

- [TC39 Proposals Repository](https://github.com/tc39/proposals)
- [TC39 Finished Proposals](https://github.com/tc39/proposals/blob/main/finished-proposals.md)
- [ECMA-262 Specification](https://tc39.es/ecma262/)
- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [V8 Blog](https://v8.dev/blog)
- [WebKit Blog](https://webkit.org/blog/)
- [SpiderMonkey Blog](https://spidermonkey.dev/)

## Contributing

Found an old pattern that Claude still defaults to? Open an issue or PR. The most valuable contributions are:

1. **New gotchas** — patterns where Claude consistently uses the old approach
2. **Support updates** — engine version updates as browsers ship new features
3. **Stage 3 → 4 promotions** — moving proposals from `future.md` to the appropriate reference file when they reach Stage 4

## License

MIT
