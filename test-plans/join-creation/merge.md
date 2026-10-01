# Test plan: `merge`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/merge.md](../../operators/join-creation/merge.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/merge.ts` |
| Signature | `merge(...sources, concurrent?: number): Observable<T>` |
| Status | Stable. Pipeable form is `mergeWith` / `mergeAll`. |

## Behavior under test

Subscribe to sources, up to `concurrent` at a time (default Infinity), and forward their nexts as they arrive. Complete when every source has completed. Error on the first error. Extra sources wait in a queue when concurrency is bounded.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active set, queue of not-yet-subscribed sources, completed count } ∪ { stopped }`.

Memory at subscribe: Empty active set, full queue, completed count 0.

Events: `{ subscribe, innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next(v), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Subscribe fills `active` up to the concurrency limit.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerNext` does not change membership.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` removes that inner, increments completed, pulls from the queue if any.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `All sources completed → `stopped`.`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerError → stopped`.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext(v) → next(v)`.`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → ε`, or `complete` when the completed count reaches the source count.`
9. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerError(e) → error(e)`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `A numeric trailing argument is concurrency, not a source.`
12. Source edge 2: ``concurrent: 1` is sequential and matches `concat` order.`

## Sequence from the analysis

`merge(timer(2).pipe(mapTo('late')), of('early'))` may write `next('early')` before `next('late')`. Order across sources is arrival order.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
