# Test plan: `concatMap`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/concatMap.md](../../operators/transformation/concatMap.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/concatMap.ts` |
| Signature | `concatMap(project, resultSelector?): OperatorFunction<T, R>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Project each source value to an inner observable and flatten with concurrency 1. Later source values wait in a queue until the active inner completes. Inner and outer errors fail the output. Complete when the outer is done and the queue and active inner are empty.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active inner | ⊥, queue of outer values, outerDone, stopped }`.

Memory at subscribe: No active inner, empty queue, outer not done.

Events: `{ outerNext(v), outerError, outerComplete, innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next(r), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` enqueues, and starts an inner if none is active.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerNext` leaves the queue unchanged.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` pops the next queued outer into `active`, or clears `active`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Both outer done and idle → `stopped`.`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Any error → `stopped`.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext(r) → next(r)` (after optional result selector).`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → complete` only if outer is done and the queue is empty, else `ε`.`
9. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε`.`
10. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Errors copy through.`
11. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
12. Source edge 1: `Concurrency is fixed at 1. That is the only difference from `mergeMap` in `mergeInternals`.`
13. Source edge 2: `Project throw → error, queue discarded.`

## Sequence from the analysis

Outer `1, 2` with project `x => of(x, x)` writes `next(1) next(1) next(2) next(2)`. The second inner cannot start before the first completes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
