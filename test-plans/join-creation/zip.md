# Test plan: `zip`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/zip.md](../../operators/join-creation/zip.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/zip.ts` |
| Signature | `zip(...sources, resultSelector?): Observable<T[]>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Pair values by index. Each source has a queue. When every queue is non-empty, shift one value from each and emit the tuple. Complete when any source completes and its queue cannot form another tuple. Error on any error.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = per-source queue (sequence of V) plus done flags, or `stopped`. Queues make `S` infinite.

Memory at subscribe: Empty queues, no source done.

Events: `{ subscribe, next_i(v), error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next_i(v)` appends to queue i. If all queues are non-empty, shift one from each.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``complete_i` marks done. If queue i is empty and done, → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``error_i → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next_i → next(tuple)` if the append filled the last empty queue, else `ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete_i → complete` if no further tuple can be formed, else `ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error_i(e) → error(e)`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Unlike `combineLatest`, values are consumed, not reused.`
10. Source edge 2: `A result selector projects the tuple inside `G` only.`

## Sequence from the analysis

`zip(of(1, 2), of('a'))` writes `next([1, 'a']) complete`. The `2` stays queued and is dropped when the shorter source completes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
