# Test plan: `reduce`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/mathematical/reduce.md](../../operators/mathematical/reduce.md).

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/reduce.ts` |
| Signature | `reduce(accumulator, seed?): OperatorFunction<T, R>` |
| Status | Stable. |

## Behavior under test

Fold the source. Emit the final accumulator on complete only. With a seed, empty source emits the seed. Without a seed, empty source errors `EmptyError`, and the first value becomes the initial acc without calling the accumulator.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { needSeed, holding(acc, i), stopped }`.

Memory at subscribe: `holding(seed, 0)` if seed given, else `needSeed`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(acc), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``needSeed × next(v) → holding(v, 1)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``holding × next → holding(accumulator(acc, v, i), i+1)`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` in `holding → next(acc) · complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` in `needSeed → error(EmptyError)`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Accumulator throw → `error`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Seed is emitted on empty complete.`
11. Source edge 2: `No seed plus one value emits that value on complete without calling the accumulator.`

## Sequence from the analysis

`of(1, 2, 3).pipe(reduce((a, b) => a + b, 0))` writes `next(6) complete` and nothing earlier. `scan` would have written the intermediates.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
