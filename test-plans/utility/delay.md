# Test plan: `delay`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/delay.md](../../operators/utility/delay.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/delay.ts` |
| Signature | `delay(due: number |
| Status | Stable. |

## Behavior under test

Shift next notifications by `due`. Complete is delayed so it stays after delayed nexts. Error is not delayed. Each next is a scheduled action; state is the queue of pending actions.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { pending queue of scheduled nexts, completeArmed, stopped }`.

Memory at subscribe: Empty queue.

Events: `{ next(v), error, complete, dueTick(v), completeTick, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` enqueues a due tick.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``dueTick` removes that item.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete` arms a complete tick after pending nexts.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``error → stopped` immediately.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε` (schedule action).`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``dueTick(v) → next(v)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error` now.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``completeTick → complete`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Date due is absolute.`
12. Source edge 2: `Unsubscribe cancels pending ticks.`

## Sequence from the analysis

`of(1).pipe(delay(1000))` writes nothing at subscribe time and `next(1) complete` about a second later.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
