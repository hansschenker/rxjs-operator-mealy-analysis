# Test plan: `flatMap`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/flatMap.md](../../operators/transformation/flatMap.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Deprecated alias |
| RxJS 7.x source | `src/internal/operators/flatMap.ts` |
| Signature | `flatMap(project, resultSelector?, concurrent?): OperatorFunction<T, R>` |
| Status | Deprecated alias of `mergeMap`. |

## Behavior under test

`flatMap.ts` re-exports `mergeMap`. Concurrency defaults to Infinity. Inners are subscribed as outer values arrive, up to the limit, and their nexts are interleaved.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Same as `mergeMap`: active count, queue, outerDone, stopped.

Memory at subscribe: Active 0, empty queue.

Events: Higher-order alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Identical to `mergeMap`.`
3. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Identical to `mergeMap`.`
4. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
5. Source edge 1: `File is a re-export.`
6. Source edge 2: `Prefer `mergeMap`.`

## Sequence from the analysis

Same word as `mergeMap` on the same projection.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
