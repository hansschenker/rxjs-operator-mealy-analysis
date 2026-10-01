# Test plan: `throttleTime`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/throttleTime.md](../../operators/filtering/throttleTime.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/throttleTime.ts` |
| Signature | `throttleTime(duration, scheduler?, config?): MonoTypeOperatorFunction<T>` |
| Status | Stable. |

## Behavior under test

`throttle` with a timer duration. Default leading edge: emit, then ignore for `duration`. Config can enable trailing emit at the end of the window.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, throttling(pending | ⊥), stopped }`.

Memory at subscribe: `idle`.

Events: `{ next(v), error, complete, tick, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × next → throttling`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``throttling × next` updates pending if trailing.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``tick → idle` or re-enters throttling on a trailing emit.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × next → next(v)` when leading.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``tick → next(pending)` when trailing and pending, else `ε`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Default config is leading only.`
9. Source edge 2: `Scheduler defaults to asyncScheduler.`

## Sequence from the analysis

`throttleTime(1000)` on a burst emits the first value and suppresses the rest until the timer fires.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
