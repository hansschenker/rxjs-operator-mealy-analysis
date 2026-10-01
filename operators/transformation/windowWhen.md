# `windowWhen` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowWhen.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `windowWhen(closingSelector: () => Observable<any>): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Window analogue of `bufferWhen`. A window is open from subscribe. When the closing observable emits, complete the window, emit a new one, and call `closingSelector` again.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open window, stopped }`.

## 2. Initial state (S0)

First window emitted, closer subscribed.

## 3. Input alphabet (Z)

`{ next(v), error, complete, closeNext, closeError, unsubscribe }`.

## 4. Output alphabet (A)

Window observables and inner notifications.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `closeNext` swaps the open window.
- Source next keeps it.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `subscribe → next(window$)`.
- `next(v) →` inner next.
- `closeNext →` inner complete · outer `next(newWindow$)`.
- `complete →` inner complete · outer complete.

## Worked trace

Two closer emissions produce three windows if the source is still active after the second close (the third is the newly opened one).

## Why this is Mealy rather than Moore

Close input writes a two-letter word (complete old, emit new) from the single open-window state.

## Edge cases fixed by the 7.x source

- `closingSelector` runs per window.
- Selector throw is an error.

## Source anchors

- `src/internal/operators/windowWhen.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
