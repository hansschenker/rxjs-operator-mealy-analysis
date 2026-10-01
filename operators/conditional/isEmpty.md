# `isEmpty` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/isEmpty.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `isEmpty(): OperatorFunction<T, boolean>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

If the source emits any next, emit `false` and complete, unsubscribing. If the source completes with no next, emit `true` and complete.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(boolean), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next → stopped`.
- `complete → stopped`.

## 6. Output function (G : S × Z → A*)

- `next → next(false) · complete`.
- `complete → next(true) · complete`.
- `error → error`.

## Worked trace

`EMPTY.pipe(isEmpty())` writes `next(true) complete`. `of(1).pipe(isEmpty())` writes `next(false) complete`.

## Why this is Mealy rather than Moore

Next and complete are the two inputs that select false vs true from the empty state.

## Edge cases fixed by the 7.x source

- Does not wait for complete after the first value.
- Error is not converted to a boolean.

## Source anchors

- `src/internal/operators/isEmpty.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
