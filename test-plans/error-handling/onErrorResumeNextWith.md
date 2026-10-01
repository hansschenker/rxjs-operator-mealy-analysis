# Test plan: `onErrorResumeNextWith`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/error-handling/onErrorResumeNextWith.md](../../operators/error-handling/onErrorResumeNextWith.md).

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable continuation |
| RxJS 7.x source | `src/internal/operators/onErrorResumeNextWith.ts` |
| Signature | `onErrorResumeNextWith(...nextSources): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Forward the source. On error or complete, subscribe to the next continuation instead of failing. Every source is continued past its error. The output completes when the last continuation completes. An error is swallowed and becomes the signal to move on.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ reading(i), stopped }`.

Memory at subscribe: `reading(0)` on the piped source.

Events: `{ innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next, complete }`. Errors are not in the normal output word.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerNext` stays.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerError` or `innerComplete` advances i, or stops if i was last.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Unsubscribe → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerError → ε` (switch action), unless it was the last source, in which case the sequence completes.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → ε` or `complete` if nothing remains.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Both error and complete move to the next source.`
10. Source edge 2: `Creation cousin is `onErrorResumeNext`.`

## Sequence from the analysis

A source that errors, continued with `of(1)`, writes `next(1) complete` and no error.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
