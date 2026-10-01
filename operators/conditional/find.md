# `find` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/find.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `find(predicate, thisArg?): OperatorFunction<T, T | undefined>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Emit the first matching value and complete. If the source completes with no match, emit `undefined` and complete. Does not error on a miss. Unsubscribes after a match.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { searching(i), stopped }`.

## 2. Initial state (S0)

`searching(0)`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T | undefined), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Non-match increments i.
- Match → `stopped`.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Match → `next(v) · complete`.
- Non-match → `ε`.
- Complete → `next(undefined) · complete`.

## Worked trace

`find(x => x > 10)` on `1 2 3` writes `next(undefined) complete`.

## Why this is Mealy rather than Moore

Complete input writes undefined because state is still searching. A matching next writes the value instead.

## Edge cases fixed by the 7.x source

- Unlike `first`, a miss is `undefined`, not `EmptyError`.
- Index is passed to the predicate.

## Source anchors

- `src/internal/operators/find.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
