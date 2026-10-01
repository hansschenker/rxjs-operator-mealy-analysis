# Test plan: `multicast`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/multicast.md](../../operators/multicasting/multicast.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/multicast.ts` |
| Signature | `multicast(subjectOrFactory, selector?): OperatorFunction<T, R>` |
| Status | Deprecated on 7.x in favor of `share` / `connect`. Still in the operator directory. |

## Behavior under test

Share one subscription to the source through a Subject. Without a selector, the result is a `ConnectableObservable`: subscribers attach to the subject, and `connect()` subscribes the subject to the source. With a selector, the subject is wired for the duration of the selector's observable.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { disconnected, connected, stopped }`. Subject memory (replay or not) is a parameter of the subject, modeled as extra state if the subject is a ReplaySubject or BehaviorSubject.

Memory at subscribe: `disconnected`.

Events: `{ subscriberJoin, subscriberLeave, connect, sourceNext, sourceError, sourceComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }` toward subject subscribers.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``connect` in disconnected → `connected` and subscribe source to subject.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``sourceNext` stays connected.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source terminal → `stopped` (subject terminated).`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Selector form connects while the selected observable is subscribed.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``sourceNext →` subject `next` (fan-out is the subject's job).`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Before connect, source symbols are not produced.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Late subscribers see whatever the subject replays.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Factory form builds a fresh subject per connectable subscription.`
11. Source edge 2: `Passing a subject instance shares that subject.`
12. Source edge 3: `Deprecated; `share` covers refcounted use.`

## Sequence from the analysis

Two subscribers and one `connect()` cause one source subscription. Both receive later nexts. Without connect, neither receives source values.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
