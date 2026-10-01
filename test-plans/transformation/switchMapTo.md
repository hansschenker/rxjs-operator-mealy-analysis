# Test plan: `switchMapTo`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/switchMapTo.md](../../operators/transformation/switchMapTo.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/switchMapTo.ts` |
| Signature | `switchMapTo(inner, resultSelector?): OperatorFunction<T, R>` |
| Status | Deprecated. Use `switchMap(() => inner)`. |

## Behavior under test

`switchMap` whose project ignores the outer value and resubscribes to the same inner observable.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Same as `switchMap`.

Memory at subscribe: No inner.

Events: Same higher-order alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Same as `switchMap`.`
3. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Same as `switchMap`.`
4. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
5. Source edge 1: `Resubscribe, not a shared subscription.`
6. Source edge 2: `Deprecated.`

## Sequence from the analysis

`clicks.pipe(switchMapTo(interval(1000)))` restarts the interval on every click; only the latest interval emits.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
