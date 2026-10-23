# One syntax layer for every Lang198x language

**Status:** Adopted; implementation not started.
**Decided:** 23 October 2026.

## Goals

Three goals shape every choice in this record:

1. **Best in class.** The tools should be as good as any modern language tool,
   not just good enough for retro work.
2. **Broad compatibility, ideally full.** Each dialect should accept what the
   original tool accepted, reject what it rejected, and produce the same result.
3. **Idiomatic, well-structured Rust with clear documentation.** Code should be
   easy to read and change. Documentation uses plain English, at about
   Flesch-Kincaid grade 10.

## The decision

Lang198x builds one shared syntax layer, the crate `lang198x-syntax`. Every
language front end uses it, whether the language is an assembler or a BASIC.

1. **The crate holds shared building blocks only.** These are the lossless
   tree, source text, tokens, provenance, diagnostics and test helpers. There is
   no shared grammar, no shared AST and no shared semantics. Each language owns
   its own.
2. **We write the tree ourselves.** The crate has no runtime dependencies.
3. **One hand-written parser per family of dialects.** A family is a group of
   dialects that share their structure. A *dialect profile* records, as data,
   how each member differs.
4. **Provenance lives in side tables, not in the tree.** Every token can be
   traced back to the file, macro, include or native file it came from.
5. **The formatter only changes what a profile marks as safe.** The default is
   to change nothing. A dialect's formatter ships only after the original tool
   confirms that formatting changed nothing.
6. **Line-numbered BASICs get a program-order layer** above the tree. The tree
   always records the page exactly as written.
7. **Original runtimes are the oracle.** Assemblers already have a reference
   verdict corpus. BASICs gain one recorded from the original tokeniser running
   in Emu198x.

This record builds on the family's decision of 21 October 2026, which moved
assembly front ends to Lang198x. That decision left the choice of tree library
to stage 2 of its migration, and this record makes that choice. It also covers
the BASICs. The first AMOS kit avoided a shared framework until a second
language needed one. The Pasmo front end is that second language.

## The pipeline

```text
native file ──codec──▶ UTF-8 text + offset map          (Format198x)
text ──lexer, driven by the parser──▶ source tree       (per file, as written)
source tree ──typed views──▶ lowering ──▶ asm198x-ir    (assemblers)
                                      └─▶ semantic model (each BASIC)
source tree ──▶ formatter
source tree + typed views + semantics ──▶ language services (CLI and LSP)
```

During lowering, the front end may ask the syntax layer to expand a macro or
include. Expansion produces new text. The syntax layer parses that text into an
*expansion tree* and records where each part came from.

The formatter and the language services work on source trees. The assembler
works on the IR. Expansion trees are never formatted, because they are not
source.

## Why we write our own tree

A survey on 23 October 2026 compared `rowan`, `cstree`, `biome_rowan`,
`syntree`, Typst's syntax crate, tree-sitter and a hand-written tree.

- **`rowan`**, used by rust-analyzer, is the strongest library. It is also in
  the middle of a rewrite. Version 0.17 removed its editing API, and the 0.18
  alphas redesign how whitespace and comments attach to tokens. Its original
  author has written that bulk analysis is better served by plain arrays.
- **`cstree`** has a single maintainer. Its string interning would thread a
  resolver through every front end that reads text.
- **The others** either force their own parser framework, or are built for one
  host project.

Our needs are narrow. Trees are shallow, files are small, and rebuilding a whole
file is cheap. We do not need what `rowan` does best, which is cheap editing and
sharing of parts of a tree. A tree of our own costs roughly 1,000 to 1,500 lines.
In return, it lets us forbid `unsafe` code, keep the crate free of dependencies,
and keep the API to what we actually use.

**The cost:** we lose rust-analyzer's daily testing, and we own every bug. The
tests under *Testing* exist to pay that cost. Nothing outside the crate can see
how the tree is stored, so we can still switch to a library later without
touching any language.

## The tree

- **Storage.** The tree is one flat array of entries in source order. Each entry
  records its kind, its start and length in the text, its parent and the size of
  its subtree. The source text is stored once. A handle to a node is the tree
  plus an index, so handles are cheap to copy. Trees never change once built,
  so threads can share them freely.
