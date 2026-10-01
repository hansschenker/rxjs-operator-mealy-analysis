# Test plan: `count`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/mathematical/count.md](../../operators/mathematical/count.md).

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/count.ts` |
| Signature | `count(predicate?): OperatorFunction<T, number>` |
| Status | Stable. |

## Behavior under test

Count source nexts, or count those matching `predicate`. Emit the count on complete, then complete. No emission before complete.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { counting(n, i), stopped }`.

Memory at subscribe: `counting(0, 0)`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(number), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Matching next increments n and i.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Non-matching increments i only.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(n) · complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Predicate throw → `error`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Empty source emits `next(0) complete`.`
10. Source edge 2: `Predicate receives index.`

## Sequence from the analysis

`of(1, 2, 3, 4).pipe(count(x => x % 2 === 0))` writes `next(2) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
