# Test plan: `timeout`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/timeout.md](../../operators/utility/timeout.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeout.ts` |
| Signature | `timeout(configOrDue, scheduler?): OperatorFunction<T, T |
| Status | Stable. 7.x config form `{ each, first, with, meta, scheduler }` is the full machine. Numeric due is shorthand for `each`. |

## Behavior under test

Arm a timer on subscribe (`first`) and after each next (`each`). If the timer fires before the next source signal, either error with `TimeoutError` or switch to the `with` observable. A source next resets the each-timer.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { waiting(timer), forwardingReplacement, stopped }`.

Memory at subscribe: `waiting` with the first-timer armed.

Events: `{ next, error, complete, timeoutTick, replacementNext, replacementError, replacementComplete, unsubscribe }`.

Possible notifications: `{ next, error(TimeoutError), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` cancels and rearms the each-timer.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``timeoutTick → stopped` if no `with`, else `forwardingReplacement`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → `stopped` and cancels the timer.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → next` and rearm action.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``timeoutTick → error(TimeoutError)` or `ε` plus switch.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Replacement notifications copy through.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: ``with` changes the timeout from a terminal error into a switch.`
10. Source edge 2: ``meta` is attached to TimeoutError.`
11. Source edge 3: `Absolute dates are allowed for first.`

## Sequence from the analysis

`each: 1000` with a silent source writes `error(TimeoutError)` about a second after subscribe if `first` is also exceeded.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
