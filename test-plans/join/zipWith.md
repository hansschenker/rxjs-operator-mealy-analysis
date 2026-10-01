# Test plan: `zipWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/zipWith.md](../../operators/join/zipWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/zipWith.ts` |
| Signature | `zipWith(...otherSources): OperatorFunction<T, any[]>` |
| Status | Stable. |

## Behavior under test

Zip the piped source with the other sources by index. Each side has a queue. Emit when every queue is non-empty, then shift one from each. Complete when any side completes and cannot form another tuple.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Queue per source plus done flags, or stopped.

Memory at subscribe: Empty queues.

Events: `{ next_i, error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Next appends. Shift a row when all queues are non-empty.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete on an empty queue → stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Next → next(tuple) if the append filled the last hole, else ε.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete → complete if no further row is possible, else ε.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Error → error.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Values are consumed, not reused as in combineLatest.`
10. Source edge 2: `Same pairing rule as creation `zip`.`

## Sequence from the analysis

`of(1, 2).pipe(zipWith(of('a')))` writes `next([1, 'a']) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
