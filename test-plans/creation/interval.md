# Test plan: `interval`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/interval.md](../../operators/creation/interval.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold scheduled producer |
| RxJS 7.x source | `src/internal/observable/interval.ts` |
| Signature | `interval(period = 0, scheduler = asyncScheduler): Observable<number>` |
| Status | Stable. |

## Behavior under test

Subscribe schedules a periodic action. The n-th tick writes `next(n)` starting at 0. There is no complete. Unsubscribe cancels the schedule.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, ticking(n), stopped }` with `n ∈ ℕ`. Infinite state space, finite control.

Memory at subscribe: `idle`.

Events: `{ subscribe, tick, unsubscribe }`.

Possible notifications: `{ next(n) }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → ticking(0)` (first tick armed).`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``ticking(n) × tick → ticking(n+1)`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``ticking × unsubscribe → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``ticking(n) × tick → next(n)`, then the index advances.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe` and `unsubscribe` write `ε`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: ``period <= 0` still schedules; it does not emit synchronously in a loop on the async scheduler.`
9. Source edge 2: `Each subscriber has a private counter. `interval` is cold.`

## Sequence from the analysis

`interval(1000)` over three ticks writes `next(0) next(1) next(2)` and remains in `ticking(3)` until unsubscribe.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
