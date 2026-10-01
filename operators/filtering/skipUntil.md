# `skipUntil` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipUntil.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `skipUntil(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Drop source values until `notifier` emits once. Then unsubscribe the notifier and mirror the source. Notifier error is an error. Notifier complete without a next leaves the machine skipping forever until the source ends.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { skipping, forwarding, stopped }`.

## 2. Initial state (S0)

`skipping`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `skipping × notifierNext → forwarding`.
- `skipping × next → skipping`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `skipping × next → ε`.
- `forwarding × next → next(v)`.
- `notifierNext → ε` (it only flips state).

## Worked trace

Source values before a click are dropped; the click itself is not emitted; later source values pass.

## Why this is Mealy rather than Moore

Source next writes `ε` or `next(v)` according to whether notifier input has already moved the state.

## Edge cases fixed by the 7.x source

- Notifier value is ignored, only its arrival matters.
- Source complete while still skipping writes `complete` with no values.

## Source anchors

- `src/internal/operators/skipUntil.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
