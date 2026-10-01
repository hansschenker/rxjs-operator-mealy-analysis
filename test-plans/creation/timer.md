# Test plan: `timer`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/timer.md](../../operators/creation/timer.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold scheduled producer |
| RxJS 7.x source | `src/internal/observable/timer.ts` |
| Signature | `timer(due: number |
| Status | Stable. |

## Behavior under test

Subscribe schedules the first emission at `due`. That emission is `next(0)`. If no period is given, it then completes. If a period is given, it continues as `interval`, emitting 1, 2, ... . A `Date` due time is absolute.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, waitingDue(n), ticking(n), stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, dueTick, periodTick, unsubscribe }`.

Possible notifications: `{ next(n), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → waitingDue(0)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waitingDue(0) × dueTick → stopped` if no period.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``waitingDue(0) × dueTick → ticking(1)` if a period is set.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``ticking(n) × periodTick → ticking(n+1)`.`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped`.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``dueTick → next(0)` and, with no period, `· complete`.`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``periodTick` in `ticking(n) → next(n)`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``interval` is `timer(period, period)`.`
11. Source edge 2: `Due time `0` still goes through the scheduler; it is not a synchronous `of(0)`.`

## Sequence from the analysis

`timer(500)` writes `next(0) complete`. `timer(500, 1000)` writes `next(0)` then `next(1)`, `next(2)`, ... until unsubscribe.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
