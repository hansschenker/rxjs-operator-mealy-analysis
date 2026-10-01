# Test plan: `connect`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/connect.md](../../operators/multicasting/connect.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable selector multicast |
| RxJS 7.x source | `src/internal/operators/connect.ts` |
| Signature | `connect(selector, config?): OperatorFunction<T, R>` |
| Status | Stable. The supported replacement for `multicast` with a selector. |

## Behavior under test

On subscribe, build a connector subject (default Subject), subscribe the selector's result, and connect the source into the subject for the lifetime of that subscription. `config.connector` picks the subject. There is no manual `connect()` call.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ connecting(subject), stopped }`.

Memory at subscribe: Entered on subscribe by constructing the subject and calling the selector.

Events: `{ subscribe, sourceNext, sourceError, sourceComplete, selectorNext, selectorError, selectorComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }` from the selector observable.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Subscribe connects source to subject and subscribes to `selector(subject)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source terminal completes or errors the subject.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Selector terminal → stopped.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Unsubscribe disconnects.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Selector notifications are the output word.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next is delivered to the subject, which the selector may or may not forward.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `One connection per subscriber to the result, unless the selector itself shares.`
10. Source edge 2: `Connector factory runs per subscribe.`

## Sequence from the analysis

`source.pipe(connect(shared => shared.pipe(take(2))))` shares one source subscription for the selector and completes when the selector completes, tearing down the connection.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
