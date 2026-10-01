# Test plan: `mergeMap`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/mergeMap.md](../../operators/transformation/mergeMap.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/mergeMap.ts` |
| Signature | `mergeMap(project, resultSelector?, concurrent = Infinity): OperatorFunction<T, R>` |
| Status | Stable. `resultSelector` deprecated. Also known as `flatMap`. |

## Behavior under test

Project each outer value to an inner and subscribe immediately up to `concurrent`. Further outers queue. Forward inner nexts as they arrive, interleaved. Complete when outer is done, queue is empty, and no inner is active.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active count, queue, outerDone, stopped }`.

Memory at subscribe: Active 0, empty queue, outer not done.

Events: Higher-order alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` enqueues and starts inners while `active < concurrent`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` decrements active and starts a queued inner.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Idle, empty queue, outer done → `stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Any error → `stopped` and inners unsubscribed.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → complete` only in the idle-and-outer-done state, else `ε`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``concurrent: 1` reduces to `concatMap`.`
11. Source edge 2: ``flatMap.ts` is a deprecated alias.`

## Sequence from the analysis

`of(1, 2).pipe(mergeMap(x => of(x, x)))` writes four nexts, possibly interleaved if inners were async. With `of` they subscribe sequentially but both are allowed.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
