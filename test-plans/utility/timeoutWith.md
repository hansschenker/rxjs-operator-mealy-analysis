# Test plan: `timeoutWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/timeoutWith.md](../../operators/utility/timeoutWith.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeoutWith.ts` |
| Signature | `timeoutWith(due, withObservable, scheduler?): OperatorFunction<T, T |
| Status | Deprecated. Use `timeout({ each, with })`. |

## Behavior under test

Numeric timeout that switches to `withObservable` instead of erroring. Same waiting-timer machine as `timeout` with a replacement branch.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { waiting, switched, stopped }`.

Memory at subscribe: `waiting`.

Events: `{ next, error, complete, timeoutTick, innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Timeout tick → `switched` and subscribe to `with`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next rearms while waiting.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Timeout tick → `ε` (switch action).`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner notifications then copy through.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `No TimeoutError on the default path.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Deprecated.`
9. Source edge 2: `Scheduler defaults to async.`

## Sequence from the analysis

A silent source and `timeoutWith(1000, of('fallback'))` writes `next('fallback') complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
