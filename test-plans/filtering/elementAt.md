# Test plan: `elementAt`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/elementAt.md](../../operators/filtering/elementAt.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/elementAt.ts` |
| Signature | `elementAt(index: number, defaultValue?): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Emit the value at the given zero-based index and complete. If the source completes before that index, emit `defaultValue` if supplied, otherwise error with `ArgumentOutOfRangeError`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { counting(i), stopped }` for `0 ≤ i ≤ index`.

Memory at subscribe: `counting(0)`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``counting(i) × next → counting(i+1)` if `i < index`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``counting(index) × next → stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Early complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``counting(index) × next(v) → next(v) · complete`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Earlier next → `ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Early complete → `next(default) · complete` if default given, else `error(ArgumentOutOfRangeError)`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Negative index errors.`
10. Source edge 2: `Unsubscribes after the match so later source values are not pulled.`

## Sequence from the analysis

`elementAt(1)` on `a b c` writes `next(b) complete` and unsubscribes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
