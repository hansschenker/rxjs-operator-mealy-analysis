# Test plan: `exhaustMap`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/exhaustMap.md](../../operators/transformation/exhaustMap.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/exhaustMap.ts` |
| Signature | `exhaustMap(project, resultSelector?): OperatorFunction<T, R>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Project outer values to inners, but only subscribe when no inner is active. Outer values that arrive while busy are ignored, not queued. This is `exhaust` plus a projection.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, busy, stopped }` with `outerDone`.

Memory at subscribe: `idle`.

Events: Higher-order alphabet; `outerNext(v)` carries the value passed to `project`.

Possible notifications: `{ next(r), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × outerNext → busy` if project returns.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``busy × outerNext → busy` with no subscribe.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete → idle` or `stopped` if outer done.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Project throw or inner error → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → complete` only when outer is done and state returns to idle.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `No queue. Contrast `concatMap`, which would buffer the ignored clicks.`
11. Source edge 2: `Result selector can still see the outer value that was accepted.`

## Sequence from the analysis

`clicks.pipe(exhaustMap(() => interval(1000).pipe(take(3))))` ignores clicks until the current three ticks finish.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
