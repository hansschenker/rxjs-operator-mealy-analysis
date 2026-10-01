# Test plan: `catchError`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/error-handling/catchError.md](../../operators/error-handling/catchError.md).

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/catchError.ts` |
| Signature | `catchError(selector: (err, caught) => ObservableInput<T>): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Forward source nexts and complete. On source error, call `selector(error, caught)` and subscribe to the returned observable instead. If the selector returns `caught` (the source observable passed in), this resubscribes. Selector throw is an error. After the switch, the replacement's notifications are forwarded.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { forwarding(source), forwarding(replacement), stopped }`.

Memory at subscribe: `forwarding(source)`.

Events: `{ next, error, complete, replacementNext, replacementError, replacementComplete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source next stays.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source error → `forwarding(replacement)` if selector returns, else `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Replacement terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → `next`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source error → `ε` (switch action) or `error` if selector throws.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Replacement notifications copy through.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source error is not forwarded as error when the selector handles it.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Returning `caught` is the retry-by-resubscribe pattern and can loop.`
11. Source edge 2: `Only errors are caught; complete is not a catch input.`

## Sequence from the analysis

Source errors, selector returns `of(0)`, output writes the source nexts so far, then `next(0) complete`, and no error.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
