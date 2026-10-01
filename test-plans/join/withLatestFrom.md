# Test plan: `withLatestFrom`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/withLatestFrom.md](../../operators/join/withLatestFrom.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/withLatestFrom.ts` |
| Signature | `withLatestFrom(...others, project?): OperatorFunction<T, R>` |
| Status | Stable. Project form deprecated in favor of a later `map`. |

## Behavior under test

Subscribe to the other sources and remember their latest values. Only a next from the main source emits, and only once every other has a latest. The output is `[main, ...latests]` or the projection. Others completing does not complete the output. Main complete completes the output.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { latest_i: V | ⊥, stopped }`.

Memory at subscribe: All others `⊥`.

Events: `{ mainNext(v), mainError, mainComplete, otherNext_i, otherError_i, otherComplete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``otherNext_i` stores latest.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``mainNext` does not store a lasting main value.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Main complete or any error → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``mainNext(v) → next([v, ...latests])` if every other has a latest, else `ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``otherNext → ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``mainComplete → complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Other complete without a value leaves the machine unable to emit.`
10. Source edge 2: `Other error fails the output.`

## Sequence from the analysis

Main emits before the other has emitted: `ε`. After the other emits `a` and main emits `1`: `next([1, a])`. A later other value does not emit by itself.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
