# Plan: ca65's other CPUs (stage 4)

**Status:** Proposed 2026-10-31. Step 1 is under way.
**Binding records:** this repository's
[shared syntax layer decision](../../decisions/shared-syntax-layer.md) and
198x's
[assembly source layer decision](https://github.com/stevehill1981/198x/blob/main/decisions/assembly-source-layer-moves-to-lang198x.md),
stage 4.

Stage 3 ([`ca65-front-end.md`](ca65-front-end.md)) moved Asm198x's main ca65
reader onto `lang198x-ca65` for the 6502 and the 4510, and the old reader's
assembler and formatter are gone. This stage makes the 65816 and the HuC6280
modes of the same front end, as the decision asks, and moves Asm198x's
`65816` and `huc6280` dialects onto it. Then `ca65_flat.rs`, `ca65_816.rs`,
`ca65_huc6280.rs` and what is left of the old main reader are deleted.

The other ca65 CPUs whose instruction sets Isa198x already has come in the
same stage, because each is mostly data: the 6502X, 6502DTV, 65SC02, 65C02,
W65C02 and 65CE02. Stage 5 (the 45GS02, M740 and SWEET16) needs Isa198x
first and has its own plan.

## Settled before starting

- **The CPU is one axis of the one profile.** `Cpu` gains every ca65 CPU,
  chosen by `.setcpu`, the `.P*` commands, `.pushcpu` and `.popcpu`, and
  asked about by the `.IFP*` conditionals. Each brings ca65's mnemonic table
  and registers, from ca65 V2.19's `instr.c` and `scanner.c`. A CPU that
  lowering does not handle yet is refused as "not lowered yet", as now.
- **The syntax that really differs plugs into the one parser:** the 65816's
  long and indirect-long forms (`[dp]`, `[dp],y`, `f:`, `al,x`) and block
  moves, and the HuC6280's block transfers. Everything else is the 6502's
  operand grammar.
- **The 65816's widths are expander state,** like the CPU: `.a8`, `.a16`,
  `.i8`, `.i16` and `.smart`, which `rep` and `sep` change under `.smart`.
  Lowering reads them to size an immediate.
- **Lowering looks instructions up in Isa198x,** as the 4510's does: a base
  set and the extension the CPU adds. Where ca65 spells an instruction its
  own way, lowering maps the spelling; where ca65's table disagrees with
  Isa198x's, ca65 decides for the `ca65` profile and the difference is
  recorded.
- **Support for a CPU is claimed only once it has its own verdict section**
  in Asm198x's corpus, as the decision says. The 65816 has 340 ca65 verdicts
  and the HuC6280 234 today; the others start with none.
- **The reference decides,** with obvious ca65 bugs fixed rather than copied
  and named as divergences, as in stage 3.

## The gates

- cc65's own opcode test for each CPU (`test/asm/opcodes/<cpu>-opcodes.s`
  and its `.ref`), measured on V2.19 under `.org 0` as for the 4510, lowers
  to ca65's bytes. cc65 has one for every CPU here but the 65816.
- Every recorded ca65 verdict for the CPU replays through the walk.
- For the 65816 and HuC6280, the flat readers' own tests pass through the
  walk, as the main reader's did in stage 3, before the switch.

## Steps

Each step is a `lang198x-ca65` pull request and release, then the Asm198x
pull request that takes it, each passing format, Clippy, tests and the
coverage floor.

### 1. Every CPU on the axis

`Cpu` gets all thirteen of ca65's CPUs, with their names for `.setcpu`, their
`.P*` command, their `.IFP*` conditional, their registers and their mnemonic
tables. The expander follows them all. Lowering still lowers only the 6502
and 4510; the others are refused as now, but by name. Tests: each command and
conditional measured on V2.19, and each table checked against `instr.c`.

### 2. The 6502's relatives

The 6502X, 6502DTV, 65SC02, 65C02, W65C02 and 65CE02, one or two per pull
request, each against cc65's opcode test. Asm198x records a verdict section
for each from the form sweep and fuzz it already runs for the 6502, arbitrated
by ca65, and the walk gives each statement its CPU's instruction sets.

### 3. The HuC6280

Its block transfers (`tii`, `tdd`, `tin`, `tia`, `tai`) take three operands,
and `st0`, `st1`, `st2`, `tam` and `tma` their own. Gate: cc65's opcode test
and the 234 HuC6280 verdicts. Then Asm198x's `huc6280` dialect becomes the
walk with the HuC6280 as its starting CPU, and the flat reader's tests pass
through it.

### 4. The 65816

Widths as expander state, `.smart`, the long and indirect-long forms, block
moves (`mvn`, `mvp`), `f:` and 24-bit addresses, and `.a8`-style sizing of
immediates. Gate: the 340 65816 verdicts and the flat reader's tests. Then
Asm198x's `65816` dialect becomes the walk with the 65816 as its starting
CPU.

### 5. Delete the old readers

`ca65_flat.rs`, `ca65_816.rs`, `ca65_huc6280.rs` and the rest of
`dialects/ca65/` go. Their formatters move to `lang198x_ca65::format`, under
the gate stage 3's formatter shipped under.

## What could go wrong

- **ca65 and Isa198x may disagree** on what a CPU has, as the decision warns
  for the 65C02's Rockwell set. ca65 decides for this profile; the difference
  is recorded in Isa198x rather than worked around in lowering.
- **The 65816's widths change sizes,** so an `.a16` that the flat reader got
  wrong changes every immediate after it. The verdicts catch this only where
  they cover it, so step 4 adds probes for each width command and for
  `.smart` with `rep` and `sep`.
