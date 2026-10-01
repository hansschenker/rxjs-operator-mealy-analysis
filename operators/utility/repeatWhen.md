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

## Explanation

`repeatWhen` is a deprecated notifier resubscribe on the RxJS 7.x line. Deprecated. Use `repeat({ delay })`. On source complete, push a notification into a subject and resubscribe when `notifier` emits. Notifier error or complete ends the output. Source error is forwarded and does not repeat.

In plain terms, the operator keeps this memory: { forwarding, waiting, stopped }. At subscription, before any source notification, that memory is forwarding, notifier subscribed. It reacts to these events: { next, error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A notifier that emits once causes the source to be subscribed twice. The first complete is not written downstream.

Details that a marble diagram often leaves out: Notifier is subscribed once. Deprecated.

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

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { forwarding, waiting, stopped }.

Memory at subscribe: forwarding, notifier subscribed.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Source complete | waiting and signals the notifier | ε |
| notifierNext | forwarding via resubscribe | next |
| notifier terminal or source error | stopped | error |
| notifierComplete | named by the output row; memory change is in the transition rows above | complete |
| Source error | named by the output row; memory change is in the transition rows above | error |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

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
