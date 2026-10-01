# Test plan: `findIndex`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/conditional/findIndex.md](../../operators/conditional/findIndex.md).

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/findIndex.ts` |
| Signature | `findIndex(predicate, thisArg?): OperatorFunction<T, number>` |
| Status | Stable. |

## Behavior under test

Emit the index of the first match and complete. If none, emit `-1` and complete. Unsubscribes after a match.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { searching(i), stopped }`.

Memory at subscribe: `searching(0)`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(number), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Non-match → `searching(i+1)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Match or complete → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Match → `next(i) · complete`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete with no match → `next(-1) · complete`.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Non-match → `ε`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Index is zero-based.`
9. Source edge 2: `Does not error on a miss.`

## Sequence from the analysis

`findIndex(x => x === 3)` on `1 2 3` writes `next(2) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
