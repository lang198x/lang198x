# Lang198x

Language analysis and language services for vintage programming languages.
The first target is AMOS, including its product versions, release differences
and extensions. Blitz follows through its own investigation.

**Status: foundation and design.** There is no parser, language server,
interpreter or compiler implementation yet. This repository provides a reviewable
first milestone and three source fixtures with retained native observations.

## Start here

- [First AMOS parser milestone](docs/plans/amos-first-parser.md): proposed surface,
  implementation options and checks required before coding.
- [AMOS comparison corpus](corpus/amos/README.md): what each fixture establishes,
  its source provenance and the unqualified product lines.
- [Scope and boundaries](decisions/README.md): ownership and migration direction.

The intended tools can support listing checks, diagnostics, source navigation,
formatting and completion, with LSP adapters for editors. These are scope and
future capabilities, not features shipped by this foundation.

## Contributing

Useful first contributions are minimal source cases, original release notes and
extension documentation, and named-version observations that can be reproduced.
Record the exact source and installation, the action, the result and its limits.
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
The teaching-source fixtures retain their [upstream MIT licence](corpus/LICENSE).
No original AMOS runtime, disk, ROM or native binary program is distributed.
