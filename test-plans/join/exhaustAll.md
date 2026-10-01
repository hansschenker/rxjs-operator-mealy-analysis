# Test plan: `exhaustAll`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/exhaustAll.md](../../operators/join/exhaustAll.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/exhaustAll.ts` |
| Signature | `exhaustAll(): OperatorFunction<ObservableInput<T>, T>` |
| Status | Stable. |

## Behavior under test

Subscribe to an inner only if none is active. Inners that arrive while busy are dropped, not queued. `exhaust` is this function.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, busy, outerDone, stopped }`.

Memory at subscribe: `idle`.

Events: Higher-order alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × outerNext → busy`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``busy × outerNext → busy` (dropped).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete → idle` or `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete when outer is done and idle.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Dropped inners are not subscribed.`
10. Source edge 2: `See `exhaust.ts`, which re-exports this.`

## Sequence from the analysis

A second inner emitted before the first completes never gets a subscription.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
