# Test plan: `takeLast`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/takeLast.md](../../operators/filtering/takeLast.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/takeLast.ts` |
| Signature | `takeLast(count: number): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Buffer at most `count` values. On source complete, emit the buffer in order and complete. Error passes through without emitting the buffer.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { buffer (ring of size ≤ count), stopped }`.

Memory at subscribe: Empty buffer.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next` pushes, dropping the oldest if over count.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete → stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``error → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(b1)…next(bn) · complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error` with no flush.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: ``count <= 0` completes with no values once the source completes.`
10. Source edge 2: `Must wait for complete, so it does not work on a non-completing source.`

## Sequence from the analysis

`takeLast(2)` on `1 2 3 4` writes `next(3) next(4) complete` after the source completes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
