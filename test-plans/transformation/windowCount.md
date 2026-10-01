# Test plan: `windowCount`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/windowCount.md](../../operators/transformation/windowCount.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowCount.ts` |
| Signature | `windowCount(windowSize: number, startWindowEvery: number = windowSize): OperatorFunction<T, Observable<T>>` |
| Status | Stable. |

## Behavior under test

Window analogue of `bufferCount`. Open windows of `windowSize` source values, starting a new window every `startWindowEvery` values. Emit each window observable when it opens. Complete a window when it reaches `windowSize`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { list of open windows with their counts, source count, stopped }`.

Memory at subscribe: First window emitted, count 0.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: Outer window observables; inner next/complete.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` increments counts, may complete full windows, may open a new window.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete completes open windows and stops.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `On open: outer `next(window$)`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `On source next: `next(v)` into each open window.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `On full: that window `complete`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: ``startWindowEvery < windowSize` overlaps.`
9. Source edge 2: ``windowSize < 1` errors.`

## Sequence from the analysis

`windowCount(2)` on `a b c d` emits two windows: `[a b]` and `[c d]`, each completed.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
