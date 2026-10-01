# Test plan: `retry`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/error-handling/retry.md](../../operators/error-handling/retry.md).

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/retry.ts` |
| Signature | `retry(countOrConfig?): MonoTypeOperatorFunction<T>` |
| Status | Stable. 7.x accepts a number or `{ count, delay, resetOnSuccess }`. |

## Behavior under test

On source error, resubscribe if retries remain. `count` is the number of resubscriptions (Infinity by default in the config object path; the numeric shorthand is the retry count). `delay` can be a duration or a notifier of the error. `resetOnSuccess` clears the attempt counter after a successful next. Complete passes through and does not retry.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { forwarding(attempt), waitingDelay, stopped }`.

Memory at subscribe: `forwarding(0)`.

Events: `{ next, error, complete, delayTick, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` stays forwarding; may reset attempt to 0.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``error` → `waitingDelay` if attempts remain, else `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``delayTick` → `forwarding(attempt+1)` via resubscribe.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → next`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → ε` if a retry will happen, else `error(e)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → complete`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Resubscribe is an action, not a notification.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Count Infinity retries forever.`
12. Source edge 2: `Delay notifier error becomes the output error and stops retries.`
13. Source edge 3: ``resetOnSuccess` changes the counter transition on next.`

## Sequence from the analysis

A source that errors twice with `retry(1)` writes the first attempt's nexts, suppresses the first error, resubscribes, then forwards the second error.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
