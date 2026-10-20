# First AMOS comparison cases

This is a small research corpus, **not parser test results**. There is no parser
implementation in this repository. `cases.json` records earlier original-runtime
observations; it is not a frozen public package or interchange schema.

| Case | Source binding | Retained original observation |
| --- | --- | --- |
| `message` | Byte-identical to the maintained five-line message at the pinned samples revision. | AMOS Professional 2.00 accepted F2 Test after typed entry. |
| `edited-message` | Byte-identical to the maintained changed question. | F2 Test accepted the edit; separate retained evidence covers native Save and independent cold Load. |
| `misspelt-command` | Reconstructed from the recorded replacement of the title's `Print` with `Prit` in the changed program. It is not an exported native source file. | F2 Test rejected the recorded typo. Repairing it to `Print` was then accepted. |

## Sources and evidence

The maintained source revision is
[code-samples `633cd580`](https://github.com/code198x/code-samples/tree/633cd580/commodore-amiga/amos/bank-or-risk/stages/opening).
The [recorded native route](https://github.com/code198x/docs/tree/d9c8a1005851adfec3314e68070378f466561fe5/audit/2026-10-amos-first-lesson-pair)
contains requests, captures and observations. The `wrong-command` request records
the exact `Prit` line replacement. The reconstructed fixture preserves the rest
of the maintained edited source; it has not been rerun as an independently
imported whole file. Keep that limit when using it as a future reproduction case.

`cases.json` binds each plain-text file by SHA-256. The suffix `.amos.txt`
explicitly distinguishes these UTF-8 listings from native tokenised `.AMOS`
programs. [The upstream MIT licence and notice](../LICENSE) apply to these
maintained/derived teaching snippets.

## Profile limits

[Pro 2.00](profiles/pro-2.00.json) identifies the specific previously exercised
installation and emulator source. These observations do not qualify every build
with the same version label. Neither all bundled extensions nor third-party
extensions have been qualified by these message cases.

[Pending product lines](profiles/pending.json) keep Creator and Easy AMOS visible.
Their results, other Pro releases and later derivatives remain unknown here.
The earlier [reference investigation](https://github.com/code198x/docs/blob/d9c8a1005851adfec3314e68070378f466561fe5/platforms/commodore-amiga/amos/reference-profiles.md)
separates candidate media from working installations. Do not manufacture passing
cells for missing environments.

## Interpreting a result

AMOS F2 checks more than pure grammar. In particular, an unrecognised word might
be a procedure name requiring resolution rather than a lexical error. The typo
case's native rejection is an acceptance observation; a future parser can model
its source structure while a separate check rejects an unresolved invocation.
The host parser must not mirror the F2 verdict as its syntax verdict.

Future cases should identify source, product/release, components/extensions,
representation and native action precisely. Keep original outputs/errors and
rejected controls. Corpus/hash validation proves the records agree with their
inputs; it does not execute AMOS, prove a parser correct or measure learning.
