# Test plan: `combineLatestWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/combineLatestWith.md](../../operators/join/combineLatestWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/combineLatestWith.ts` |
| Signature | `combineLatestWith(...otherSources): OperatorFunction<T, [T, ...A]>` |
| Status | Stable. Pipeable replacement for the old `combineLatest` operator signature. |

## Behavior under test

Subscribe to the source and to each other source. Remember the latest of each. Emit a tuple only after every participant has a value, then on every subsequent next from any of them. Complete when all complete. Error if any errors.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Per-source `{ latest: V | ⊥, done }` plus stopped. Index 0 is the piped source.

Memory at subscribe: All latest `⊥`, none done.

Events: `{ next_i, error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next_i` stores latest_i.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `All done → stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Any error → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next_i → next(snapshot)` if every latest is present, else `ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete_i → complete` only when all are done.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error_i → error`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Equivalent to `combineLatest([source, ...others])` after subscription.`
10. Source edge 2: `Does not emit on complete by itself.`

## Sequence from the analysis

Source emits 1 before the other emits: `ε`. Other emits `a`: `next([1, a])`. Source emits 2: `next([2, a])`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
