# Test plan: `sampleTime`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/sampleTime.md](../../operators/filtering/sampleTime.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/sampleTime.ts` |
| Signature | `sampleTime(period, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

`sample` driven by a periodic scheduler instead of a notifier. Emits the latest value seen during the period, if any.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { none, fresh(v), stopped }`.

Memory at subscribe: `none`.

Events: `{ next(v), error, complete, tick, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next → fresh`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``tick` clears fresh.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``tick` in `fresh(v) → next(v)`, else `ε`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `No flush on complete.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Period is scheduler time, not source count.`
9. Source edge 2: `Empty periods emit nothing.`

## Sequence from the analysis

Values inside a period collapse to one next on the tick, the last of them.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
