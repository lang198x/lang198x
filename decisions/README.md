# Decisions

The family's Lang198x language-tools decision
owns sibling scope: language frontends, analysis and language services/LSP.
Cross-language project decisions belong here; language-specific decisions live
in the implementing repository.

The [agreed first AMOS kit](https://github.com/lang198x/amos/blob/main/decisions/first-language-kit.md)
uses its own repository, a handwritten Rust parser and distributable TextMate
highlighting. No shared AST or parser-framework dependency is adopted.
The [original milestone](../docs/plans/amos-first-parser.md) is retained research. The source-backed
migration inventory (20 October 2026)
identifies candidates without moving code or published APIs.
