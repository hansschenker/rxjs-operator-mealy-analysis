# Test plan: `sequenceEqual`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/conditional/sequenceEqual.md](../../operators/conditional/sequenceEqual.md).

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable comparison |
| RxJS 7.x source | `src/internal/operators/sequenceEqual.ts` |
| Signature | `sequenceEqual(compareTo, comparator?): OperatorFunction<T, boolean>` |
| Status | Stable. |

## Behavior under test

Compare the source to `compareTo` index by index, like zip plus an equality check. Emit `false` and complete on the first mismatch or if the lengths differ. Emit `true` and complete if both complete with equal paired values. Comparator defaults to `===`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Two queues plus done flags, or stopped. Buffers values that arrive before their pair.

Memory at subscribe: Empty queues, neither done.

Events: `{ next_source, next_other, error_either, complete_source, complete_other, unsubscribe }`.

Possible notifications: `{ next(boolean), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `A next appends to that side's queue. When both queues are non-empty, shift a pair.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `A length mismatch on complete → stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Both done with empty queues → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `A shifted pair that compares equal → ε.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `A shifted pair that compares unequal → next(false) · complete.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete that reveals a leftover value on the other side → next(false) · complete.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Both exhausted equally → next(true) · complete.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Uses a comparator, not deep equality.`
11. Source edge 2: `Errors from either side are forwarded.`

## Sequence from the analysis

`of(1, 2).pipe(sequenceEqual(of(1, 2)))` writes `next(true) complete`. Against `of(1, 3)` it writes `next(false) complete` at the second pair.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
