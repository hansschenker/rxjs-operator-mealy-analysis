# Test plan: `single`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/single.md](../../operators/filtering/single.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/single.ts` |
| Signature | `single(predicate?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Expect exactly one matching value. Remember it. A second match errors with `SequenceError`. On complete, emit the one match. No match or complete-without-match errors `EmptyError` unless the implementation's empty path applies. Unsubscribes on the second match.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { none, one(v), stopped }`.

Memory at subscribe: `none`.

Events: `{ next(v), error, complete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``none × matching next → one(v)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``one × matching next → stopped` (error).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Matching next in `none → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Matching next in `one → error(SequenceError)`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete in `one → next(v) · complete`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete in `none → error(EmptyError)`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Predicate narrows what counts as a match.`
11. Source edge 2: `Non-matching values are ignored.`

## Sequence from the analysis

`of(2).pipe(single())` writes `next(2) complete`. `of(2, 4).pipe(single())` writes `error(SequenceError)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
