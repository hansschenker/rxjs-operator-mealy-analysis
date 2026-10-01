# Test plan: `using`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/using.md](../../operators/creation/using.md).

| | |
|---|---|
| Category | Creation |
| Kind | Resource factory |
| RxJS 7.x source | `src/internal/observable/using.ts` |
| Signature | `using(resourceFactory, observableFactory): Observable<T>` |
| Status | Stable. |

## Behavior under test

Subscribe calls `resourceFactory()`, then `observableFactory(resource)`, and subscribes to that result. Unsubscribe or terminal disposes the resource. A factory throw errors the subscription and still disposes if the resource was created.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ idle, forwarding(resource), stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Subscribe → forwarding if both factories return.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Inner terminal or unsubscribe → stopped and dispose.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Factory throw → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner notifications copy through.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Factory throw → error.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Dispose is an action on the ending input, not a notification.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Resource factory runs per subscription.`
10. Source edge 2: `Disposal is tied to the subscription, not to garbage collection.`

## Sequence from the analysis

A resource opened on subscribe is closed when the inner completes or the consumer unsubscribes, even if no value was emitted.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
