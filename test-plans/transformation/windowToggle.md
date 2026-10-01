# Test plan: `windowToggle`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/windowToggle.md](../../operators/transformation/windowToggle.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowToggle.ts` |
| Signature | `windowToggle(openings, closingSelector): OperatorFunction<T, Observable<T>>` |
| Status | Stable. |

## Behavior under test

Window analogue of `bufferToggle`. Each opening emits a new window observable and subscribes to a closer. Source values go to every open window. Closer next completes that window.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { list of open windows, stopped }`.

Memory at subscribe: No window until the first opening.

Events: `{ next(v), error, complete, opening, closing_i, unsubscribe }`.

Possible notifications: Outer window observables; inner notifications.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``opening` appends a window.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``closing_i` removes it.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` does not change membership.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``opening → next(window$)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) →` inner next on each open window.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``closing_i →` that window `complete`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source complete completes open windows and the outer.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Values with no open window are dropped.`
11. Source edge 2: `Closing complete without a value does not emit a boundary by itself in the same way a next does; it unsubscribes the closer.`

## Sequence from the analysis

One opening, two values, one closing: one window observable emits both values and completes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
