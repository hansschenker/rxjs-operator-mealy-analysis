# `repeatWhen` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Deprecated notifier resubscribe |
| RxJS 7.x source | `src/internal/operators/repeatWhen.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `repeatWhen(notifier: (notifications) => Observable<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Deprecated. Use `repeat({ delay })`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

On source complete, push a notification into a subject and resubscribe when `notifier` emits. Notifier error or complete ends the output. Source error is forwarded and does not repeat.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ forwarding, waiting, stopped }`.

## 2. Initial state (S0)

`forwarding`, notifier subscribed.

## 3. Input alphabet (Z)

`{ next, error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source complete → waiting and signals the notifier.
- notifierNext → forwarding via resubscribe.
- notifier terminal or source error → stopped.

## 6. Output function (G : S × Z → A*)

- Source next → next.
- Source complete → ε.
- notifierError → error.
- notifierComplete → complete.
- Source error → error.

## Worked trace

A notifier that emits once causes the source to be subscribed twice. The first complete is not written downstream.

## Why this is Mealy rather than Moore

Complete is consumed as an input to the notifier. The output complete comes from a later notifier input, not from the source complete.

## Edge cases fixed by the 7.x source

- Notifier is subscribed once.
- Deprecated.

## Source anchors

- `src/internal/operators/repeatWhen.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
