# Test plan: `groupBy`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/groupBy.md](../../operators/transformation/groupBy.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/groupBy.ts` |
| Signature | `groupBy(keySelector, elementSelector?, durationSelector?, connector?): OperatorFunction<T, GroupedObservable<K, R>>` |
| Status | Stable. |

## Behavior under test

Map each source value to a key. Emit a `GroupedObservable` the first time a key is seen. Later values for that key are nexted into the subject's group. `durationSelector`, if present, closes a group when its notifier emits. Source complete completes every open group and then the outer. Each group is its own small machine.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { map key → { subject, open }, stopped }`. Infinite key space possible.

Memory at subscribe: Empty map.

Events: `{ next(v), error, complete, durationNext(k), durationError(k), unsubscribe }` plus group-subscriber subscribe/unsubscribe (refcounts on the connector subject).

Possible notifications: `{ next(GroupedObservable), error, complete }` on the outer, and `{ next(element), error, complete }` on each group.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` computes key. Missing key adds a group. Existing open key stays.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``durationNext(k)` marks that group closed so a later value with the same key opens a new group.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete closes all groups and stops.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source error errors all groups and stops.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(group$)` if the key is new, else `ε` on the outer. The group machine writes `next(elementSelector(v))`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete →` complete each group, then outer `complete`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error →` error each group and the outer.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Late subscribers to a group see only what the connector subject replays (`Subject` by default, so nothing already past).`
11. Source edge 2: `Duration complete also closes the group.`

## Sequence from the analysis

Values `{id:1}`, `{id:1}`, `{id:2}` emit two group observables. The first group writes two elements; the second writes one.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
