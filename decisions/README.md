# Decisions

The [umbrella Lang198x decision](https://github.com/stevehill1981/198x/blob/dd9bdfa0cf272fb87763c15afa45f6b35e0978e2/decisions/lang198x-language-tools.md)
owns sibling scope: language frontends, analysis and language services/LSP.
Cross-language project decisions belong here; language-specific decisions live
in the implementing repository.

The [agreed first AMOS kit](https://github.com/lang198x/amos/blob/main/decisions/first-language-kit.md)
uses its own repository, a handwritten Rust parser and distributable TextMate
highlighting. No shared AST or parser-framework dependency is adopted.
The [original milestone](../docs/plans/amos-first-parser.md) is retained research. The source-backed
[migration inventory](https://github.com/stevehill1981/198x/blob/dd9bdfa0cf272fb87763c15afa45f6b35e0978e2/docs/brainstorms/2026-10-20-lang198x-migration-inventory.md)
identifies candidates without moving code or published APIs.
