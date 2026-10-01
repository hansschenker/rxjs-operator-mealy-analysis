# `takeWhile` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/takeWhile.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `takeWhile(predicate, inclusive = false): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`takeWhile` is a pipeable operator on the RxJS 7.x line. Stable. Emit while `predicate(value, index)` is true. The first false value completes the output. If `inclusive`, that failing value is emitted before complete. Unsubscribes when the predicate fails.

In plain terms, the operator keeps this memory: S = { taking(i), stopped }. At subscription, before any source notification, that memory is taking(0). It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: takeWhile(x => x < 3) on 1 2 3 4 writes next(1) next(2) complete. Inclusive also writes next(3) before complete.

Details that a marble diagram often leaves out: Index increments per source next while taking. Inclusive flag is a parameter of `G`, not an extra state.

## Role in the notification machine

Emit while `predicate(value, index)` is true. The first false value completes the output. If `inclusive`, that failing value is emitted before complete. Unsubscribes when the predicate fails.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { taking(i), stopped }`.

## 2. Initial state (S0)

`taking(0)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Predicate true → `taking(i+1)`.
- Predicate false → `stopped`.
- Source complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Predicate true → `next(v)`.
- Predicate false → `complete`, or `next(v) · complete` if inclusive.

## Worked trace

`takeWhile(x => x < 3)` on `1 2 3 4` writes `next(1) next(2) complete`. Inclusive also writes `next(3)` before complete.

## Why this is Mealy rather than Moore

The failing input writes `complete` or `next·complete` according to the inclusive parameter and the predicate on that input.

## Edge cases fixed by the 7.x source

- Index increments per source next while taking.
- Inclusive flag is a parameter of `G`, not an extra state.

## Source anchors

- `src/internal/operators/takeWhile.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
