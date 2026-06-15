# Frozen Acceptance Gates

Written during the architect step before any building. Never edited mid-run to force a pass.

## Authoring rule (D2)

For any lane whose artifact crosses a trust boundary (parses external input, uses
model output, reads a clock, accepts a score/index), its gate MUST require the
four-value boundary test from `references/contracts.md` — normal / mapped / None /
tampered. See that doc for the contract.

(empty — populated per run)
