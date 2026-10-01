# Test plan: `mapTo`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/mapTo.md](../../operators/transformation/mapTo.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/mapTo.ts` |
| Signature | `mapTo(value: R): OperatorFunction<T, R>` |
| Status | Deprecated in 7.x. Use `map(() => value)`. |

## Behavior under test

Replace every source next with the same constant. No index. Error and complete pass through.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active, stopped }`.

Memory at subscribe: `active`.

Events: `{ next(_), error, complete, unsubscribe }`.

Possible notifications: `{ next(constant), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``active × next → active`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``active × next(_) → next(constant)`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error` and `complete` copy through.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `The constant is captured when `mapTo` is called, not per next.`
8. Source edge 2: `Deprecated, tuple unchanged.`

## Sequence from the analysis

`of(1, 2, 3).pipe(mapTo('x'))` writes `next('x') next('x') next('x') complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
