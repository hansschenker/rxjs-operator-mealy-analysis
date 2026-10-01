# Test plan: `bufferCount`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/bufferCount.md](../../operators/transformation/bufferCount.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferCount.ts` |
| Signature | `bufferCount(bufferSize: number, startBufferEvery: number = bufferSize): OperatorFunction<T, T[]>` |
| Status | Stable. |

## Behavior under test

Maintain a list of open buffers. Every `startBufferEvery` source values, open a new buffer. A buffer emits and closes when it reaches `bufferSize`. Default `startBufferEvery = bufferSize` is non-overlapping. On complete, emit every non-empty open buffer then complete.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { buffers: T[][], count since last open, stopped }`.

Memory at subscribe: One empty buffer, count 0, unless `bufferSize < 1` which errors.

Events: `{ next(v), error(e), complete, unsubscribe }`.

Possible notifications: `{ next(T[]), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` appends `v` to every open buffer, may close full buffers, and may open a new buffer when `count` hits `startBufferEvery`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal inputs → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) →` the concatenation of `next(buf)` for each buffer that just reached `bufferSize`, else `ε`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(buf)` for each non-empty open buffer, then `complete`.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error(e) → error(e)`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: ``bufferSize < 1` errors on subscribe.`
9. Source edge 2: ``startBufferEvery` smaller than `bufferSize` creates overlapping buffers.`

## Sequence from the analysis

`bufferCount(3, 1)` on `a b c d` emits `[a,b,c]`, then `[b,c,d]`, and on complete the trailing partials `[c,d]` and `[d]`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
