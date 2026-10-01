# Test plan: `audit`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/audit.md](../../operators/filtering/audit.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/audit.ts` |
| Signature | `audit(durationSelector: (value: T) => ObservableInput<any>): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

On a source next, if no duration is open, subscribe to `durationSelector(value)` and remember the latest value. Further source nexts update the remembered value but do not restart the duration. When the duration emits, emit the latest value and become idle. Duration complete without a next also ends the audit window in 7.x and can emit. Source complete emits any pending value then completes.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, auditing(latest), stopped }`.

Memory at subscribe: `idle`.

Events: `{ next(v), error, complete, durationNext, durationComplete, durationError, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × next(v) → auditing(v)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``auditing × next(v) → auditing(v)` (duration unchanged).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``auditing × durationNext|durationComplete → idle`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source complete or error → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × next → ε` (starts duration).`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``auditing × next → ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``durationNext → next(latest)`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete → next(latest) · complete` if auditing, else `complete`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Trailing value is flushed on source complete.`
12. Source edge 2: `Duration error errors the output.`

## Sequence from the analysis

Values 1 then 2 while the duration is open, then duration next, writes `next(2)` once.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
