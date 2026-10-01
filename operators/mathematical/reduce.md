# `reduce` — Mealy 6-tuple

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/reduce.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `reduce(accumulator, seed?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`reduce` is a pipeable operator on the RxJS 7.x line. Stable. Fold the source. Emit the final accumulator on complete only. With a seed, empty source emits the seed. Without a seed, empty source errors `EmptyError`, and the first value becomes the initial acc without calling the accumulator.

In plain terms, the operator keeps this memory: S = { needSeed, holding(acc, i), stopped }. At subscription, before any source notification, that memory is holding(seed, 0) if seed given, else needSeed. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2, 3).pipe(reduce((a, b) => a + b, 0)) writes next(6) complete and nothing earlier. scan would have written the intermediates.

Details that a marble diagram often leaves out: Seed is emitted on empty complete. No seed plus one value emits that value on complete without calling the accumulator.

## Role in the notification machine

Fold the source. Emit the final accumulator on complete only. With a seed, empty source emits the seed. Without a seed, empty source errors `EmptyError`, and the first value becomes the initial acc without calling the accumulator.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { needSeed, holding(acc, i), stopped }`.

## 2. Initial state (S0)

`holding(seed, 0)` if seed given, else `needSeed`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(acc), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `needSeed × next(v) → holding(v, 1)`.
- `holding × next → holding(accumulator(acc, v, i), i+1)`.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete` in `holding → next(acc) · complete`.
- `complete` in `needSeed → error(EmptyError)`.
- Accumulator throw → `error`.

## Worked trace

`of(1, 2, 3).pipe(reduce((a, b) => a + b, 0))` writes `next(6) complete` and nothing earlier. `scan` would have written the intermediates.

## Why this is Mealy rather than Moore

Same accumulator transition as `scan`, but `G` writes `ε` on next and the acc word on complete. The difference between scan and reduce is entirely in `G`.

## Edge cases fixed by the 7.x source

- Seed is emitted on empty complete.
- No seed plus one value emits that value on complete without calling the accumulator.

## Source anchors

- `src/internal/operators/reduce.ts` uses `scanInternals` and emits on complete.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
