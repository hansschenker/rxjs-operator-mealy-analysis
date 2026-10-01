# Test plan: `bufferTime`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/bufferTime.md](../../operators/transformation/bufferTime.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferTime.ts` |
| Signature | `bufferTime(bufferTimeSpan, bufferCreationInterval?, maxBufferSize?, scheduler?): OperatorFunction<T, T[]>` |
| Status | Stable. |

## Behavior under test

Open a buffer and emit it when `bufferTimeSpan` elapses. Optional `bufferCreationInterval` opens additional buffers on a cadence (overlapping windows). Optional `maxBufferSize` closes a buffer early when it fills. Scheduler defaults to async.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { open buffers with their close times, stopped }`.

Memory at subscribe: One open buffer armed to close after `bufferTimeSpan`.

Events: `{ next(v), error, complete, spanTick(id), creationTick, unsubscribe }`.

Possible notifications: `{ next(T[]), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` appends to open buffers; a buffer at `maxBufferSize` closes.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``spanTick(id)` closes that buffer and, if no creation interval, opens a successor.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``creationTick` opens another buffer.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``spanTick` and max-size close → `next(buf)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete →` emit open buffers then `complete`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next` itself → `ε` unless it hit `maxBufferSize`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Empty time spans still emit `[]`.`
11. Source edge 2: `Creation interval plus span is the overlapping form.`

## Sequence from the analysis

`bufferTime(1000)` over values inside one second emits one array per second, including empty arrays when a span had no values.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
