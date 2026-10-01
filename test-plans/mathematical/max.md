# Test plan: `max`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/mathematical/max.md](../../operators/mathematical/max.md).

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/max.ts` |
| Signature | `max(comparer?): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Reduce with a greater-than comparer. Emit the max on complete. Empty source errors with `EmptyError`. Comparer defaults to numeric `>`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { empty, holding(max), stopped }`.

Memory at subscribe: `empty`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``empty × next(v) → holding(v)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``holding × next(v) → holding(v)` if v wins the comparer, else unchanged.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` in `holding → next(max) · complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` in `empty → error(EmptyError)`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Empty errors.`
10. Source edge 2: `Comparer throw is an error.`
11. Source edge 3: `Implemented as a reduce.`

## Sequence from the analysis

`of(3, 1, 2).pipe(max())` writes `next(3) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
