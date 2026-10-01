# `windowToggle` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowToggle.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `windowToggle(openings, closingSelector): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Window analogue of `bufferToggle`. Each opening emits a new window observable and subscribes to a closer. Source values go to every open window. Closer next completes that window.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { list of open windows, stopped }`.

## 2. Initial state (S0)

No window until the first opening.

## 3. Input alphabet (Z)

`{ next(v), error, complete, opening, closing_i, unsubscribe }`.

## 4. Output alphabet (A)

Outer window observables; inner notifications.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `opening` appends a window.
- `closing_i` removes it.
- `next` does not change membership.

## 6. Output function (G : S × Z → A*)

- `opening → next(window$)`.
- `next(v) →` inner next on each open window.
- `closing_i →` that window `complete`.
- Source complete completes open windows and the outer.

## Worked trace

One opening, two values, one closing: one window observable emits both values and completes.

## Why this is Mealy rather than Moore

Opening and closing inputs write outer or inner terminal letters; source next writes inner letters. Membership state selects the targets.

## Edge cases fixed by the 7.x source

- Values with no open window are dropped.
- Closing complete without a value does not emit a boundary by itself in the same way a next does; it unsubscribes the closer.

## Source anchors

- `src/internal/operators/windowToggle.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
