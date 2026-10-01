# Test plan: `debounce`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/debounce.md](../../operators/filtering/debounce.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/debounce.ts` |
| Signature | `debounce(durationSelector: (value: T) => ObservableInput<any>): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Each source next unsubscribes the previous duration and subscribes to `durationSelector(value)`, storing the value. When that duration emits, emit the stored value. A newer source next cancels the previous duration. Source complete emits the pending value if any, then completes.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, pending(v, duration), stopped }`.

Memory at subscribe: `idle`.

Events: `{ next(v), error, complete, durationNext, durationError, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` replaces pending duration → `pending(v, newDuration)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``durationNext → idle`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``durationNext → next(v)` for the value that armed this duration.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(pending) · complete` if pending, else `complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Duration selector sees the latest value.`
10. Source edge 2: `Complete flushes.`

## Sequence from the analysis

Values 1, 2, 3 each restarting the duration, then a quiet duration next, writes `next(3)` only.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
