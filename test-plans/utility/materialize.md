# Test plan: `materialize`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/materialize.md](../../operators/utility/materialize.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/materialize.ts` |
| Signature | `materialize(): OperatorFunction<T, Notification<T>>` |
| Status | Stable. |

## Behavior under test

Turn next into `next(Notification.createNext(v))`. Turn error into `next(Notification.createError(e))` followed by `complete`. Turn complete into `next(Notification.createComplete())` followed by `complete`. Downstream sees no error from the source; errors are values.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active, stopped }`.

Memory at subscribe: `active`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(Notification), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next stays active.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source error or complete → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(N.Next(v))`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error(e) → next(N.Error(e)) · complete`.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(N.Complete()) · complete`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `The output does not error for source errors.`
9. Source edge 2: `Useful before a delay if error timing must be queued like values.`

## Sequence from the analysis

A failing source becomes a completing source whose last value is an error notification.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
