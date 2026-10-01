# Test plan: `timeInterval`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/timeInterval.md](../../operators/utility/timeInterval.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeInterval.ts` |
| Signature | `timeInterval(scheduler = asyncScheduler): OperatorFunction<T, TimeInterval<T>>` |
| Status | Stable. |

## Behavior under test

Emit `{ value, interval }` where `interval` is scheduler time since the previous emission, or since subscription for the first value.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active(lastTime), stopped }`.

Memory at subscribe: `active(scheduler.now()` at subscribe`)`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(TimeInterval), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` replaces `lastTime` with now.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next({ value: v, interval: now - lastTime })`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Error and complete copy through.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `Uses the scheduler clock, not `Date.now` directly, when a scheduler is passed.`
8. Source edge 2: `Complete is not wrapped.`

## Sequence from the analysis

Two values one second apart write intervals near 0 (or subscribe delay) and near 1000.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
