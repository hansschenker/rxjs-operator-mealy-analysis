# Test plan: `skipUntil`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/skipUntil.md](../../operators/filtering/skipUntil.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipUntil.ts` |
| Signature | `skipUntil(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

Drop source values until `notifier` emits once. Then unsubscribe the notifier and mirror the source. Notifier error is an error. Notifier complete without a next leaves the machine skipping forever until the source ends.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { skipping, forwarding, stopped }`.

Memory at subscribe: `skipping`.

Events: `{ next(v), error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``skipping × notifierNext → forwarding`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``skipping × next → skipping`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``skipping × next → ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``forwarding × next → next(v)`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``notifierNext → ε` (it only flips state).`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Notifier value is ignored, only its arrival matters.`
10. Source edge 2: `Source complete while still skipping writes `complete` with no values.`

## Sequence from the analysis

Source values before a click are dropped; the click itself is not emitted; later source values pass.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
