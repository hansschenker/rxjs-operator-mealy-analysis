# `scan` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable accumulator |
| RxJS 7.x source | `src/internal/operators/scan.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `scan(accumulator, seed?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Emit the running accumulation on every source next. With a seed, the first next calls `accumulator(seed, value, index)`. Without a seed, the first next is emitted as the initial acc and the accumulator starts at the second value. Complete and error pass through; the last acc is not re-emitted on complete.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { needSeed, holding(acc, i), stopped }`. `needSeed` only exists when no seed was given.

## 2. Initial state (S0)

`holding(seed, 0)` if seed given, else `needSeed`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(acc), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `needSeed × next(v) → holding(v, 1)`.
- `holding(acc, i) × next(v) → holding(accumulator(acc, v, i), i+1)` or `stopped` if it throws.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `needSeed × next(v) → next(v)`.
- `holding × next(v) → next(accumulator(...))`.
- Throw → `error`.
- `complete → complete`.

## Worked trace

`of(1, 2, 3).pipe(scan((a, b) => a + b, 0))` writes `next(1) next(3) next(6) complete`.

## Why this is Mealy rather than Moore

`G` applies the accumulator to state and input, and `T` stores that same result. This is the standard Mealy accumulator.

## Edge cases fixed by the 7.x source

- No seed plus empty source completes with no next.
- Index passed to the accumulator counts accumulator applications.

## Source anchors

- `src/internal/operators/scan.ts` and `scanInternals.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