- **Lossless.** Joining the text of every token in order gives back the input,
  byte for byte. Whitespace, comments and original spelling are all tokens in
  the tree.
- **Kinds.** Each language numbers its own node and token kinds. A small shared
  set covers whitespace, newlines, comments, errors and *verbatim* regions (see
  *The formatter*).
- **Lines.** Line-oriented languages give each line its own node. A comment then
  belongs to the line it sits on, which is what a formatter needs.
- **Text.** Trees hold UTF-8 text. Offsets are 32-bit, so a file can be up to
  4 GiB. Older character sets, such as PETSCII or the Amiga's, are decoded
  before parsing, and the decoder records an offset map back to the original
  bytes.
- **Building.** The parser calls `start_node`, `token` and `finish_node`. A
  checkpoint lets it wrap something it has already parsed in a new node, which
  is how a hand-written parser handles operator precedence. Misuse returns an
  error. It never panics.
- **Errors.** Parsing never fails. Broken source becomes `ERROR` nodes, and
  diagnostics are kept in a separate list keyed by text range. Diagnostics keep
  the family's distinction between *unsupported*, *invalid* and *unknown*.
- **Navigation.** The crate provides parents, children, siblings, descendants,
  text ranges, `token_at_offset` and `covering_element`. A new accessor is added
  only when a front end needs it.

## Lexing driven by the parser

Some dialects cannot be split into tokens without knowing what the parser
expects next:

- In vasm's Motorola syntax, whitespace after the operands starts a comment,
  unless the `-spaces` option is given. Knowing where the operands end means
  knowing the instruction.
- C64 BASIC accepts `FORI=1TO10`, with no spaces between keywords.
- Many assemblers treat a word in column 1 as a label and an indented word as an
  instruction.

So each lexer works on demand. The parser asks for the next token in a given
mode. Nothing has to split the whole file into tokens first.

## Dialect families and profiles

Many dialects differ only along a small set of axes:

- the characters that start a comment;
- whether column 1 means a label;
- whether whitespace inside or after the operands is a comment or an error;
- number syntax, and whether the way a number is written changes the encoding;
- whether symbols and keywords are case-sensitive;
- how macros substitute their arguments;
- limits of the original tool, such as line length or tab handling.

A **family** is a set of dialects that share one parser. A **profile** declares,
as data, where one dialect sits on each axis. Candidate families include:

- **Z80:** Pasmo, SjASMPlus and sjasm.
- **Motorola 68000:** Devpac, vasm's Motorola syntax, AsmOne, Seka and PhxAss.
- **6502:** ca65, DASM, xa and 64tass.

The verdict corpus proves membership. A dialect joins a family only when the
family's parser matches its verdicts.

Profiles also model the product and release, as the family rules require. A
profile names the original tool version it describes. ca65's CPU modes are one
more axis within the ca65 profile.

Where data is not enough, a profile can supply a small code hook. If a hook
keeps growing, that is evidence the dialect belongs to a different family.

The same profile drives both the parser and the formatter. One table states
what a dialect's text means and what can be changed safely.

## Macros, includes and conditionals

**Source trees record the page.** Each physical file gets its own tree, exactly
as written. Macro definitions, include directives and every branch of every
conditional are ordinary nodes. The syntax layer evaluates nothing.

**Front ends drive expansion.** Conditionals, repeat counts and token functions
often depend on evaluated values. Examples are `.if` on a label, `rept` with a
computed count, and forward references in multi-pass assemblers such as vasm and
asl. So expansion cannot be a pure stage that runs before parsing. The front end
expands macros and includes as it lowers. The syntax layer supplies the tools
for doing so.

**Expansion produces text.** Macro systems substitute arguments in two ways.
Some, like ca65 and SjASMPlus, substitute whole tokens. Others substitute
*inside* tokens. Devpac-style macros, as in vasm, write `label\@` for a unique
label or `move.\0` for a size given as an argument. A macro body of the second
kind cannot always be split into tokens until it is expanded. So expansion
always produces text, which is then lexed and parsed into an expansion tree.

**Provenance is tracked by character range.** Each expansion tree has a sorted
table. The table maps ranges of the expansion's text to where each range came
from. Origins include:

