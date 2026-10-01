# Test plan: `find`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/conditional/find.md](../../operators/conditional/find.md).

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/find.ts` |
| Signature | `find(predicate, thisArg?): OperatorFunction<T, T |
| Status | Stable. |

## Behavior under test

Emit the first matching value and complete. If the source completes with no match, emit `undefined` and complete. Does not error on a miss. Unsubscribes after a match.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { searching(i), stopped }`.

Memory at subscribe: `searching(0)`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(T | undefined), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Non-match increments i.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Match → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Match → `next(v) · complete`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Non-match → `ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete → `next(undefined) · complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Unlike `first`, a miss is `undefined`, not `EmptyError`.`
10. Source edge 2: `Index is passed to the predicate.`

## Sequence from the analysis

`find(x => x > 10)` on `1 2 3` writes `next(undefined) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
