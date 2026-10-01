# Test plan: `from`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/from.md](../../operators/creation/from.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold conversion function |
| RxJS 7.x source | `src/internal/observable/from.ts` |
| Signature | `from(input: ObservableInput<T>, scheduler?: SchedulerLike): Observable<T>` |
| Status | Stable. |

## Behavior under test

`from` adapts an `ObservableInput` into an Observable. Arrays and array-likes are enumerated; iterables are pulled; promises become a single next+complete or an error; an existing Observable is subscribed and forwarded; async iterables are pulled until return. The scheduler, if any, spreads synchronous emissions.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, pulling(cursor), awaiting(promise | asyncIterator), forwarding, stopped }`. The cursor is the iteration state. Promise and async-iterator waits are distinct only in how the next input is produced.

Memory at subscribe: `idle`.

Events: `{ subscribe, yield(v), yieldDone, reject(e), innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next(v), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → pulling(0)` for array/iterable, `awaiting` for promise/async iterable, `forwarding` for an Observable, `stopped` if conversion throws.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``pulling(i) × yield(v) → pulling(i+1)`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``pulling × yieldDone → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``awaiting × yield(v) → awaiting` (async iterator) or `stopped` (promise success is terminal).`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``awaiting × reject(e) → stopped`.`
7. Transition 6: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``forwarding` mirrors the inner terminal transitions.`
8. Transition 7: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped` (async iterator `return()` is attempted).`
9. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``pulling × yield(v) → next(v)`.`
10. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``pulling × yieldDone → complete`.`
11. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``awaiting × yield(v) → next(v)` and, for a promise, also `· complete`.`
12. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``reject(e) → error(e)`.`
13. Output 5: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Observable input: identity forward.`
14. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
15. Source edge 1: `A string is an iterable of characters.`
16. Source edge 2: `Readable-stream style async iterables must be closed on unsubscribe.`
17. Source edge 3: `Scheduler does not change the word, only when its letters are delivered.`

## Sequence from the analysis

`from([10, 20])`: subscribe, `yield(10)`, `yield(20)`, `yieldDone` writes `next(10) next(20) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
