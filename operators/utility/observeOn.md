# `observeOn` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/observeOn.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `observeOn(scheduler, delay = 0): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`observeOn` is a pipeable operator on the RxJS 7.x line. Stable. Schedule each next, error, and complete on `scheduler`. Order is preserved by the scheduler queue. Delay shifts each scheduled action.

In plain terms, the operator keeps this memory: S = { queue of scheduled notifications, stopped }. At subscription, before any source notification, that memory is Empty queue. It reacts to these events: { next, error, complete, scheduledFire, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2).pipe(observeOn(asyncScheduler)) writes next(1) next(2) complete on a later turn, not inside the synchronous subscribe.

Details that a marble diagram often leaves out: Unlike `delay`, error is also scheduled. Delay 0 still leaves the current stack if the scheduler is async.

## Role in the notification machine

Schedule each next, error, and complete on `scheduler`. Order is preserved by the scheduler queue. Delay shifts each scheduled action.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { queue of scheduled notifications, stopped }`.

## 2. Initial state (S0)

Empty queue.

## 3. Input alphabet (Z)

`{ next, error, complete, scheduledFire, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Input notifications enqueue.
- `scheduledFire` dequeues.
- Unsubscribe clears the queue.

## 6. Output function (G : S × Z → A*)

- Source notification → `ε` at arrival.
- `scheduledFire →` the queued letter.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { queue of scheduled notifications, stopped }.

Memory at subscribe: Empty queue.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Input notifications enqueue | Input notifications enqueue | ε at arrival |
| scheduledFire dequeues | scheduledFire dequeues | the queued letter |
| Unsubscribe clears the queue | Unsubscribe clears the queue | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1, 2).pipe(observeOn(asyncScheduler))` writes `next(1) next(2) complete` on a later turn, not inside the synchronous subscribe.

## Why this is Mealy rather than Moore

The fire input writes the letter stored from an earlier notification input. Arrival itself writes `ε`.

## Edge cases fixed by the 7.x source

- Unlike `delay`, error is also scheduled.
- Delay 0 still leaves the current stack if the scheduler is async.

## Source anchors

- `src/internal/operators/observeOn.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
