# Test plan: `publishReplay`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/publishReplay.md](../../operators/multicasting/publishReplay.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishReplay.ts` |
| Signature | `publishReplay(bufferSize?, windowTime?, selector?, scheduler?): OperatorFunction<T, T>` |
| Status | Deprecated. ReplaySubject-backed multicast. |

## Behavior under test

ConnectableObservable over a `ReplaySubject(bufferSize, windowTime)`. Subscribers receive the buffered window on join, then live values after connect.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { disconnected(buffer), connected(buffer), stopped }`. Buffer is a bounded queue.

Memory at subscribe: `disconnected(empty buffer)`.

Events: `multicast` alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next appends to the replay buffer while connected.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Join does not change connection.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscriberJoin → next` for each buffered value still inside windowTime.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``sourceNext → next` to live subscribers and a buffer append.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `Does not refCount unless composed with `refCount`.`
8. Source edge 2: `Deprecated in favor of `share({ connector: () => new ReplaySubject(...) })`.`

## Sequence from the analysis

Connect, emit 1 then 2, late subscriber joins: late subscriber gets `next(1) next(2)` from the buffer if bufferSize allows.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
