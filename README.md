# typescript-senior-review

A [Claude Code](https://claude.com/claude-code) plugin that dispatches an **Opus 4.6** senior TypeScript developer agent to review your code across **18 angles** — quality, type-system correctness, architecture, maintainability, security, performance, error handling, testing, modern feature adoption, and ecosystem fit.

The reviewer is a fresh-context subagent with strict **read-only** tool access. Findings come back severity-tagged with concrete code rewrites, file:line locations, and citations. The orchestrator presents the report and asks which findings to apply — nothing is auto-fixed without your explicit selection.

## What it does

When you invoke the `review-typescript` skill (or ask Claude to review your TypeScript):

1. **Scope resolution** — single file, directory, git diff, staged, PR diff, or whole project
2. **Project context gathering** — `tsconfig.json` strictness flags, configured linter, framework, key deps
3. **Agent dispatch** — fresh-context Opus 4.6 subagent with read-only tools (`Read`, `Grep`, `Glob`, `Bash`, optional MCP tools)
4. **The agent** reads your code, runs `tsc --noEmit` and your linter, reviews against the 18 angles, and returns findings in a strict format
5. **Present results** — the orchestrator displays the verbatim report and asks which findings you want applied

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/TheMizeGuy/typescript-senior-review.git

# 2. Install the plugin
claude plugin install typescript-senior-review@typescript-senior-review

# 3. Restart Claude Code for the plugin to load
```

After restart, verify with `claude plugin list` and look for `typescript-senior-review@typescript-senior-review`.

## Usage

| Invocation | What it reviews |
|---|---|
| `/typescript-senior-review:review-typescript` | Uncommitted + staged TS changes (default) |
| `/typescript-senior-review:review-typescript staged` | Only staged changes |
| `/typescript-senior-review:review-typescript pr` | Diff vs `main`/`master` |
| `/typescript-senior-review:review-typescript src/` | All TS files in `src/` |
| `/typescript-senior-review:review-typescript src/auth/login.ts` | Single file |
| `/typescript-senior-review:review-typescript all` | Entire project (excluding `node_modules`, `dist`, `build`, `.next`, `out`, `coverage`) |

You can also ask Claude in plain English: "review my TypeScript", "check this file for issues", "audit my TS code", "pre-PR TypeScript review". The skill description triggers automatically.

## The 18 review angles

| # | Angle | Examples |
|---|---|---|
| 1 | Type-system correctness | `any`, unsafe casts (`as unknown as`), `!` non-null, missed narrowing, weak generic constraints |
| 2 | Strict mode adherence | `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `useUnknownInCatchVariables` |
| 3 | Best practices | `interface` vs `type`, discriminated unions, `as const` + `satisfies`, branded types, exhaustiveness |
| 4 | Anti-patterns | Enums (vs `as const` objects), namespaces, `React.FC`, `Array.includes` narrowing failure, barrel files, `try/catch` swallowing, `@ts-ignore` over `@ts-expect-error` |
| 5 | Architecture | Module boundaries, circular imports, abstraction layers, dependency direction |
| 6 | Maintainability | Naming, complexity, dead code, comment quality |
| 7 | Compile-time performance | Wide unions, deep generics, recursive conditional types, missing return-type annotations on exports |
| 8 | Runtime performance | V8 anti-patterns, hidden class instability, allocation in loops, hot-path validation costs |
| 9 | Error handling | `throw` vs Result, catch narrowing of `unknown`, `Error.cause` chaining, swallowed errors, AbortSignal handling |
| 10 | Security | Unvalidated external input, prototype pollution, ReDoS, injection, secrets in source/logs, supply chain |
| 11 | Testing | Edge case coverage, mocks at boundaries only, type tests for public APIs, assertion strength |
| 12 | Modern feature adoption | `using` / `await using`, `satisfies`, `const T`, `NoInfer`, inferred type predicates, stage-3 decorators |
| 13 | Module system | `import type` discipline, `verbatimModuleSyntax`, ESM/CJS boundaries, package.json exports |
| 14 | Lint / format consistency | Disabled lint rules, suppression comments, inconsistent style |
| 15 | API design (libraries) | Public surface, breaking-change risk, brand types at boundaries, error types in return signatures |
| 16 | Documentation | JSDoc on public APIs, `@example`, `@deprecated`, README accuracy |
| 17 | Concurrency / async | `Promise.all` correctness (loses concurrent failures vs `allSettled`), race conditions |
| 18 | Resource lifecycle | DB connections, file handles, timers, listeners — leaked or properly disposed (`using`) |

## Output format

Each finding follows this exact template:

```
### [SEVERITY] [Category]: <one-line title>

**File:** `path/to/file.ts:42-58`

**Issue:** Plain-English explanation of what is wrong.

**Why it matters:** Concrete consequences — bug, security risk, performance hit, future maintenance pain.

**Current code:**
```ts
// minimal extract showing the problem
```

**Suggested rework:**
```ts
// concrete rewrite that fixes it, complete enough to apply verbatim
```
```

## Severity scale

| Severity | Meaning | Examples |
|---|---|---|
| **CRITICAL** | Bug, data loss, security vulnerability, will break in production | Unvalidated user input parsed as `any`, race condition, prototype pollution, SQL injection |
| **HIGH** | Will cause real problems soon | Type unsoundness in hot path, missing error handling on async, broken module boundary |
| **MEDIUM** | Should fix; quality / maintainability cost | `any` where `unknown` works, missing exhaustiveness, weak test |
| **LOW** | Nice to fix; consistency or polish | Naming inconsistency, missing JSDoc, could use newer TS feature |
| **NIT** | Personal preference / micro-optimization | Style choices, ordering, idiom variations |

## Components

| Type | Name | Purpose |
|---|---|---|
| Skill | `review-typescript` | User-invoked entry point; gathers scope and dispatches the agent |
| Agent | `senior-typescript-reviewer` | Opus 4.6 reviewer that reads code, runs tooling, returns findings |

## Tool access

The agent is **read-only by design**. It has:

| Tool | Purpose |
|---|---|
| `Read`, `Grep`, `Glob` | Read source files, search patterns |
| `Bash` | Run `tsc --noEmit`, `eslint`, `biome check` |
| `TodoWrite` | Track findings during long reviews |
| `WebSearch`, `WebFetch` | Verify against latest ecosystem state |
| `mcp__plugin_context7_context7__*` (optional) | Live library docs if the [context7](https://context7.com/) plugin is installed |
| `mcp__plugin_goodmem_goodmem__*` (optional) | Semantic memory queries if [GoodMem](https://goodmem.ai/) MCP is configured |

It does **not** have `Edit`, `Write`, or `Agent` access. Findings are advisory — the orchestrator (your main Claude session) applies them based on your selection.

## Optional enhancements

The plugin works fine with just the built-in tools, but findings get better with:

- **[Context7](https://context7.com/) MCP** — live library docs for library-specific findings (zod, fastify, React, etc.). Install: `claude plugin install context7`.
- **[GoodMem](https://goodmem.ai/) MCP** — semantic memory search for cross-session learnings. If you have a Learnings space, pass its UUID in the dispatch prompt and the agent will query it.
- **A local TypeScript knowledge base** — e.g., an Obsidian vault under `~/Claude/vault/TypeScript/` with reference docs the agent can cite. Any path works — just tell the agent where it is.

None are required. The plugin is tested to work without them.

## Why a plugin instead of a skill?

This used to be tempting to build as a standalone skill (469-line instructions loaded inline into your conversation). Two reasons for the plugin / orchestrator + subagent pattern instead:

1. **Fresh context** — the reviewer runs in an isolated subagent that has never seen the conversation that wrote the code. No pattern blindness.
2. **Forced model** — the agent is pinned to `model: opus`. Your main session can be on any model; the reviewer is always Opus 4.6.
3. **Read-only enforcement** — the agent has Read but not Edit/Write. Impossible to "accidentally fix" code mid-review.

## License

MIT. See [LICENSE](LICENSE).

## Credits

Built by [mize](https://github.com/TheMizeGuy). Backed by the [Claude Code](https://claude.com/claude-code) plugin system and Anthropic's Opus 4.6 model.
