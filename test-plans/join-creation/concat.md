# Test plan: `concat`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/concat.md](../../operators/join-creation/concat.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/concat.ts` |
| Signature | `concat(...sources): Observable<T>` |
| Status | Stable. Pipeable cousin is `concatWith` / `concatAll`. |

## Behavior under test

Subscribe to the first source and forward it until it completes, then subscribe to the next, and so on. One active inner. An error from any source errors the output and later sources are not subscribed. Complete after the last source completes.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { reading(i) | 0 ≤ i ≤ n } ∪ { stopped }`. `reading(i)` means source `i` is the active subscription.

Memory at subscribe: `reading(0)` after subscribe; `n = 0` (no sources) is already done.

Events: `{ subscribe, innerNext(v), innerError(e), innerComplete, unsubscribe }`.

Possible notifications: `{ next(v), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``reading(i) × innerNext → reading(i)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``reading(i) × innerComplete → reading(i+1)` if `i+1 < n`, else `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``reading(i) × innerError → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext(v) → next(v)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerError(e) → error(e)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → ε` if another source remains; `complete` if it was the last.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Sources are subscribed lazily, not up front.`
11. Source edge 2: `Promises and arrays are normalized via `from` when they are reached.`

## Sequence from the analysis

`concat(of(1,2), of(3))` writes `next(1) next(2) next(3) complete`. The `3` cannot appear before the first source completes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
