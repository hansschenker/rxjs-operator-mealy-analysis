# Test plan: `subscribeOn`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/subscribeOn.md](../../operators/utility/subscribeOn.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/subscribeOn.ts` |
| Signature | `subscribeOn(scheduler, delay = 0): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Schedule the act of subscribing to the source. After that, notifications are forwarded directly (no extra observe queue). Unsubscribe before the scheduled subscribe cancels it.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { scheduled, forwarding, stopped }`.

Memory at subscribe: `scheduled`.

Events: `{ subscribe, schedulerTick, sourceNext, sourceError, sourceComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``schedulerTick → forwarding` and subscribe to source.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source terminal → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Unsubscribe while scheduled → `stopped` without subscribing.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``schedulerTick → ε` (the subscribe action).`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source notifications copy through once forwarding.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Does not move notifications onto the scheduler; pair with `observeOn` for that.`
10. Source edge 2: `Delay delays the subscription, not each value.`

## Sequence from the analysis

A synchronous `of(1)` under `subscribeOn(asyncScheduler)` does not emit inside the caller stack; it emits when the scheduler runs the subscription.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
