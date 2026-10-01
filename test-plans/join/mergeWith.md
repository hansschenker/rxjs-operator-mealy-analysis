# Test plan: `mergeWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/mergeWith.md](../../operators/join/mergeWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/mergeWith.ts` |
| Signature | `mergeWith(...otherSources): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Subscribe to the source and the other sources together and forward whichever next arrives. Complete when all complete. Error on the first error. Concurrency is unbounded.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Active set and completed count, or stopped.

Memory at subscribe: All sources subscribed, completed count 0.

Events: `{ next_i, error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next_i` does not change membership.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete_i` increments the done count.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `All done → stopped.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Any error → stopped.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next_i(v) → next(v)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete_i → complete` only when the done count equals the source count, else `ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error_i → error`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Equivalent to `merge(source, ...others)`.`
11. Source edge 2: `No concurrency argument on this signature.`

## Sequence from the analysis

`of(1).pipe(mergeWith(of(2)))` writes both values, order following synchronous subscription order, then complete.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
