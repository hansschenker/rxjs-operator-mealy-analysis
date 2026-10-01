# Test plan: `skipLast`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/skipLast.md](../../operators/filtering/skipLast.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipLast.ts` |
| Signature | `skipLast(count: number): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Hold a ring buffer of `count` values. Once the buffer is full, each new value emits the oldest and pushes the new one. The last `count` values are never emitted. Complete does not flush them.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { buffer (queue of length ≤ count), stopped }`.

Memory at subscribe: Empty queue.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` enqueues. If length would exceed count, dequeue.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped` without flushing.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(oldest)` if the buffer was already full, else `ε`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → complete`.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: ``count <= 0` forwards everything.`
8. Source edge 2: `Must observe complete or unsubscribe to know the tail; it cannot emit the tail earlier.`

## Sequence from the analysis

`skipLast(2)` on `1 2 3 4 5` writes `next(1) next(2) next(3) complete`. 4 and 5 stay buffered.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
