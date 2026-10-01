# `concatAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/concatAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `concatAll(): OperatorFunction<ObservableInput<T>, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`concatAll` is a pipeable higher-order join on the RxJS 7.x line. Stable. Flatten inners one at a time in the order the source emitted them. Queue later inners. Equivalent to `mergeAll(1)` / `concatMap(x => x)`.

In plain terms, the operator keeps this memory: S = { active, queue, outerDone, stopped }. At subscription, before any source notification, that memory is Idle, empty queue. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source of(of(1, 2), of(3)) through concatAll writes next(1) next(2) next(3) complete.

Details that a marble diagram often leaves out: An inner is not subscribed until it reaches the head of the queue. Error abandons the queue.

## Role in the notification machine

Flatten inners one at a time in the order the source emitted them. Queue later inners. Equivalent to `mergeAll(1)` / `concatMap(x => x)`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, queue, outerDone, stopped }`.

## 2. Initial state (S0)

Idle, empty queue.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` enqueues and starts if idle.
- `innerComplete` starts the next queued inner.
- Idle and outer done → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `innerComplete → complete` only if nothing remains.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active, queue, outerDone, stopped }.

Memory at subscribe: Idle, empty queue.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| outerNext enqueues and starts if idle | outerNext enqueues and starts if idle | next |
| innerComplete starts the next queued inner | innerComplete starts the next queued inner | complete only if nothing remains |
| Idle and outer done | stopped | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Source `of(of(1, 2), of(3))` through `concatAll` writes `next(1) next(2) next(3) complete`.

## Why this is Mealy rather than Moore

Same concurrency-1 Mealy machine as `concatMap` with identity project.

## Edge cases fixed by the 7.x source

- An inner is not subscribed until it reaches the head of the queue.
- Error abandons the queue.

## Source anchors

- `src/internal/operators/concatAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
