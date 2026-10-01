# Test plan: `throttle`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/filtering/throttle.md](../../operators/filtering/throttle.md).

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/throttle.ts` |
| Signature | `throttle(durationSelector, config?): MonoTypeOperatorFunction<T>` |
| Status | Stable. Config `{ leading, trailing }` added on the 7.x line. |

## Behavior under test

Leading-edge by default: the first value emits immediately and starts `durationSelector(value)`. Values during the duration update a trailing buffer. When the duration ends, if trailing is enabled and a value was buffered, emit it and start a new duration. Default config is leading true, trailing false.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, throttling(trailingValue | ⊥), stopped }`.

Memory at subscribe: `idle`.

Events: `{ next(v), error, complete, durationNext, durationComplete, unsubscribe }`.

Possible notifications: `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × next → throttling` (leading emit already decided by G).`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``throttling × next` stores trailing value.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Duration end → `idle`, or back to throttling if a trailing emit starts a new window.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Leading next in `idle → next(v)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Next while throttling → `ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Duration end → `next(trailing)` if config.trailing and a value is buffered, else `ε`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete may emit the trailing value when trailing is set.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: ``leading: false, trailing: true` becomes audit-like.`
12. Source edge 2: `Duration selector receives the value that opened the window.`

## Sequence from the analysis

Default throttle: values 1, 2, 3 inside one duration write only `next(1)`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
