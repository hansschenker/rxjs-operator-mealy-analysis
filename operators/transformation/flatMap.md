# `flatMap` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Deprecated alias |
| RxJS 7.x source | `src/internal/operators/flatMap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `flatMap(project, resultSelector?, concurrent?): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated alias of `mergeMap`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`flatMap` is a deprecated alias on the RxJS 7.x line. Deprecated alias of `mergeMap`. `flatMap.ts` re-exports `mergeMap`. Concurrency defaults to Infinity. Inners are subscribed as outer values arrive, up to the limit, and their nexts are interleaved.

In plain terms, the operator keeps this memory: Same as mergeMap: active count, queue, outerDone, stopped. At subscription, before any source notification, that memory is Active 0, empty queue. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Same word as mergeMap on the same projection.

Details that a marble diagram often leaves out: File is a re-export. Prefer `mergeMap`.

## Role in the notification machine

`flatMap.ts` re-exports `mergeMap`. Concurrency defaults to Infinity. Inners are subscribed as outer values arrive, up to the limit, and their nexts are interleaved.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `mergeMap`: active count, queue, outerDone, stopped.

## 2. Initial state (S0)

Active 0, empty queue.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Identical to `mergeMap`.

## 6. Output function (G : S × Z → A*)

- Identical to `mergeMap`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: Same as mergeMap: active count, queue, outerDone, stopped.

Memory at subscribe: Active 0, empty queue.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Identical to mergeMap | Identical to mergeMap | Identical to mergeMap |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Same word as `mergeMap` on the same projection.

## Why this is Mealy rather than Moore

Alias only. The complete-or-ε decision on innerComplete still reads active count and outerDone.

## Edge cases fixed by the 7.x source

- File is a re-export.
- Prefer `mergeMap`.

## Source anchors

- `src/internal/operators/flatMap.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
