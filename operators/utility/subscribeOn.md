# `subscribeOn` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/subscribeOn.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `subscribeOn(scheduler, delay = 0): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`subscribeOn` is a pipeable operator on the RxJS 7.x line. Stable. Schedule the act of subscribing to the source. After that, notifications are forwarded directly (no extra observe queue). Unsubscribe before the scheduled subscribe cancels it.

In plain terms, the operator keeps this memory: S = { scheduled, forwarding, stopped }. At subscription, before any source notification, that memory is scheduled. It reacts to these events: { subscribe, schedulerTick, sourceNext, sourceError, sourceComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A synchronous of(1) under subscribeOn(asyncScheduler) does not emit inside the caller stack; it emits when the scheduler runs the subscription.

Details that a marble diagram often leaves out: Does not move notifications onto the scheduler; pair with `observeOn` for that. Delay delays the subscription, not each value.

## Role in the notification machine

Schedule the act of subscribing to the source. After that, notifications are forwarded directly (no extra observe queue). Unsubscribe before the scheduled subscribe cancels it.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { scheduled, forwarding, stopped }`.

## 2. Initial state (S0)

`scheduled`.

## 3. Input alphabet (Z)

`{ subscribe, schedulerTick, sourceNext, sourceError, sourceComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `schedulerTick → forwarding` and subscribe to source.
- Source terminal → `stopped`.
- Unsubscribe while scheduled → `stopped` without subscribing.

## 6. Output function (G : S × Z → A*)

- `subscribe → ε`.
- `schedulerTick → ε` (the subscribe action).
- Source notifications copy through once forwarding.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { scheduled, forwarding, stopped }.

Memory at subscribe: scheduled.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| schedulerTick | forwarding and subscribe to source | ε (the subscribe action) |
| Source terminal | stopped | ε |
| Unsubscribe while scheduled | stopped without subscribing | nothing named on a separate output row |
| Source notifications copy through once forwarding | named by the output row; memory change is in the transition rows above | Source notifications copy through once forwarding |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A synchronous `of(1)` under `subscribeOn(asyncScheduler)` does not emit inside the caller stack; it emits when the scheduler runs the subscription.

## Why this is Mealy rather than Moore

The tick input changes state and performs the subscribe action. Source next is a later input that writes the value letter.

## Edge cases fixed by the 7.x source

- Does not move notifications onto the scheduler; pair with `observeOn` for that.
- Delay delays the subscription, not each value.

## Source anchors

- `src/internal/operators/subscribeOn.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
