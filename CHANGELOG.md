# Changelog

All notable changes to the `typescript-senior-review` plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-14

### Added

- Initial public release of the `typescript-senior-review` plugin
- `review-typescript` skill: scope resolution (file / directory / `staged` / `diff` / `pr` / `all`), project context gathering (`tsconfig.json` strictness flags, configured linter, framework, key dependencies), and dispatch of the reviewer subagent
- `senior-typescript-reviewer` agent: Opus 4.6, read-only tools (Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, optional Context7 MCP, optional GoodMem MCP), strict 18-angle review process
- 18 review angles spanning type-system correctness, strict mode adherence, best practices, anti-patterns, architecture, maintainability, compile-time and runtime performance, error handling, security, testing, modern feature adoption, module system, lint consistency, API design, documentation, concurrency, and resource lifecycle
- 5-level severity scale (CRITICAL / HIGH / MEDIUM / LOW / NIT) with concrete code rewrite blocks on every finding
- Marketplace manifest at `.claude-plugin/marketplace.json` for one-step `claude plugin marketplace add` install
- MIT license

[0.1.0]: https://github.com/TheMizeGuy/typescript-senior-review/releases/tag/v0.1.0
