# `finalize` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable teardown |
| RxJS 7.x source | `src/internal/operators/finalize.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `finalize(callback): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`finalize` is a pipeable teardown on the RxJS 7.x line. Stable. Mirror every notification. Call `callback` once on unsubscribe, error, or complete — whichever ends the subscription first. The callback is not a notification. A throw from the callback is reported on teardown.

In plain terms, the operator keeps this memory: { active, stopped }. A flag records that the callback has run. At subscription, before any source notification, that memory is active, callback not run. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A take(1) downstream unsubscribes after the first next; finalize runs on that unsubscribe, not on a later source complete.

Details that a marble diagram often leaves out: Callback runs on unsubscribe even if the source has not terminated. It runs once, not per notification.

## Role in the notification machine

Mirror every notification. Call `callback` once on unsubscribe, error, or complete — whichever ends the subscription first. The callback is not a notification. A throw from the callback is reported on teardown.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ active, stopped }`. A flag records that the callback has run.

## 2. Initial state (S0)

`active`, callback not run.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }` plus the finalize action.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next stays active.
- Error, complete, or unsubscribe → stopped and marks the callback done.

## 6. Output function (G : S × Z → A*)

- Notifications are copied.
- The ending input also runs the callback exactly once.
- Unsubscribe writes `ε` downstream and still runs the callback.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { active, stopped }. A flag records that the callback has run.

Memory at subscribe: active, callback not run.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Next stays active | Next stays active | nothing named on a separate output row |
| Error, complete, or unsubscribe | stopped and marks the callback done | Unsubscribe writes ε downstream and still runs the callback |
| Notifications are copied | named by the output row; memory change is in the transition rows above | Notifications are copied |
| The ending input also runs the callback exactly once | named by the output row; memory change is in the transition rows above | The ending input also runs the callback exactly once |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A take(1) downstream unsubscribes after the first next; finalize runs on that unsubscribe, not on a later source complete.

## Why this is Mealy rather than Moore

The ending input both forwards a terminal letter (or `ε` on unsubscribe) and fires the callback. Next does neither side effect.

## Edge cases fixed by the 7.x source

- Callback runs on unsubscribe even if the source has not terminated.
- It runs once, not per notification.

## Source anchors

- `src/internal/operators/finalize.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
