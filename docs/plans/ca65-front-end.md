# Plan: the ca65 front end (stage 3)

**Status:** Proposed 2026-10-26, for agreement before any code.
**Binding records:** this repository's
[shared syntax layer decision](../../decisions/shared-syntax-layer.md) and
198x's
[assembly source layer decision](https://github.com/stevehill1981/198x/blob/main/decisions/assembly-source-layer-moves-to-lang198x.md),
stage 3.

Build `lang198x/ca65` for the CPUs Asm198x's main ca65 reader covers, and
move that reader onto it under the gate the PasmoNext profile passed: the new
path matches the reference tool's recorded verdicts, including rejections,
before it becomes the default. Then delete the old path. Stage 4 (65816 and
HuC6280 as modes) and stage 5 (45GS02, M740, SWEET16) follow under their own
plans.

## Settled before starting

- **The profile is ca65 V2.19 at cc65 commit `8d86ca7`.** Asm198x's
  `ops/arbiter/identities` pins it (cc65 has tagged no release since 2.19),
  and its version line reads `ca65 V2.19 - Git 8d86ca7`.
- **One family, one profile.** The shared syntax layer decision makes ca65 a
  family of one, with its CPU modes as an axis of that profile. Stage 3 builds
  the 6502 mode; stage 4 adds the others.
- **The gate runs in Asm198x,** behind a switch, with the corpus there and a
  pinned pair of versions, as for PasmoNext.
- **Linking stays in Asm198x.** ca65's output reaches byte identity only after
  ld65 links it, so the conformance contract covers the linked result. The
  front end lowers to `asm198x-ir`; Asm198x keeps segments, layout and its
  bounded ld65.
- **The reference decides,** not the old reader. Where the two disagree, the
  new front end follows ca65, or fixes an obvious ca65 bug under 198x's
  `reference-bugs-are-fixed-not-copied.md`.
- **The freeze holds** (asm198x#940): the old readers get no parity fixes
  while this runs, except correctness bugs that affect learners today.

## What to match

Asm198x's ca65 code is about 16,900 lines: the main reader
(`dialects/ca65/`, 10,400 lines with its NES link and project support), the
shared flat walk (`ca65_flat.rs`, 3,000), and the 65816 and HuC6280 readers
(2,700 together, stage 4). Its ca65-arbitrated verdicts are thin for the
main reader: 51 NES curriculum programs, and no 6502 form, fuzz or probe
verdicts, because ACME arbitrates the 6502 forms. The 65816 and HuC6280
readers have 504 form verdicts and 65 probes between them.

## Steps

Each step is one pull request that passes format, Clippy, tests and the
coverage floor.

### 1. Verdicts first (Asm198x)

- Run the 6502 form sweep and the differential fuzz through ca65 as well as
  ACME, as asm198x#985 and #987 did for SjASMPlus.
- Triage asm198x#940's open ca65 issues: each is either a learner-facing
  correctness bug, fixed in the old reader, or a parity case, recorded as a
  probe with ca65's verdict and closed with a link.
- Probes for each lexical and preprocessing axis, measured before the lexer
  is written.

### 2. The repository

`lang198x/ca65`, with the standard skeleton, crate `lang198x-ca65`, the
optional `ir` feature as in `z80-asm`, and a manual first crates.io publish
before Trusted Publishing.

### 3. Lexer and profile axes

ca65's tokens, its number and string forms, `@cheap` and unnamed labels,
and the CPU mode as an axis, each a unit test of a measured case.

### 4. Parser

One `LINE` per source line, ca65's control commands, and its expression
grammar with `<`, `>` and `^` binding tight.

### 5. Preprocessing

`.macro` and `.define`, `.repeat`, the `.if` family, includes, and the token
functions (`.concat`, `.ident`, `.left`, `.mid`, `.right` and the rest).
Tokens they make carry `Synthesised` provenance. This is the stage's hard
part, and why ca65 moves before the simpler dialects.

### 6. Lowering and the walk (Asm198x)

A walk for ca65 beside PasmoNext's: scopes and `.proc`, segments, imports and
exports, feeding Asm198x's existing NES link.

### 7. The switch

Make the walk the default for the main reader, move `fmt`, then delete the
old path, as for PasmoNext.

## Questions to settle

1. Does this order work, with verdicts and the #940 triage before the
   repository exists?
2. `lang198x/ca65` has to be created, and its crate published once by hand.
   Who does that, and when?
