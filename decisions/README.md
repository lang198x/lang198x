# Decisions

The family's Lang198x language-tools decision
owns sibling scope: language frontends, analysis and language services/LSP.
Cross-language project decisions belong here; language-specific decisions live
in the implementing repository.

The [agreed first AMOS kit](https://github.com/lang198x/amos/blob/main/decisions/first-language-kit.md)
uses its own repository, a handwritten Rust parser and distributable TextMate
highlighting. No shared AST or parser-framework dependency is adopted.

[One syntax layer for every Lang198x language](shared-syntax-layer.md) adopts
a shared, dependency-free lossless tree, `lang198x-syntax`. Every front end
uses it, from assemblers and BASICs to C, Pascal, Forth or COBOL, while each
language keeps its own grammar, AST and semantics. The record also covers
dialect families with one repository each, provenance, the formatter,
line-numbered BASICs and original-runtime oracles.
The [original milestone](../docs/plans/amos-first-parser.md) is retained research. The source-backed
migration inventory (20 October 2026)
identifies candidates without moving code or published APIs.
