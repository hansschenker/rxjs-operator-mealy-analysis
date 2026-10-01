# Test plan: `bufferToggle`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/bufferToggle.md](../../operators/transformation/bufferToggle.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferToggle.ts` |
| Signature | `bufferToggle(openings, closingSelector): OperatorFunction<T, T[]>` |
| Status | Stable. |

## Behavior under test

`openings` emits start signals. Each opening subscribes to `closingSelector(openingValue)`. Source values are copied into every buffer opened and not yet closed. A closing emission emits that buffer and drops it.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { list of { buf, closingSub }, stopped }`.

Memory at subscribe: No open buffers.

Events: `{ next(v), error, complete, opening(o), openingError, closing_i, closingError_i, closingComplete_i, unsubscribe }`.

Possible notifications: `{ next(T[]), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``opening(o)` appends a new buffer and subscribes to its closer.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` appends to all open buffers.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``closing_i` removes buffer i.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Any error → `stopped`. Source complete → `stopped` (open buffers are not flushed by source complete in bufferToggle; they are discarded).`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Closing complete without a next does not emit; it just ends that closer.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``closing_i → next(buf_i)`.`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → ε`.`
9. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error → error`.`
10. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → complete` with no trailing buffers.`
11. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
12. Source edge 1: `Source complete does not emit partial buffers.`
13. Source edge 2: `Multiple openings can overlap.`

## Sequence from the analysis

Opening at value 1, values 1 2, closing, writes `next([1, 2])`. Values that arrive with no open buffer are dropped.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
