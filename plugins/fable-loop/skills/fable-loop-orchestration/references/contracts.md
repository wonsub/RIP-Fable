# Contracts — distrust by default (D2)

Trust nothing by default: not external input, not model output, not your own
future self. Validate at every boundary and re-derive anything you did not
compute yourself. The model proposes; the engine decides.

## The four reflexes

1. **Re-derive untrusted values.** Discard a value handed to you and rebuild it
   from a source you control. Canonical example: throw away a model-supplied date
   and reconstruct it from the engine/system clock.
2. **Clamp to the legal range.** Coerce every value into its contract before use —
   e.g. clamp a score to 1–5, an index into bounds, an enum to its allowed set.
   Out-of-range is contained at the boundary, never propagated inward.
3. **Make the boundary explicit.** Every function crossing a trust boundary states
   its precondition and what it does on violation (reject / clamp / default).
4. **Future-self is untrusted too.** Leave the invariant as an assertion in the
   code, not in your memory — the next caller (including you) will forget it.

## The four-value boundary test (required gate)

Every boundary-crossing unit ships with tests for all four cases:

| Case     | Input                               | Expected behavior                       |
|----------|-------------------------------------|-----------------------------------------|
| normal   | a valid, in-range value             | passes through, correct result          |
| mapped   | a value needing transform/clamp     | coerced to the legal value              |
| None     | missing / null / empty              | safe default or explicit rejection      |
| tampered | malformed / out-of-range / hostile  | rejected or clamped, never trusted      |

A unit that crosses a trust boundary without all four cases tested does not meet
the definition of done.