| Origin | Meaning |
|---|---|
| `Source` | Copied from a file, with its range. |
| `Included` | From an included file, with the include site. |
| `Expanded` | From a macro body, with the call site. |
| `Synthesised` | Made by a token function such as ca65's `.concat` or `.ident`, with every input and the call site. |
| `Decoded` | From a native or tokenised file, with its byte range. |
| `Merged` | From a BASIC `MERGE`, with the merged file. |

Expansions nest, so origins chain upwards until they reach real source. Lowering
carries the chain into `asm198x-ir` and each BASIC's semantic model. Diagnostics
and debug information then point at the text a person wrote.

## The formatter

**It never chooses line breaks.** In nearly every target language, a newline
ends a statement. So the formatter only adjusts spacing, indentation and column
alignment, plus keyword case where a profile allows it. It needs no layout
engine.

**It never adds, removes, reorders or renumbers lines.**

**It changes only what the profile marks as safe.** The default is to change
nothing. These are the known hazards:

- **Column 1.** Moving a word into or out of column 1 can turn an instruction
  into a label, or a label into an instruction.
- **Spaces in operands.** In vasm's Motorola syntax, adding a space after a
  comma turns the rest of the line into a comment.
- **Number spelling.** In ACME, `$00fc` forces 16-bit addressing where `$fc`
  would not. The formatter never rewrites how a number is written.
- **Symbol case.** Many assemblers treat `Loop` and `loop` as different labels.
  Case changes apply only to keywords that the profile declares case-insensitive.
- **Text-level macro arguments.** Some macro systems compare arguments as text,
  so a new space can change a conditional's outcome. Spacing inside macro
  arguments is left alone.
- **Original tool limits.** Padding must not push a line past the original
  tool's length limit.
- **Values that depend on position.** Line numbers stay fixed, because lines
  never move. A profile that exposes column positions marks them as unsafe.

**Verbatim regions are left exactly as written.** A macro body that substitutes
inside tokens is one example. Any region the lexer cannot be sure about is
another.

**Tokenised BASICs differ.** Where the machine decides spacing itself, as the
Spectrum does, formatting means reproducing what `LIST` shows. Where the machine
stores spaces as typed, as the C64 does, the usual whitespace rules apply. Each
profile says which kind it is.

**Four gates apply before a dialect's formatter ships:**

1. Formatting twice gives the same result as formatting once.
2. The sequence of tokens other than whitespace and comments is unchanged.
3. For assemblers, the original reference tool assembles every formatted corpus
   source to identical bytes and symbols. This catches meaning that our own
   lexer has missed.
4. For BASICs, the program order and line numbers are unchanged. For tokenised
   BASICs, tokenising the result gives the same stored program.

## Line-numbered BASICs

The 8-bit BASICs, BBC BASIC and STOS use line numbers. AMOS, GFA-BASIC and
Blitz are label-based.

- **Program order.** On the machine, a typed line replaces any line with the
  same number and takes its place by number. A text listing can therefore hold
  lines out of order, or the same number twice. The source tree keeps the page.
  The semantic model builds the program the machine would hold. Lines are
  sorted, a later duplicate replaces an earlier one, and each case gets a
  diagnostic.
- **References.** `GOTO`, `GOSUB`, `RESTORE`, `RUN` and `ON … GOTO` refer to
  line numbers, so line numbers work like labels. Go to definition and find
  references follow from that. A computed target, such as Spectrum BASIC's
  `GO TO n*10` or BBC BASIC's `GOTO (expression)`, is *unknown*. It is not an
  error.
- **Merging.** `MERGE` combines programs by line number, not by position. It is
  a program-level operation in the semantic model, with a `Merged` origin.
- **Hidden values.** The Spectrum stores each number in a line twice: as the
  digits shown and as a hidden five-byte binary value. The two can disagree. The
  decoder records the hidden value as provenance and flags any mismatch, so
  converting back to a native file loses nothing.
- **Renumbering** is a separate refactoring, not formatting. It updates every
  literal target. If a computed target might be affected, it refuses or asks
  for confirmation.

## Original-runtime oracles

The assembler verdict corpus already records what each reference tool accepts,
rejects and produces. BASICs gain the same kind of evidence. The original
tokeniser runs in Emu198x, and we record what it stores and what `LIST` shows
back. Those observations become expectations, written down before a surface is
claimed. ROMs, snapshots and runtime-derived binaries are never published.

