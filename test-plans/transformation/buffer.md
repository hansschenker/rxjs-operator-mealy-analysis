# Test plan: `buffer`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/buffer.md](../../operators/transformation/buffer.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/buffer.ts` |
| Signature | `buffer(closingNotifier: Observable<any>): OperatorFunction<T, T[]>` |
| Status | Stable. |

## Behavior under test

Collect source values into an array. Each time `closingNotifier` emits, emit the current array and start a fresh one. Notifier error errors the output. Source error errors the output. Source complete emits the open buffer then completes. Notifier complete emits the open buffer and completes.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { open(buf), stopped }`. `buf` is the array collected since the last close.

Memory at subscribe: `open([])`.

Events: `{ next(v), error(e), complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

Possible notifications: `{ next(T[]), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``open(buf) × next(v) → open(buf ++ [v])`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``open(buf) × notifierNext → open([])`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``open × (complete | notifierComplete | error | notifierError | unsubscribe) → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → ε` (value is stored, not forwarded).`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierNext → next(buf)` using the buffer from before the reset.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` or `notifierComplete → next(buf) · complete`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error(e)` or `notifierError(e) → error(e)`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `A notifier next with an empty buffer still emits `[]`.`
11. Source edge 2: `Values are not shared across buffers; the array is replaced.`

## Sequence from the analysis

Source `1, 2`, notifier click, source `3`, source complete writes `next([1, 2]) next([3]) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
