# Test plan: `pairwise`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/pairwise.md](../../operators/transformation/pairwise.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/pairwise.ts` |
| Signature | `pairwise(): OperatorFunction<T, [T, T]>` |
| Status | Stable. |

## Behavior under test

Remember the previous value. The first next only stores. From the second next on, emit `[previous, current]` and shift memory. Complete and error pass through. A single-value source completes with no emission.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { empty, holding(prev), stopped }`.

Memory at subscribe: `empty`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next([T, T]), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``empty × next(v) → holding(v)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``holding(p) × next(v) → holding(v)`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``empty × next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``holding(p) × next(v) → next([p, v])`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → complete`, `error → error`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `No emission on complete for a dangling first value.`
10. Source edge 2: `Pairs overlap by one.`

## Sequence from the analysis

`of(1, 2, 3).pipe(pairwise())` writes `next([1, 2]) next([2, 3]) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
