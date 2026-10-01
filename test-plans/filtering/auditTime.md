# Test plan: `auditTime`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/auditTime.md](../../operators/filtering/auditTime.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/auditTime.ts` |
| Signature | `auditTime(duration, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

`audit` with a timer duration. The first value in an idle period starts a timer and is not emitted yet. Values during the timer overwrite the pending value. Timer fire emits the latest and goes idle.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, auditing(latest), stopped }`.

Memory at subscribe: `idle`.

Events: `{ next(v), error, complete, tick, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × next(v) → auditing(v)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``auditing × next(v) → auditing(v)`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``tick → idle`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Source next → `ε`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``tick → next(latest)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` flushes pending then completes.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Unlike `throttleTime`, this is trailing-edge.`
11. Source edge 2: `Complete flushes.`

## Sequence from the analysis

Two values inside the duration and a tick write a single `next` of the second value.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
