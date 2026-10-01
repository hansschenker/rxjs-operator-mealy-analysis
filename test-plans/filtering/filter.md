# Test plan: `filter`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/filter.md](../../operators/filtering/filter.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/filter.ts` |
| Signature | `filter(predicate: (value, index) => boolean, thisArg?): MonoTypeOperatorFunction<T>` |
| Status | Stable. `thisArg` deprecated. |

## Behavior under test

Emit source nexts for which `predicate(value, index)` is true. Index increments on every source next, including filtered-out values. Error and complete pass through. Predicate throw is an error.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active(i), stopped }`.

Memory at subscribe: `active(0)`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``active(i) × next → active(i+1)` if predicate returns.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Throw or terminal → `stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(v)` if predicate is true, else `ε`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Terminal inputs copy through.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `Index is the source index, not the count of passed values.`
8. Source edge 2: `Does not unsubscribe early.`

## Sequence from the analysis

`of(1, 2, 3, 4).pipe(filter(x => x % 2 === 0))` writes `next(2) next(4) complete`. Indexes seen by the predicate are 0, 1, 2, 3.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
