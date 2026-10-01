# Test plan: `onErrorResumeNext`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/error-handling/onErrorResumeNext.md](../../operators/error-handling/onErrorResumeNext.md).

| | |
|---|---|
| Category | Error handling |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/onErrorResumeNext.ts` |
| Signature | `onErrorResumeNext(...sources): Observable<T>` |
| Status | Stable. |

## Behavior under test

Creation form of `onErrorResumeNextWith`. Subscribe to the first source. On its error or complete, subscribe to the next. Swallow errors. Complete after the last source completes.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ reading(i), stopped }`.

Memory at subscribe: `reading(0)`.

Events: `{ innerNext, innerError, innerComplete, unsubscribe }`.

Possible notifications: `{ next, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error or complete advances i.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Last source terminal → stopped.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner next → next.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner error → ε and continue, or complete if it was the last.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Final complete → complete.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Same continuation rule as the pipeable `onErrorResumeNextWith`.`
9. Source edge 2: `A source that completes also continues.`

## Sequence from the analysis

`onErrorResumeNext(throwError(() => 'x'), of(1))` writes `next(1) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
