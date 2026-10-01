# Test plan: `delayWhen`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/delayWhen.md](../../operators/utility/delayWhen.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/delayWhen.ts` |
| Signature | `delayWhen(delayDurationSelector, subscriptionDelay?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Each source next subscribes to `delayDurationSelector(value, index)` and emits the value when that duration emits. Complete waits until every outstanding delay has emitted. Error is immediate. Optional `subscriptionDelay` delays the subscription to the source itself.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { delaying set of (value, durationSub), sourceDone, stopped }`.

Memory at subscribe: Empty set. Source not yet subscribed if subscriptionDelay is set.

Events: `{ next(v), error, complete, durationNext_i, durationError_i, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` adds a delay entry.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``durationNext_i` removes it.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete marks sourceDone; stop when the set is empty.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``durationNext_i → next(value_i)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → `ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source complete → `ε`, then `complete` when the last delay fires.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error` now.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Duration selector throw is an error.`
12. Source edge 2: `Subscription delay is an extra initial wait state.`

## Sequence from the analysis

Two values with different duration selectors can emit out of source order.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
