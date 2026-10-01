# Test plan: `windowTime`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/windowTime.md](../../operators/transformation/windowTime.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowTime.ts` |
| Signature | `windowTime(windowTimeSpan, windowCreationInterval?, maxWindowSize?, scheduler?): OperatorFunction<T, Observable<T>>` |
| Status | Stable. |

## Behavior under test

Window analogue of `bufferTime`. Span timers close windows; optional creation interval opens extra windows; optional max size closes early.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { open windows with timers, stopped }`.

Memory at subscribe: First window open and emitted.

Events: `{ next(v), error, complete, spanTick, creationTick, unsubscribe }`.

Possible notifications: Window observables and their notifications.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Ticks close or open windows.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` forwards into open windows and may hit max size.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Open action → outer `next(window$)`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → inner nexts.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Span tick → inner complete, and maybe a new outer next.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Empty windows are still emitted and completed.`
9. Source edge 2: `Scheduler owns the ticks.`

## Sequence from the analysis

`windowTime(1000)` emits a new window observable every second and completes the previous one, even if it saw no values.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
