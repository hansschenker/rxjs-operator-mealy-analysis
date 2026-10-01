# Test plan: `expand`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/expand.md](../../operators/transformation/expand.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable recursive higher-order operator |
| RxJS 7.x source | `src/internal/operators/expand.ts` |
| Signature | `expand(project, concurrent = Infinity, scheduler?): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Emit the source value, and also subscribe to `project(value)` whose emissions are emitted and recursively expanded. `concurrent` bounds active inners. It is `mergeMap` with feedback of outputs into the project function. No implicit complete until the source and every recursive inner complete.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active count, queue of values still to expand, outerDone, stopped }`.

Memory at subscribe: Active 0, empty queue.

Events: `{ outerNext(v), outerError, outerComplete, innerNext(v), innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` and `innerNext` enqueue an expansion and emit.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Active expansions are started up to `concurrent`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Idle and outer done and empty queue → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext(v) → next(v)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext(v) → next(v)`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Completion word only when nothing remains to expand.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: ``concurrent: 1` serializes recursion and can change interleaving, not the set of values for a pure project.`
10. Source edge 2: `Scheduler shifts recursive subscribes.`

## Sequence from the analysis

`of(1).pipe(expand(x => x < 3 ? of(x+1) : EMPTY))` writes `next(1) next(2) next(3) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
