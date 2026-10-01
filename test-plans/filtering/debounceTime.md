# Test plan: `debounceTime`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/debounceTime.md](../../operators/filtering/debounceTime.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/debounceTime.ts` |
| Signature | `debounceTime(dueTime, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Store the latest value and the time it arrived. One scheduled task is armed if none exists. When the task runs, if `now < lastTime + dueTime`, reschedule the remainder; otherwise emit `lastValue` and clear. Source complete flushes the pending value then completes. Source error does not flush. Unsubscribe clears `lastValue` and the task.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, holding(lastValue, lastTime, task | null), stopped }`.

Memory at subscribe: `idle` (`activeTask = null`, `lastValue = null`).

Events: `{ next(v), error, complete, taskFire, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` sets `lastValue`, `lastTime`, and arms `activeTask` only if it was null.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``taskFire` either stays holding and reschedules, or goes `idle` if the due time has elapsed.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete` or `error` or `unsubscribe → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``taskFire → ε` if rescheduled; `next(lastValue)` if due.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(lastValue) · complete` when a task is active, else `complete`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error(e) → error(e)` with no flush.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `One task, not a reset-and-replace per value.`
11. Source edge 2: `Complete flushes; error does not.`
12. Source edge 3: `Finalize nulls `lastValue` and `activeTask`.`

## Sequence from the analysis

`1` at t=0, `2` at t=40, dueTime 100. The single task fires, sees lastTime 40, reschedules, then writes `next(2)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
