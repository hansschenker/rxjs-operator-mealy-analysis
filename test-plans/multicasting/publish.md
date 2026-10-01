# Test plan: `publish`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/publish.md](../../operators/multicasting/publish.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publish.ts` |
| Signature | `publish(selector?): OperatorFunction<T, T>` |
| Status | Deprecated. `publish()` is `multicast(() => new Subject())`. |

## Behavior under test

ConnectableObservable backed by a plain Subject. No replay, no initial value. Subscribers see only nexts after they subscribe and after `connect()`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { disconnected, connected, stopped }`. Subject has no memory.

Memory at subscribe: `disconnected`.

Events: Same as `multicast`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Same connect/disconnect structure as `multicast` with a Subject factory.`
3. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next while connected → `next` to current subject subscribers only.`
4. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `No replay on subscriberJoin.`
5. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
6. Source edge 1: `Must call `connect()` or `refCount()`.`
7. Source edge 2: `Error on the subject is sticky for late subscribers.`

## Sequence from the analysis

Connect, source emits 1, late subscriber joins, source emits 2. Late subscriber sees only 2.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
