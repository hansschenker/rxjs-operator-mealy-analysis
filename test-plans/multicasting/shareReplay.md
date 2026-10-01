# Test plan: `shareReplay`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/shareReplay.md](../../operators/multicasting/shareReplay.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable replay multicast |
| RxJS 7.x source | `src/internal/operators/shareReplay.ts` |
| Signature | `shareReplay(configOrBufferSize?, windowTime?, scheduler?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

`share` with a ReplaySubject connector. On 7.x the implementation sets `resetOnError: true`, `resetOnComplete: false`, and `resetOnRefCountZero` from the `refCount` option (default false). So the replay buffer survives completion and, by default, survives the last subscriber leaving. New subscribers receive the buffered window.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ cold, hot(buffer, refCount), stopped }` with a ReplaySubject buffer.

Memory at subscribe: `cold`, empty buffer.

Events: `{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete, resetTick }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `First join connects and subscribes to the source.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next appends to the replay buffer.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete does not reset (default).`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source error resets because resetOnError is true.`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Leave at refCount 0 does not unsubscribe the source unless refCount: true.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Join → next for each buffered value, then live values.`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → next to current subscribers and a buffer append.`
9. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source complete → complete, and late join still replays then completes.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Default refCount false keeps the source subscribed.`
12. Source edge 2: `Config object form accepts bufferSize, windowTime, refCount, scheduler.`
13. Source edge 3: `Implemented via `share`.`

## Sequence from the analysis

First subscriber sees 1, 2 and unsubscribes. A later subscriber still receives 1, 2 from the buffer if refCount is false and the source already produced them.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
