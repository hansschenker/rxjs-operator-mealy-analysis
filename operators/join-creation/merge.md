# `merge` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/merge.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `merge(...sources, concurrent?: number): Observable<T>` |
| Status on the 7.x line | Stable. Pipeable form is `mergeWith` / `mergeAll`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`merge` is a join creation function on the RxJS 7.x line. Stable. Pipeable form is `mergeWith` / `mergeAll`. Subscribe to sources, up to `concurrent` at a time (default Infinity), and forward their nexts as they arrive. Complete when every source has completed. Error on the first error. Extra sources wait in a queue when concurrency is bounded.

In plain terms, the operator keeps this memory: S = { active set, queue of not-yet-subscribed sources, completed count } ∪ { stopped }. At subscription, before any source notification, that memory is Empty active set, full queue, completed count 0. It reacts to these events: { subscribe, innerNext, innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: merge(timer(2).pipe(mapTo('late')), of('early')) may write next('early') before next('late'). Order across sources is arrival order.

Details that a marble diagram often leaves out: A numeric trailing argument is concurrency, not a source. `concurrent: 1` is sequential and matches `concat` order.

## Role in the notification machine

Subscribe to sources, up to `concurrent` at a time (default Infinity), and forward their nexts as they arrive. Complete when every source has completed. Error on the first error. Extra sources wait in a queue when concurrency is bounded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active set, queue of not-yet-subscribed sources, completed count } ∪ { stopped }`.

## 2. Initial state (S0)

Empty active set, full queue, completed count 0.

## 3. Input alphabet (Z)

`{ subscribe, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Subscribe fills `active` up to the concurrency limit.
- `innerNext` does not change membership.
- `innerComplete` removes that inner, increments completed, pulls from the queue if any.
- All sources completed → `stopped`.
- `innerError → stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext(v) → next(v)`.
- `innerComplete → ε`, or `complete` when the completed count reaches the source count.
- `innerError(e) → error(e)`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active set, queue of not-yet-subscribed sources, completed count } ∪ { stopped }.

Memory at subscribe: Empty active set, full queue, completed count 0.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Subscribe fills active up to the concurrency limit | Subscribe fills active up to the concurrency limit | nothing named on a separate output row |
| innerNext does not change membership | innerNext does not change membership | next(v) |
| innerComplete removes that inner, increments completed, pulls from the queue if any | innerComplete removes that inner, increments completed, pulls from the queue if any | ε, or complete when the completed count reaches the source count |
| All sources completed | stopped | nothing named on a separate output row |
| innerError | stopped | error(e) |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`merge(timer(2).pipe(mapTo('late')), of('early'))` may write `next('early')` before `next('late')`. Order across sources is arrival order.

## Why this is Mealy rather than Moore

Whether `innerComplete` writes `complete` or `ε` depends on the completed-count state and on that input.

## Edge cases fixed by the 7.x source

- A numeric trailing argument is concurrency, not a source.
- `concurrent: 1` is sequential and matches `concat` order.

## Source anchors

- `src/internal/observable/merge.ts` and `src/internal/operators/mergeAll.ts` via `mergeInternals`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
