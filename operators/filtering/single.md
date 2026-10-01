# `single` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/single.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `single(predicate?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Expect exactly one matching value. Remember it. A second match errors with `SequenceError`. On complete, emit the one match. No match or complete-without-match errors `EmptyError` unless the implementation's empty path applies. Unsubscribes on the second match.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { none, one(v), stopped }`.

## 2. Initial state (S0)

`none`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `none × matching next → one(v)`.
- `one × matching next → stopped` (error).
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Matching next in `none → ε`.
- Matching next in `one → error(SequenceError)`.
- Complete in `one → next(v) · complete`.
- Complete in `none → error(EmptyError)`.

## Worked trace

`of(2).pipe(single())` writes `next(2) complete`. `of(2, 4).pipe(single())` writes `error(SequenceError)`.

## Why this is Mealy rather than Moore

The second match is an error only because state is already `one`. Complete emits only from `one`.

## Edge cases fixed by the 7.x source

- Predicate narrows what counts as a match.
- Non-matching values are ignored.

## Source anchors

- `src/internal/operators/single.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
