# Settings that change how source is read

**Status:** Proposed 2026-11-03.
**Builds on:** [one syntax layer for every Lang198x language](shared-syntax-layer.md),
in particular *Parsing that needs context*, *Dialect families and profiles*
and the rule that older character sets are decoded before parsing.

## The problem

Many assemblers have commands that change how the lines after them are read.
Lang198x's families meet them one at a time, and each could solve them its
own way. This record sets one way for every family.

Measured here, or met by Asm198x's readers for these tools:

| Tool | Commands | What they change | Source |
|---|---|---|---|
| ca65 V2.19 | `.case`, `.localchar`, `.linecont`, `.feature` (19 features) | name case, the cheap-local character, joined lines, comments, number and string syntax, labels without colons, and more | measured |
| SjASMPlus 1.24.0 | `OPT --syntax=…`, `OPT push`, `OPT pop`, `OPT reset` | brackets as memory or immediate, multi-argument lines, warnings | measured |
| Macro Assembler AS | `RELAXED` | number syntax: `012` is twelve before it and ten after | probed by Asm198x |
| lwasm | `PRAGMA`, `OPT` | 49 switches, such as whether 6309 instructions are allowed | probed by Asm198x |
| rgbasm | `OPT`, `PUSHO`, `POPO` | options, with a stack | refused by Asm198x as not yet built |
| vasm | `opt` | options | listed by Asm198x |

Two measurements show that tools differ where it matters:

- **A macro body.** ca65 keeps a body in the tokens it was read as, so the
  body reads as it did where it was defined. SjASMPlus reads a body again at
  each call, under the settings in force there. With SjASMPlus,
  `OPT --syntax=B` before a call changes how the body's `ld a,(1)` is read,
  even though the body was defined before it.
- **A stack.** SjASMPlus and rgbasm keep a stack of settings that a command can
  push and pop. ca65 has a stack only for the CPU.

Both tools read source as bytes, so a byte from `$80` to `$FF` in a string is
that byte. ca65 refuses such a byte outside a string or comment.

## The decision

**A family's reading settings are an axis of its profile, held in its mode.**
The profile declares each setting a dialect has and its starting value, as
data. The mode that the lexer and parser read under carries the current
values. A family already passes a mode to its lexer for the CPU, so this
widens it. It does not add a second path.

**The rules every family follows:**

1. **A setting acts from the line after its command.** Nothing read before it
   changes.
2. **A line read under an older mode is read again.** A family parses a file
   ahead. When a command changes a setting, a later line that was parsed under
   the old mode is lexed and parsed again before it is used. The tokens of a
   single line are enough for this, except for rule 3.
3. **A setting that joins lines reads the rest of the text again.** Line
   continuations and C-style comments decide where lines end. When one
   changes, the family parses the rest of the current text again, from the
   next line, under the new mode. Earlier lines are left as they are, and
   provenance still traces every line to its file.
4. **A macro body follows the dialect.** Whether a body reads as at its
   definition or as at each call is a profile axis, set from measurements of
   the tool. A body kept as tokens follows the definition. A body read again
   at each call follows the call.
5. **Settings outlive an include and ignore a skipped branch,** unless a
   tool's measurements show otherwise.
6. **A stack is the dialect's.** Push, pop and reset exist where the tool has
   them, with the tool's own limits and errors.
7. **A setting that changes meaning, not reading, reaches the host.** Some
   settings, such as ca65's `force_range` or `org_per_seg`, change what a line
   does rather than how it is read. They travel with each line to lowering,
   like the CPU, and the assembler acts on them.
8. **Every setting has measured lines before it is built,** including the
   cases where the tool refuses it.

**Source arrives as bytes, and a shared decoder turns it into text.**
`lang198x-syntax` gains the decoder that the syntax-layer record calls for:
bytes in, UTF-8 text and an offset map out, reversible in every case. Its
first character set is Latin-1, in which every byte is the character of the
same number. That gives ca65's and SjASMPlus's bytes exactly. A front end
keeps a text entry point for tools that already have text, such as an
editor. Each family's profile names the character set its tool reads.

## What this is not

- **Not a settings framework in the syntax layer.** Reading settings mean
  something only to the family that owns them. `lang198x-syntax` gains the
  decoder and nothing else from this record.
- **Not one stack shape for every tool.** Rule 6 leaves each dialect's stack
  to its own measurements.
- **Not a promise to build every feature of every tool.** A setting is built
  when its dialect needs it, under these rules.

## Drift triggers

Stop and reread this record if anyone proposes to:

- give a family a second way to pass reading settings to its lexer;
- read a line under a mode other than the one in force where it is read;
- make a macro body follow the definition or the call without measuring the
  tool;
- decode source in a way that cannot round-trip, or outside the shared
  decoder;
- add a generic settings type to `lang198x-syntax`.

## Related records

- [One syntax layer for every Lang198x language](shared-syntax-layer.md).
- lang198x/ca65's plan for its settings, which follows this record.
- lang198x/z80-asm's SjASMPlus profile plan, whose label-case axis is one of
  these settings.

## Log

- **3 November 2026.** Proposed. While planning ca65's `.case`, `.localchar`,
  `.linecont` and `.feature`, Steve asked whether the design belonged above
  ca65, since other assemblers surely do the same. A survey found reading
  settings in SjASMPlus, rgbasm, lwasm, vasm and Macro Assembler AS.
  Measurement showed that ca65 and SjASMPlus differ on macro bodies.
