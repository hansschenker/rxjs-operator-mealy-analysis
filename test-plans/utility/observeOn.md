# Test plan: `observeOn`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/observeOn.md](../../operators/utility/observeOn.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/observeOn.ts` |
| Signature | `observeOn(scheduler, delay = 0): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Schedule each next, error, and complete on `scheduler`. Order is preserved by the scheduler queue. Delay shifts each scheduled action.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { queue of scheduled notifications, stopped }`.

Memory at subscribe: Empty queue.

Events: `{ next, error, complete, scheduledFire, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Input notifications enqueue.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``scheduledFire` dequeues.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Unsubscribe clears the queue.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source notification → `ε` at arrival.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``scheduledFire →` the queued letter.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Unlike `delay`, error is also scheduled.`
9. Source edge 2: `Delay 0 still leaves the current stack if the scheduler is async.`

## Sequence from the analysis

`of(1, 2).pipe(observeOn(asyncScheduler))` writes `next(1) next(2) complete` on a later turn, not inside the synchronous subscribe.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
