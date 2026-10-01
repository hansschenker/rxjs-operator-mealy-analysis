# `throwError` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold error producer |
| RxJS 7.x source | `src/internal/observable/throwError.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `throwError(errorFactory: () => any, scheduler?: SchedulerLike): Observable<never>` |
| Status on the 7.x line | Stable. Passing an error instance directly is deprecated; the factory form is per-subscription. |

SuperGrok is the main contributor of this analysis.

## Explanation

`throwError` is a cold error producer on the RxJS 7.x line. Stable. Passing an error instance directly is deprecated; the factory form is per-subscription. Subscribe calls the error factory and writes `error`. No `next`, no `complete`. Scheduler delays the error notification.

In plain terms, the operator keeps this memory: S = { idle, scheduled, stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, schedulerTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: throwError(() => new Error('x')) writes error(Error('x')) on subscribe.

Details that a marble diagram often leaves out: Instance form is deprecated because the same error object would be shared across subscriptions. Does not complete.

## Role in the notification machine

Subscribe calls the error factory and writes `error`. No `next`, no `complete`. Scheduler delays the error notification.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, scheduled, stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, schedulerTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ error(e) }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → stopped` (sync) or `scheduled`.
- `scheduled × schedulerTick → stopped`.
- `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `subscribe` or `schedulerTick → error(factory())`.
- Factory throw is still an `error` notification.

## Worked trace

`throwError(() => new Error('x'))` writes `error(Error('x'))` on subscribe.

## Why this is Mealy rather than Moore

The productive input is subscribe (or its tick). The error value is computed at that input, so two subscriptions can fail differently if the factory is impure.

## Edge cases fixed by the 7.x source

- Instance form is deprecated because the same error object would be shared across subscriptions.
- Does not complete.

## Source anchors

- `src/internal/observable/throwError.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
