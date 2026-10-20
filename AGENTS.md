# Lang198x flagship

Read [PRINCIPLES.md](PRINCIPLES.md), MANIFESTO.md and PRACTICE.md. Follow the
[org context](https://github.com/lang198x/.github/blob/main/AGENTS.md) and
[umbrella ownership decision](https://github.com/stevehill1981/198x/blob/dd9bdfa0cf272fb87763c15afa45f6b35e0978e2/decisions/lang198x-language-tools.md).

This is the coordination/research hub. The AMOS implementation, current corpus,
plans and per-language decisions live in https://github.com/lang198x/amos.
The original proposal and corpus here are retained research snapshots, not
current implementation status or a second canonical test suite. No LSP or
compiler implementation is present here. Keep implementation files small, with
one supported instruction per Rust module in the AMOS repository.

- Preserve source spelling and locations; distinguish unsupported, invalid and
  unknown compatibility. A native syntax test also performs static checks and
  is not a pure grammar oracle.
- Model product, release, extensions and source representation explicitly.
- Separate parsing, static checks, semantics, file codecs and protocol delivery.
- Keep one canonical analysis implementation; editor and CLI adapters share it.
- Record independent original-runtime expectations and negative controls before
  implementing a supported surface. Do not hard-code curriculum examples.
- Published crates use `lang198x-`; Rust implementation follows edition 2024,
  Clippy clean and no unwrap in library code. No parser framework is selected.
- Ask before adding dependencies, API breaks or new architectural patterns.
- Assess migrations against their callers, old failure modes and byte/diagnostic
  contracts. This foundation performs none.
- Never publish restricted media, snapshots, runtime-derived binary programs or
  private paths. Corpus source here is plain UTF-8 text, not native .AMOS files.

The root licence is GPL-2.0-or-later, following the sibling tool workspaces.
The maintained teaching snippets under corpus/ retain their upstream MIT licence
and notice. Future third-party source reuse needs a component-level rights check.