## Typed views

Each language writes thin typed views over the tree. Examples are `Instruction`,
`MacroDef` and `Operand`. They are written by hand. A generator is considered
only once three languages show the same repetitive pattern. It would then use
`ungrammar` as a build-time-only dependency, which needs approval at the time.

## Testing

- **Round trip.** Every source in every corpus must come back byte for byte
  from its tree.
- **Fuzzing.** Random builder calls must produce a valid tree or a clean error.
  Random text must never panic a parser and must always round-trip. This uses
  the same tools already approved for AMOS: `proptest` for generated tests, and
  `libfuzzer-sys` in a separate fuzz workspace run with `cargo-fuzz`.
- **Snapshots.** A small checker of our own, not `insta`. It compares a tree dump
  with an expected `.tree` file, and setting `LANG198X_BLESS=1` rewrites the
  expected files. The dump shows one element per line, indented by depth, with
  its kind, range and, for tokens, its quoted text. Every language uses the same
  format.
- **Performance.** Benchmarks track tree building as a share of total corpus
  time. If it passes about 10%, investigate before optimising.

## Rust and documentation standards

- Rust edition 2024. Clippy clean, with warnings treated as errors.
  `unsafe_code = "forbid"`.
- No runtime dependencies in `lang198x-syntax`.
- No input can cause a panic. Library code uses no `unwrap`, and errors are
  values.
- The public API uses only the crate's own types. Public enums are
  `#[non_exhaustive]`, so variants can be added without a breaking change.
- Small modules with one job each. Language crates keep their own conventions,
  such as AMOS's one instruction per module.
- Every public item is documented, and `missing_docs` is denied. Examples in
  documentation run as tests.
- Documentation uses plain English, at about Flesch-Kincaid grade 10. It
  explains why, not only what.

## Effect on the migration

Stage 2 of the assembly move becomes: build `lang198x-syntax`, plus the Z80
family parser with a Pasmo profile. The gate is unchanged. The new front end
must match the existing front end's verdicts, including recorded rejections.
The family is designed to take further profiles, but no other dialect moves in
stage 2.

AMOS moves onto the shared tree when it builds its formatter. Its existing
source-contract checks are the gate. Its typed AST can then be built from the
tree.

## What this is not

- Not a shared grammar, AST or semantics. Each language still owns those.
- Not a parser framework or a parser generator.
- Not incremental parsing. Front ends reparse whole files until measurements show
  a need.
- Not a file codec. Native and tokenised formats stay with Format198x.
- Not a commitment to support every language or dialect now.
- Not permission to move dialects early. They move as work reaches them.

## Drift triggers

Stop and reread this record if anyone proposes to:

- add a runtime dependency to `lang198x-syntax`;
- let code outside the crate see how the tree is stored;
- store provenance in the tree;
- have the syntax layer evaluate conditionals or expand macros on its own;
- let the formatter change line breaks, line order, line numbers or token text
  beyond what a profile allows;
- ship a dialect's formatter before it passes its gates;
- write a separate parser for a dialect that fits an existing family;
- grow a profile hook into a parser of its own;
- add code generation for typed views before three languages need it;
- publish documentation well above grade 10.

## Related records

- The family decision of 21 October 2026, moving the assembly source layer to
  Lang198x. This record settles its stage 2 library choice.
- The family's Lang198x language-tools decision of 20 October 2026.
- AMOS: [first language kit](https://github.com/lang198x/amos/blob/main/decisions/first-language-kit.md)
  and [Perfect 10 parser slice](https://github.com/lang198x/amos/blob/main/decisions/perfect-ten-parser-slice.md).
- Asm198x: [macro expansion framework](https://github.com/asm198x/asm198x/blob/main/decisions/macro-expansion-framework.md)
  and [reference parity goal](https://github.com/asm198x/asm198x/blob/main/decisions/reference-parity-goal.md).

## Log

- **23 October 2026.** Recorded. Steve chose to write our own tree after the
  library survey. He asked for the design to cover assembly dialects not yet
  implemented, AMOS, Blitz, STOS, GFA-BASIC and other vintage languages, plus a
  formatter and include and macro handling. A review of blue-sky alternatives
  added dialect families with profiles and original-runtime oracles for BASICs.
  Steve set the three goals at the top of this record.
