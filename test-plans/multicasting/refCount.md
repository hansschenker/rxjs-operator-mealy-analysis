# Test plan: `refCount`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/refCount.md](../../operators/multicasting/refCount.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Connectable adapter |
| RxJS 7.x source | `src/internal/operators/refCount.ts` |
| Signature | `refCount(): OperatorFunction<T, T>` |
| Status | Deprecated. Use `share`. |

## Behavior under test

For a ConnectableObservable, the first subscriber calls `connect()`, further subscribers increment a count, and the last unsubscribe disconnects. Late subscribers do not replay unless the underlying subject does.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ refCount, connection | ⊥, stopped }`.

Memory at subscribe: refCount 0, no connection.

Events: `{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Join at 0 → connect, refCount 1.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Join increments.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Leave decrements; at 0 disconnect.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source terminal stops the subject.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → next to current subscribers.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Join writes ε unless the subject replays.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Deprecated in favor of `share`.`
10. Source edge 2: `Disconnect does not reset a sticky subject error by itself.`

## Sequence from the analysis

Two subscribers share one connection. When both leave, the source is unsubscribed.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
