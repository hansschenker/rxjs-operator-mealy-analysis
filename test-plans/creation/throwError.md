# Test plan: `throwError`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/throwError.md](../../operators/creation/throwError.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold error producer |
| RxJS 7.x source | `src/internal/observable/throwError.ts` |
| Signature | `throwError(errorFactory: () => any, scheduler?: SchedulerLike): Observable<never>` |
| Status | Stable. Passing an error instance directly is deprecated; the factory form is per-subscription. |

## Behavior under test

Subscribe calls the error factory and writes `error`. No `next`, no `complete`. Scheduler delays the error notification.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, scheduled, stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, schedulerTick, unsubscribe }`.

Possible notifications: `{ error(e) }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → stopped` (sync) or `scheduled`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``scheduled × schedulerTick → stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe` or `schedulerTick → error(factory())`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Factory throw is still an `error` notification.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Instance form is deprecated because the same error object would be shared across subscriptions.`
9. Source edge 2: `Does not complete.`

## Sequence from the analysis

`throwError(() => new Error('x'))` writes `error(Error('x'))` on subscribe.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
