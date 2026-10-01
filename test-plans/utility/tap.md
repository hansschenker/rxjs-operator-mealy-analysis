# Test plan: `tap`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/utility/tap.md](../../operators/utility/tap.md).

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/tap.ts` |
| Signature | `tap(observerOrNext?, error?, complete?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Mirror every notification, and also call the matching observer callback. A throw in a callback becomes an error output and stops mirroring. Also known historically as `do`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active, stopped }`.

Memory at subscribe: `active`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next, error, complete }` plus side-effect actions.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Next stays active if the callback returns.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Callback throw → `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped` after the callback.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(v)` after `observer.next(v)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error(e) → error(e)` after the error callback.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → complete` after the complete callback.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Callback throw → `error(thrown)` and the original notification is not mirrored.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Unsubscribe can call a finalize-style teardown if an observer with `unsubscribe` is used; 7.x tap supports an observer object.`
11. Source edge 2: `Does not change values.`

## Sequence from the analysis

`tap(console.log)` on `of(1)` writes `next(1) complete` and performs the log side effect before each letter.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
