# Test plan: `windowWhen`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/windowWhen.md](../../operators/transformation/windowWhen.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowWhen.ts` |
| Signature | `windowWhen(closingSelector: () => Observable<any>): OperatorFunction<T, Observable<T>>` |
| Status | Stable. |

## Behavior under test

Window analogue of `bufferWhen`. A window is open from subscribe. When the closing observable emits, complete the window, emit a new one, and call `closingSelector` again.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { open window, stopped }`.

Memory at subscribe: First window emitted, closer subscribed.

Events: `{ next(v), error, complete, closeNext, closeError, unsubscribe }`.

Possible notifications: Window observables and inner notifications.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``closeNext` swaps the open window.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next keeps it.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe → next(window$)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) →` inner next.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``closeNext →` inner complete · outer `next(newWindow$)`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete →` inner complete · outer complete.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``closingSelector` runs per window.`
11. Source edge 2: `Selector throw is an error.`

## Sequence from the analysis

Two closer emissions produce three windows if the source is still active after the second close (the third is the newly opened one).

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
