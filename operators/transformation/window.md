# `window` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/window.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `window(windowBoundaries: Observable<any>): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Emit a window observable immediately, forward source values into it, and on each boundary notifier next complete that window and emit a new one. Source complete completes the open window and the outer.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open window subject, stopped }`.

## 2. Initial state (S0)

A first window already emitted on subscribe.

## 3. Input alphabet (Z)

`{ subscribe, next(v), error, complete, boundaryNext, boundaryError, boundaryComplete, unsubscribe }`.

## 4. Output alphabet (A)

Outer `{ next(Observable), error, complete }`. Window `{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `boundaryNext` completes the current window and opens another.
- `next(v)` leaves the same window open.
- Source or boundary terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `subscribe → next(window$)`.
- `next(v) →` window `next(v)`; outer `ε`.
- `boundaryNext →` window `complete`, outer `next(newWindow$)`.
- `complete →` window `complete` · outer `complete`.

## Worked trace

Two boundary signals around values `1, 2` then `3` produce two window observables, the first emitting 1 and 2, the second emitting 3.

## Why this is Mealy rather than Moore

Boundary input writes a window-complete plus a new outer next; source next writes into the current window. Same outer state, different input, different word.

## Edge cases fixed by the 7.x source

- Windows are Subjects; late subscribers miss past values of that window.
- Boundary complete completes the current window and the outer.

## Source anchors

- `src/internal/operators/window.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
