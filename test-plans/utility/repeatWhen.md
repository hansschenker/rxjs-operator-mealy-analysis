# Test plan: `repeatWhen`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/repeatWhen.md](../../operators/utility/repeatWhen.md).

| | |
|---|---|
| Category | Utility |
| Kind | Deprecated notifier resubscribe |
| RxJS 7.x source | `src/internal/operators/repeatWhen.ts` |
| Signature | `repeatWhen(notifier: (notifications) => Observable<any>): MonoTypeOperatorFunction<T>` |
| Status | Deprecated. Use `repeat({ delay })`. |

## Behavior under test

On source complete, push a notification into a subject and resubscribe when `notifier` emits. Notifier error or complete ends the output. Source error is forwarded and does not repeat.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ forwarding, waiting, stopped }`.

Memory at subscribe: `forwarding`, notifier subscribed.

Events: `{ next, error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → waiting and signals the notifier.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `notifierNext → forwarding via resubscribe.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `notifier terminal or source error → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → next.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source complete → ε.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `notifierError → error.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `notifierComplete → complete.`
9. Output 5: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source error → error.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Notifier is subscribed once.`
12. Source edge 2: `Deprecated.`

## Sequence from the analysis

A notifier that emits once causes the source to be subscribed twice. The first complete is not written downstream.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
