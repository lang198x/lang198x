# Lang198x

Language analysis and language services for vintage programming languages.
AMOS comes first, including its product versions, release differences and
extensions. Blitz follows through its own investigation.

**The first AMOS language kit lives in [amos](https://github.com/lang198x/amos).**
It provides a bounded Rust parser/checker and distributable TextMate highlighting
for Shiki and VS Code. Its coverage document distinguishes implemented source
forms from original-runtime compatibility. Native files, LSP and compilation
remain follow-on work.

This repository is the coordination and research hub. Language implementations
live in separate repositories, with language-specific ASTs and small interfaces
shared only when real consumers need them.

## Start here

- [AMOS implementation and packages](https://github.com/lang198x/amos).
- [AMOS coverage](https://github.com/lang198x/amos/blob/main/docs/coverage.md).
- [Canonical AMOS corpus](https://github.com/lang198x/amos/tree/main/corpus).
- [Scope and boundaries](decisions/README.md).

The [original milestone proposal](docs/plans/amos-first-parser.md) and
[foundation corpus](corpus/amos/README.md) remain as research snapshots so existing
links and evidence bindings continue to work. New AMOS cases and implementation
plans belong in the AMOS repository.

## Contributing

Useful contributions include minimal source cases, original release notes,
extension documentation and named-version observations that can be reproduced.
Record exact source and installation, the action, the result and its limits.
Do not infer compatibility across releases or count a parser accepting code as
evidence that the original runtime accepts or executes it.

Existing Spectrum parser/AST code and BASIC diagnostics are migration candidates
to assess. No migration is performed here, and AMOS need not wait for a broader
reorganisation. Shared internals must earn their place through real consumers.

## Family and licence

[Lang198x](https://github.com/lang198x) is an independent sibling of Code198x,
Asm198x, Build198x, Format198x and Emu198x. Forge198x can consume its services as
the future workbench; language servers can serve other editors too.

The project follows the [shared principles](PRINCIPLES.md). Authored project
material uses [GPL-2.0-or-later](LICENSE), following the sibling tool workspaces.
The retained teaching fixtures keep their [upstream MIT licence](corpus/LICENSE).
No original AMOS runtime, disk, ROM or native binary program is distributed.
