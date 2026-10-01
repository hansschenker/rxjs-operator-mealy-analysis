# `takeUntil` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/takeUntil.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `takeUntil(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Mirror the source until `notifier` emits or completes. Then complete and unsubscribe the source. Notifier error is an error. The notifier value is not emitted.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { forwarding, stopped }`.

## 2. Initial state (S0)

`forwarding` with notifier subscribed.

## 3. Input alphabet (Z)

`{ next(v), error, complete, notifierNext, notifierComplete, notifierError, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `notifierNext` or `notifierComplete → stopped`.
- Source complete → `stopped`.
- Source next stays forwarding.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(v)` while forwarding.
- `notifierNext` or `notifierComplete → complete`.
- `notifierError → error`.

## Worked trace

Interval taken until a click writes the interval values so far, then `complete`, and unsubscribes the interval.

## Why this is Mealy rather than Moore

Notifier next writes `complete` without forwarding the notifier value. Source next writes `next`. Input kind selects the word.

## Edge cases fixed by the 7.x source

- Notifier complete also stops the output.
- If the notifier emits synchronously on subscribe, the source may be unsubscribed immediately.

## Source anchors

- `src/internal/operators/takeUntil.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
