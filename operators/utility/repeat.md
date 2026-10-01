# `repeat` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable resubscribe |
| RxJS 7.x source | `src/internal/operators/repeat.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `repeat(countOrConfig?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. Config form `{ count, delay }` on 7.x. |

SuperGrok is the main contributor of this analysis.

## Explanation

`repeat` is a pipeable resubscribe on the RxJS 7.x line. Stable. Config form `{ count, delay }` on 7.x. Forward the source. On complete, resubscribe if repeats remain. `count` is the number of times the source is subscribed in total in the numeric form used by 7.x docs (repeat(1) means one subscription, no extra repeat). Delay waits before the resubscribe. Error is not repeated; it is forwarded. Infinite count repeats forever.

In plain terms, the operator keeps this memory: { forwarding(n), waitingDelay, stopped }. At subscription, before any source notification, that memory is forwarding(1). It reacts to these events: { next, error, complete, delayTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1).pipe(repeat(2)) writes next(1) next(1) complete. The complete of the first subscription is swallowed.

Details that a marble diagram often leaves out: Error does not repeat. Delay notifier error becomes the output error. count Infinity never writes the final complete.

## Role in the notification machine

Forward the source. On complete, resubscribe if repeats remain. `count` is the number of times the source is subscribed in total in the numeric form used by 7.x docs (repeat(1) means one subscription, no extra repeat). Delay waits before the resubscribe. Error is not repeated; it is forwarded. Infinite count repeats forever.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ forwarding(n), waitingDelay, stopped }`.

## 2. Initial state (S0)

`forwarding(1)`.

## 3. Input alphabet (Z)

`{ next, error, complete, delayTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next stays.
- Complete → waitingDelay if another subscription is allowed, else stopped.
- delayTick → forwarding(n+1).
- Error → stopped.

## 6. Output function (G : S × Z → A*)

- Next → next.
- Complete → ε if a repeat will happen, else complete.
- Error → error.
- Resubscribe is an action.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { forwarding(n), waitingDelay, stopped }.

Memory at subscribe: forwarding(1).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Next stays | Next stays | next |
| Complete | waitingDelay if another subscription is allowed, else stopped | ε if a repeat will happen, else complete |
| delayTick | forwarding(n+1) | nothing named on a separate output row |
| Error | stopped | error |
| Resubscribe is an action | named by the output row; memory change is in the transition rows above | Resubscribe is an action |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1).pipe(repeat(2))` writes `next(1) next(1) complete`. The complete of the first subscription is swallowed.

## Why this is Mealy rather than Moore

Complete writes ε or complete depending on the remaining-count state. That is the dual of retry, which does the same on error.

## Edge cases fixed by the 7.x source

- Error does not repeat.
- Delay notifier error becomes the output error.
- count Infinity never writes the final complete.

## Source anchors

- `src/internal/operators/repeat.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
