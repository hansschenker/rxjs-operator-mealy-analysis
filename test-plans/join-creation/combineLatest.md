# Test plan: `combineLatest`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/combineLatest.md](../../operators/join-creation/combineLatest.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/combineLatest.ts` |
| Signature | `combineLatest(sources, resultSelector?): Observable<T[]>` |
| Status | Stable as a creation function. The pipeable form on 7.x is `combineLatestWith` / `combineLatestAll`. `resultSelector` deprecated. |

## Behavior under test

Subscribe to every source. Remember the latest value of each. Emit an array (or projected tuple) only once every source has produced at least one value, and again whenever any source emits after that. Complete when every source has completed. Error if any source errors. A source that completes without a value prevents any emission and completes the output when all have settled without a full tuple.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = records of { latest: V | ⊥, done: bool } per source, plus `stopped`. Infinite because values are arbitrary. Control flag `ready = ∀ latest ≠ ⊥`.

Memory at subscribe: All `latest = ⊥`, all `done = false`.

Events: `{ subscribe, next_i(v), error_i(e), complete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next_i(v)` stores `latest_i = v`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete_i` sets `done_i`. If all done, → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``error_i → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped` and unsubscribe all.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next_i(v) → next(snapshot)` if every source has a latest, else `ε`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error_i(e) → error(e)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete_i → complete` if all sources are done, else `ε`. No extra next on complete.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Dictionary form uses keys instead of indexes; the tuple shape changes, the machine does not.`
11. Source edge 2: `Empty source list completes immediately.`
12. Source edge 3: `Completion of a source that already has a latest does not emit by itself.`

## Sequence from the analysis

Sources `A: 1, 2` and `B: a`. After `1` the word is `ε` (B missing). After `a` the word is `next([1, a])`. After `2` the word is `next([2, a])`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
