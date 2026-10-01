# Test plan: `forkJoin`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/forkJoin.md](../../operators/join-creation/forkJoin.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/forkJoin.ts` |
| Signature | `forkJoin(sources): Observable<T[] |
| Status | Stable. |

## Behavior under test

Subscribe to all sources. Keep only the last value of each. Emit once, when every source has completed, the array or dictionary of last values, then complete. If any source errors, error and unsubscribe the rest. If any source completes without a value, complete without emitting.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = per-source { last: V | ⊥, done: bool } plus `stopped`.

Memory at subscribe: All `last = ⊥`, `done = false`.

Events: `{ subscribe, next_i(v), error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next(lasts), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next_i(v)` overwrites `last_i`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete_i` sets `done_i`. All done → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``error_i → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next_i → ε` always (forkJoin does not emit on next).`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete_i → next(lasts) · complete` if every source is done and every source has a last value.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete_i → complete` if every source is done but some last is ⊥.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error_i(e) → error(e)`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Empty argument list completes without a next.`
11. Source edge 2: `Dictionary keys are preserved.`
12. Source edge 3: `Unlike `combineLatest`, intermediate snapshots are not outputs.`

## Sequence from the analysis

`forkJoin([of(1, 2), of('a')])` writes a single `next([2, 'a']) complete` after both complete. The `1` is overwritten and never emitted.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
