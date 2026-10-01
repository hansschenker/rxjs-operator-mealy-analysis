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

## Explanation

`scan` is a pipeable accumulator on the RxJS 7.x line. Stable. Emit the running accumulation on every source next. With a seed, the first next calls `accumulator(seed, value, index)`. Without a seed, the first next is emitted as the initial acc and the accumulator starts at the second value. Complete and error pass through; the last acc is not re-emitted on complete.

In plain terms, the operator keeps this memory: S = { needSeed, holding(acc, i), stopped }. needSeed only exists when no seed was given. At subscription, before any source notification, that memory is holding(seed, 0) if seed given, else needSeed. It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2, 3).pipe(scan((a, b) => a + b, 0)) writes next(1) next(3) next(6) complete.

Details that a marble diagram often leaves out: No seed plus empty source completes with no next. Index passed to the accumulator counts accumulator applications.

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

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { needSeed, holding(acc, i), stopped }. needSeed only exists when no seed was given.

Memory at subscribe: holding(seed, 0) if seed given, else needSeed.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| needSeed × next(v) | holding(v, 1) | next(v) |
| holding(acc, i) × next(v) | holding(accumulator(acc, v, i), i+1) or stopped if it throws | nothing named on a separate output row |
| Terminal | stopped | next(accumulator(...)) |
| Throw | named by the output row; memory change is in the transition rows above | error |
| complete | named by the output row; memory change is in the transition rows above | complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

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
