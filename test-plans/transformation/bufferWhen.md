# Test plan: `bufferWhen`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/bufferWhen.md](../../operators/transformation/bufferWhen.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferWhen.ts` |
| Signature | `bufferWhen(closingSelector: () => Observable<any>): OperatorFunction<T, T[]>` |
| Status | Stable. |

## Behavior under test

One buffer is open. Subscribe to `closingSelector()` immediately. When that notifier emits, emit the buffer, open a new one, and subscribe to a fresh closer. Notifier complete also closes. Source complete emits the open buffer and completes.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { open(buf), stopped }`.

Memory at subscribe: `open([])` with the first closer subscribed.

Events: `{ next(v), error, complete, closeNext, closeError, closeComplete, unsubscribe }`.

Possible notifications: `{ next(T[]), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v) → open(buf ++ [v])`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``closeNext` or `closeComplete → open([])` with a new closer, unless the source has already ended.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete or any error → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``closeNext` / `closeComplete → next(buf)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(buf) · complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``closingSelector` is invoked per buffer, not once.`
11. Source edge 2: `A throw from the selector is an error output.`

## Sequence from the analysis

A closer that emits twice produces two arrays covering the values between those signals, then arms a third buffer.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
