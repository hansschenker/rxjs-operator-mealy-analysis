# Test plan: `dematerialize`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/dematerialize.md](../../operators/utility/dematerialize.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/dematerialize.ts` |
| Signature | `dematerialize(): OperatorFunction<Notification<T>, T>` |
| Status | Stable. |

## Behavior under test

Inverse of `materialize`. A next-notification becomes `next(value)`. An error-notification becomes `error`. A complete-notification becomes `complete`. The machine stops on those terminal notifications.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active, stopped }`.

Memory at subscribe: `active`.

Events: `{ next(Notification), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Next-notification stays active.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error-notification or complete-notification → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `A real source error also stops.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(Notification.createNext(v)) → next(v)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(Notification.createError(e)) → error(e)`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(Notification.createComplete()) → complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `A real complete on the source completes without synthesizing an extra value.`
10. Source edge 2: `Expects Notification objects; other values are an error in practice.`

## Sequence from the analysis

Materialized `next(1), next(complete)` dematerializes to `next(1) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
