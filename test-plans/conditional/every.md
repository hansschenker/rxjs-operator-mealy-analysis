# Test plan: `every`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/conditional/every.md](../../operators/conditional/every.md).

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/every.ts` |
| Signature | `every(predicate, thisArg?): OperatorFunction<T, boolean>` |
| Status | Stable. |

## Behavior under test

Emit `false` and complete on the first value that fails the predicate. If the source completes and none failed, emit `true` and complete. Empty source emits `true` (vacuous truth). Predicate throw is an error.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { checking(i), stopped }`.

Memory at subscribe: `checking(0)`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next(boolean), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Passing next → `checking(i+1)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Failing next → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Failing next → `next(false) · complete`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Passing next → `ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete → `next(true) · complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Empty source is true.`
10. Source edge 2: `Unsubscribes on the first failure.`

## Sequence from the analysis

`every(x => x < 3)` on `1 2 3` writes `next(false) complete` at 3 and unsubscribes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
