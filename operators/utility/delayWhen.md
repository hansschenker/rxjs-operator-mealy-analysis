# `delayWhen` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/delayWhen.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `delayWhen(delayDurationSelector, subscriptionDelay?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`delayWhen` is a pipeable operator on the RxJS 7.x line. Stable. Each source next subscribes to `delayDurationSelector(value, index)` and emits the value when that duration emits. Complete waits until every outstanding delay has emitted. Error is immediate. Optional `subscriptionDelay` delays the subscription to the source itself.

In plain terms, the operator keeps this memory: S = { delaying set of (value, durationSub), sourceDone, stopped }. At subscription, before any source notification, that memory is Empty set. Source not yet subscribed if subscriptionDelay is set. It reacts to these events: { next(v), error, complete, durationNext_i, durationError_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two values with different duration selectors can emit out of source order.

Details that a marble diagram often leaves out: Duration selector throw is an error. Subscription delay is an extra initial wait state.

## Role in the notification machine

Each source next subscribes to `delayDurationSelector(value, index)` and emits the value when that duration emits. Complete waits until every outstanding delay has emitted. Error is immediate. Optional `subscriptionDelay` delays the subscription to the source itself.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { delaying set of (value, durationSub), sourceDone, stopped }`.

## 2. Initial state (S0)

Empty set. Source not yet subscribed if subscriptionDelay is set.

## 3. Input alphabet (Z)

`{ next(v), error, complete, durationNext_i, durationError_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` adds a delay entry.
- `durationNext_i` removes it.
- Source complete marks sourceDone; stop when the set is empty.
- Error → `stopped`.

## 6. Output function (G : S × Z → A*)

- `durationNext_i → next(value_i)`.
- Source next → `ε`.
- Source complete → `ε`, then `complete` when the last delay fires.
- `error → error` now.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { delaying set of (value, durationSub), sourceDone, stopped }.

Memory at subscribe: Empty set. Source not yet subscribed if subscriptionDelay is set.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next adds a delay entry | next adds a delay entry | next(value_i) |
| durationNext_i removes it | durationNext_i removes it | ε |
| Source complete marks sourceDone; stop when the set is empty | Source complete marks sourceDone; stop when the set is empty | ε, then complete when the last delay fires |
| Error | stopped | error now |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Two values with different duration selectors can emit out of source order.

## Why this is Mealy rather than Moore

Duration-next input writes the value stored with that duration. Source next only updates state.

## Edge cases fixed by the 7.x source

- Duration selector throw is an error.
- Subscription delay is an extra initial wait state.

## Source anchors

- `src/internal/operators/delayWhen.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
