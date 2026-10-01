# Test plan: `concatWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/concatWith.md](../../operators/join/concatWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/concatWith.ts` |
| Signature | `concatWith(...otherSources): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Forward the source to completion, then subscribe to each additional source in order. One active subscription. An error skips the rest.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ reading(i) | 0 ≤ i ≤ n } ∪ { stopped }`. `i = 0` is the piped source.

Memory at subscribe: `reading(0)`.

Events: `{ innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerNext` stays on i.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` moves to i+1 or stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerError → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → ε` if another source remains, else `complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerError → error`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Later sources are not subscribed until earlier ones complete.`
10. Source edge 2: `Implemented as concat of the source plus the rest.`

## Sequence from the analysis

`of(1).pipe(concatWith(of(2, 3)))` writes `next(1) next(2) next(3) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
