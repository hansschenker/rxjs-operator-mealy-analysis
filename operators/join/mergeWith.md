# `mergeWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/mergeWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `mergeWith(...otherSources): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`mergeWith` is a pipeable join on the RxJS 7.x line. Stable. Subscribe to the source and the other sources together and forward whichever next arrives. Complete when all complete. Error on the first error. Concurrency is unbounded.

In plain terms, the operator keeps this memory: Active set and completed count, or stopped. At subscription, before any source notification, that memory is All sources subscribed, completed count 0. It reacts to these events: { next_i, error_i, complete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1).pipe(mergeWith(of(2))) writes both values, order following synchronous subscription order, then complete.

Details that a marble diagram often leaves out: Equivalent to `merge(source, ...others)`. No concurrency argument on this signature.

## Role in the notification machine

Subscribe to the source and the other sources together and forward whichever next arrives. Complete when all complete. Error on the first error. Concurrency is unbounded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Active set and completed count, or stopped.

## 2. Initial state (S0)

All sources subscribed, completed count 0.

## 3. Input alphabet (Z)

`{ next_i, error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next_i` does not change membership.
- `complete_i` increments the done count.
- All done → stopped.
- Any error → stopped.

## 6. Output function (G : S × Z → A*)

- `next_i(v) → next(v)`.
- `complete_i → complete` only when the done count equals the source count, else `ε`.
- `error_i → error`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: Active set and completed count, or stopped.

Memory at subscribe: All sources subscribed, completed count 0.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next_i does not change membership | next_i does not change membership | next(v) |
| complete_i increments the done count | complete_i increments the done count | complete only when the done count equals the source count, else ε |
| All done | stopped | error |
| Any error | stopped | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1).pipe(mergeWith(of(2)))` writes both values, order following synchronous subscription order, then complete.

## Why this is Mealy rather than Moore

A complete input writes complete or ε depending on the done-count state.

## Edge cases fixed by the 7.x source

- Equivalent to `merge(source, ...others)`.
- No concurrency argument on this signature.

## Source anchors

- `src/internal/operators/mergeWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
