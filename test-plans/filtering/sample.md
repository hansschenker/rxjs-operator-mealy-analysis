# Test plan: `sample`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/sample.md](../../operators/filtering/sample.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/sample.ts` |
| Signature | `sample(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Remember the latest source value. When `notifier` emits, emit that latest value if one arrived since the previous sample, then clear the pending flag. Notifier emissions with no fresh value write `ε`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { none, fresh(v), stopped }`.

Memory at subscribe: `none`.

Events: `{ next(v), error, complete, notifierNext, notifierError, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v) → fresh(v)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``notifierNext` in `fresh` → `none`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierNext` in `fresh(v) → next(v)`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierNext` in `none → ε`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source complete does not flush the fresh value in `sample` (unlike audit).`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `No flush on source complete.`
11. Source edge 2: `Notifier error is an error output.`

## Sequence from the analysis

Value 1, notifier, value 2, value 3, notifier writes `next(1) next(3)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
