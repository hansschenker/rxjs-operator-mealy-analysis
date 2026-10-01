# Test plan: `takeUntil`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/takeUntil.md](../../operators/filtering/takeUntil.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/takeUntil.ts` |
| Signature | `takeUntil(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Mirror the source until `notifier` emits or completes. Then complete and unsubscribe the source. Notifier error is an error. The notifier value is not emitted.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { forwarding, stopped }`.

Memory at subscribe: `forwarding` with notifier subscribed.

Events: `{ next(v), error, complete, notifierNext, notifierComplete, notifierError, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``notifierNext` or `notifierComplete → stopped`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next stays forwarding.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(v)` while forwarding.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierNext` or `notifierComplete → complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierError → error`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Notifier complete also stops the output.`
10. Source edge 2: `If the notifier emits synchronously on subscribe, the source may be unsubscribed immediately.`

## Sequence from the analysis

Interval taken until a click writes the interval values so far, then `complete`, and unsubscribes the interval.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
