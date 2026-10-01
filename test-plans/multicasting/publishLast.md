# Test plan: `publishLast`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/publishLast.md](../../operators/multicasting/publishLast.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishLast.ts` |
| Signature | `publishLast(): OperatorFunction<T, T>` |
| Status | Deprecated. AsyncSubject-backed multicast. |

## Behavior under test

Subject is an `AsyncSubject`. It emits only the last source value, and only when the source completes, to current and late subscribers. Error is sticky.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { disconnected, connected(last | ⊥), stopped(last | error) }`.

Memory at subscribe: `disconnected`, no last.

Events: `multicast` alphabet.

Possible notifications: `{ next(last), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next overwrites last.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → stopped and subject emits.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Late join after stop still reads the AsyncSubject cache.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → `ε` to subscribers.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source complete → `next(last) · complete` if a last exists, else `complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Late `subscriberJoin` after stop replays that word.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `No value before complete.`
10. Source edge 2: `Deprecated.`

## Sequence from the analysis

Source `1, 2, complete` after connect writes `next(2) complete` to subscribers. The `1` is overwritten.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
