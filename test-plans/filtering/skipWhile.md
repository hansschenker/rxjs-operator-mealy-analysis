# Test plan: `skipWhile`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/skipWhile.md](../../operators/filtering/skipWhile.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipWhile.ts` |
| Signature | `skipWhile(predicate: (value, index) => boolean): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Drop values while `predicate` is true. The first value that fails the predicate, and everything after it, is emitted. The predicate is not consulted again after that.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { skipping(i), forwarding, stopped }`.

Memory at subscribe: `skipping(0)`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``skipping × next → skipping(i+1)` if predicate true.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``skipping × next → forwarding` if predicate false.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Skipped next → `ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `The failing next and later nexts → `next(v)`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Index increments only while skipping.`
9. Source edge 2: `Once forwarding, predicate throws cannot happen because it is not called.`

## Sequence from the analysis

`skipWhile(x => x < 3)` on `1 2 3 1` writes `next(3) next(1) complete`. The later 1 passes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
