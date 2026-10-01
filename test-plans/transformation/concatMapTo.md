# Test plan: `concatMapTo`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/concatMapTo.md](../../operators/transformation/concatMapTo.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/concatMapTo.ts` |
| Signature | `concatMapTo(innerObservable, resultSelector?): OperatorFunction<T, R>` |
| Status | Deprecated in 7.x. Use `concatMap(() => inner)`. |

## Behavior under test

Identical machine to `concatMap` except `project` ignores the outer value and returns the same `ObservableInput` each time. The input is still subscribed per outer value, not shared.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Same as `concatMap`: `{ active, queue, outerDone, stopped }`.

Memory at subscribe: Idle, empty queue.

Events: Same higher-order alphabet as `concatMap`.

Possible notifications: `{ next(r), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Same transitions as `concatMap`. The constant inner is resubscribed for every dequeued outer value.`
3. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Same output rules as `concatMap`. The outer value is not part of the output unless a deprecated result selector closes over it.`
4. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
5. Source edge 1: `The same observable object is resubscribed; it is not merged concurrently.`
6. Source edge 2: `Cold inners rerun. A hot inner would share its timeline per subscription rules.`

## Sequence from the analysis

`of(1, 2).pipe(concatMapTo(of('x')))` writes `next('x') next('x') complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
