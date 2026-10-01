# `retryWhen` — Mealy 6-tuple

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/retryWhen.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `retryWhen(notifier: (errors: Observable<any>) => Observable<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Deprecated. Use `retry({ delay })`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Errors are fed to an errors subject. `notifier(errors)` is subscribed once. When that notifier emits, resubscribe to the source. When the notifier errors or completes, that terminal signal is the output (complete if the notifier completes). Source complete passes through and does not notify.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { forwarding, waiting, stopped }`.

## 2. Initial state (S0)

`forwarding`, notifier subscribed.

## 3. Input alphabet (Z)

`{ next, error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source error → `waiting` and pushes on the errors subject.
- `notifierNext → forwarding` (resubscribe).
- `notifierError|notifierComplete → stopped`.
- Source complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Source next → `next`.
- Source error → `ε` (it is an input to the notifier, not an output yet).
- `notifierError → error`.
- `notifierComplete → complete`.

## Worked trace

Notifier that emits once on error causes one resubscribe. A notifier that completes ends the output with complete rather than the source error.

## Why this is Mealy rather than Moore

Source error writes `ε` and a notifier signal. The eventual terminal word is `G` of a later notifier input. Deprecated, same tuple.

## Edge cases fixed by the 7.x source

- Notifier is subscribed once, not per error.
- A notifier that neither emits nor terminates stalls in `waiting`.

## Source anchors

- `src/internal/operators/retryWhen.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
