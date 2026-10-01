# Test plan: `startWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/startWith.md](../../operators/join/startWith.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/startWith.ts` |
| Signature | `startWith(...values, scheduler?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

On subscribe, emit the given values (via `concat(of(...values), source)` semantics) and then subscribe to the source and forward it. Scheduler shifts the prefix.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { prefix(i), forwarding, stopped }`.

Memory at subscribe: `prefix(0)`.

Events: `{ subscribe, step, sourceNext, sourceError, sourceComplete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Prefix steps advance `i`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `After the last prefix value, enter `forwarding` and subscribe to source.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Prefix step → `next(values[i])`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source notifications copy through after the prefix.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Values are emitted even if the source never emits.`
9. Source edge 2: `A scheduler argument is not a prefix value.`

## Sequence from the analysis

`of(2, 3).pipe(startWith(1))` writes `next(1) next(2) next(3) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
