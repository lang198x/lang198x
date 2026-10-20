# First AMOS parser milestone

**Status: proposal for design review.** The foundation contains comparison
fixtures and documentation, not parser or language-server code. This proposal
must be agreed before implementation or dependencies are added.

## Outcome

Given a small plain-text AMOS program and an explicit profile, return source
structure and located diagnostics without running it. Begin with the opening
message surface: `Cls`, string-literal `Print`, `Wait Key` and `End`. The two
maintained listings are examples, not exact strings to recognise.

Support each adopted form generally. Preserve arbitrary literal content and
source locations; accept valid variations established by the comparison cases.
Identify known but unsupported forms separately from invalid input. A restricted
first surface must not pretend to implement every form of `Print` or all AMOS.

The project investigates Creator, Easy AMOS, Pro and later releases from the
start. Only the recorded Pro 2.00 installation currently has observations in this
corpus. No implicit “latest Pro” profile or transfer to another release is allowed.

## Design options

| Approach | Benefit | Cost |
| --- | --- | --- |
| **Small AMOS-specific lexer/parser, no added dependency — recommended** | Direct control of source preservation, multiword forms and diagnostics; a narrow implementation can be inspected against the native cases. | We must implement recovery and grammar carefully. Keep functions and tests bounded; avoid growing a parser framework in disguise. |
| An existing lossless parser framework | May provide reusable tree/recovery machinery when editor workloads justify it. | Adds a dependency and its tree model before we have demonstrated the need. Evaluate licensing, maintenance and fit before selecting one. |

Do not adapt Spectrum grammar into AMOS grammar. Its existing AST/parser is a
migration candidate in a different dialect, with different source/byte contracts.
Source-location or diagnostic facilities can be shared when two actual consumers
show the need.

## First source contract to settle

1. Plain-text input is distinct from a native tokenised `.AMOS` file. Detect or
   require the representation and return an appropriate format diagnostic.
2. Preserve raw text and byte ranges alongside nodes. Define line/column mapping,
   tabs, line endings and encoding errors explicitly. The current UTF-8 fixtures
   do not establish all historical AMOS character behaviour.
3. Keep syntax, name/command resolution, types and version/extension availability
   separate. AMOS F2 performs more than parsing, so its verdict is not a pure
   grammar oracle. An unknown name may be a procedure call; do not reject every
   unrecognised statement head as a misspelt builtin at the lexical stage.
4. Return supported, unsupported and unknown compatibility honestly. No AST or
   diagnostic package format becomes a public compatibility promise in this step.
5. Preserve source spans through recovery. Never report whole-program success
   after skipping an unrecognised or unfinished construct.

## Comparison work before coding

The [initial corpus](../../corpus/amos/README.md) supplies two complete source
bindings and a reconstructed typo case. Extend the original-runtime observations
before deciding the lexical and command forms:

- Case, leading/trailing spaces, blank lines and alternate line endings.
- Different string contents, including punctuation that resembles separators.
- Colon separators versus labels, comments and quoting rules for each selected
  release. These cases are research requirements, not yet accepted forms.
- Missing/extra arguments, unfinished quotes, unknown names and trailing text.
- A valid form outside the first supported surface, so unsupported and invalid
  results cannot collapse into one verdict.
- At least one Creator and one installed Easy baseline, plus Pro release and
  extension differences. Keep unavailable cases visible rather than treating a
  shared keyword as sufficient compatibility evidence.

Record each exact source, profile/component identity, action, verdict and evidence
binding. Require deliberate rejected cases and a repair to an accepted result.
Use original manuals/source to explain discrepancies, then execute the named
original environment where available. Do not modify the language to make the
first examples pass.

## Implementation acceptance gates

- The selected source forms parse generally with exact source ranges and no
  hard-coded fixture recognition.
- Independent valid, boundary and malformed cases cover every supported form.
- Native acceptance/static checks and host parsing have distinct expectations.
- Unknown release/extension profiles and unsupported source forms cannot pass as
  complete compatibility. Tokenised input cannot pass as text.
- The typed/edit/repair examples and first additional comparison cases retain
  immutable evidence; the parser's green result is not their native evidence.
- The library result can later feed CLI and LSP adapters without duplicating
  grammar or analysis. No protocol service is advertised in this milestone.
- The first agreed implementation uses Rust 2024, is Clippy clean and adds no
  dependency without approval. Public crate names begin `lang198x-`.

A first parser is useful even while broader release qualification continues.
The staged support claim must name the tested profile/forms; it cannot be
marketed as complete AMOS coverage. No interpreter, compiler, LSP endpoint,
shared IR or existing-code migration is required to complete this milestone.

## Follow-on work

Grow through the actual Bank or Risk constructs, adding independent cases before
claiming support. Assess the Spectrum AST/parser and BASIC diagnostic extraction
on their own consumer/API evidence. Propose the first LSP capability only after
canonical language services can supply it. Blitz then gets its own version,
library and grammar investigation rather than an AMOS-shaped compatibility mode.
