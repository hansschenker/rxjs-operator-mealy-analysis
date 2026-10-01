# Test plan: `publishBehavior`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/publishBehavior.md](../../operators/multicasting/publishBehavior.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishBehavior.ts` |
| Signature | `publishBehavior(value: T): OperatorFunction<T, T>` |
| Status | Deprecated. BehaviorSubject-backed multicast. |

## Behavior under test

Like `publish`, but the subject is a `BehaviorSubject(value)`. Every new subscriber synchronously receives the current value, which starts as `value` even before connect.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { disconnected(current), connected(current), stopped }`.

Memory at subscribe: `disconnected(initialValue)`.

Events: `multicast` alphabet plus the behavior current value.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``subscriberJoin` does not change connection, but current is readable.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``sourceNext(v)` sets current to v if connected.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscriberJoin → next(current)` synchronously.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``sourceNext(v) → next(v)` to all current subscribers.`
6. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
7. Source edge 1: `Initial value is emitted even if the source never emits.`
8. Source edge 2: `Deprecated.`

## Sequence from the analysis

`publishBehavior(0)` subscriber before connect still gets `next(0)`. After connect and source `1`, subscribers get `next(1)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
