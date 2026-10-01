# Test plan: `takeWhile`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/takeWhile.md](../../operators/filtering/takeWhile.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/takeWhile.ts` |
| Signature | `takeWhile(predicate, inclusive = false): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Emit while `predicate(value, index)` is true. The first false value completes the output. If `inclusive`, that failing value is emitted before complete. Unsubscribes when the predicate fails.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { taking(i), stopped }`.

Memory at subscribe: `taking(0)`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Predicate true → `taking(i+1)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Predicate false → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Predicate true → `next(v)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Predicate false → `complete`, or `next(v) · complete` if inclusive.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Index increments per source next while taking.`
9. Source edge 2: `Inclusive flag is a parameter of `G`, not an extra state.`

## Sequence from the analysis

`takeWhile(x => x < 3)` on `1 2 3 4` writes `next(1) next(2) complete`. Inclusive also writes `next(3)` before complete.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
