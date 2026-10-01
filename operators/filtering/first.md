# `first` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/first.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `first(predicate?, defaultValue?): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Emit the first source value that matches `predicate` (default: all values) and complete. If the source completes with no match, emit `defaultValue` if given, else error `EmptyError`. Unsubscribes after the match.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { searching(i), stopped }`.

## 2. Initial state (S0)

`searching(0)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Non-matching next → `searching(i+1)`.
- Matching next → `stopped`.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Matching next → `next(v) · complete`.
- Non-matching → `ε`.
- Complete with no match → `next(default) · complete` or `error(EmptyError)`.

## Worked trace

`first(x => x > 2)` on `1, 2, 3, 4` writes `next(3) complete` and does not see 4.

## Why this is Mealy rather than Moore

Match decision uses predicate on the input; default-or-error uses the complete input against searching state.

## Edge cases fixed by the 7.x source

- No predicate means the first next wins.
- EmptyError is the no-default path.

## Source anchors

- `src/internal/operators/first.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
