# Test plan: `finalize`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/finalize.md](../../operators/utility/finalize.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable teardown |
| RxJS 7.x source | `src/internal/operators/finalize.ts` |
| Signature | `finalize(callback): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Mirror every notification. Call `callback` once on unsubscribe, error, or complete — whichever ends the subscription first. The callback is not a notification. A throw from the callback is reported on teardown.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ active, stopped }`. A flag records that the callback has run.

Memory at subscribe: `active`, callback not run.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next, error, complete }` plus the finalize action.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Next stays active.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error, complete, or unsubscribe → stopped and marks the callback done.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Notifications are copied.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `The ending input also runs the callback exactly once.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Unsubscribe writes `ε` downstream and still runs the callback.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Callback runs on unsubscribe even if the source has not terminated.`
9. Source edge 2: `It runs once, not per notification.`

## Sequence from the analysis

A take(1) downstream unsubscribes after the first next; finalize runs on that unsubscribe, not on a later source complete.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
