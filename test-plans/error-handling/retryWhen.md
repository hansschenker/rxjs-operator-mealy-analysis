# Test plan: `retryWhen`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/error-handling/retryWhen.md](../../operators/error-handling/retryWhen.md).

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/retryWhen.ts` |
| Signature | `retryWhen(notifier: (errors: Observable<any>) => Observable<any>): MonoTypeOperatorFunction<T>` |
| Status | Deprecated. Use `retry({ delay })`. |

## Behavior under test

Errors are fed to an errors subject. `notifier(errors)` is subscribed once. When that notifier emits, resubscribe to the source. When the notifier errors or completes, that terminal signal is the output (complete if the notifier completes). Source complete passes through and does not notify.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { forwarding, waiting, stopped }`.

Memory at subscribe: `forwarding`, notifier subscribed.

Events: `{ next, error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source error → `waiting` and pushes on the errors subject.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``notifierNext → forwarding` (resubscribe).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``notifierError|notifierComplete → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → `next`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source error → `ε` (it is an input to the notifier, not an output yet).`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierError → error`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierComplete → complete`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Notifier is subscribed once, not per error.`
12. Source edge 2: `A notifier that neither emits nor terminates stalls in `waiting`.`

## Sequence from the analysis

Notifier that emits once on error causes one resubscribe. A notifier that completes ends the output with complete rather than the source error.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
