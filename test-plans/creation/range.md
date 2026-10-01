# Test plan: `range`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/range.md](../../operators/creation/range.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold numeric producer |
| RxJS 7.x source | `src/internal/observable/range.ts` |
| Signature | `range(start: number, count?: number, scheduler?: SchedulerLike): Observable<number>` |
| Status | Stable. |

## Behavior under test

Subscribe emits `count` integers beginning at `start`, then completes. `count` defaults to `undefined` in older signatures but the 7.x form is `range(start, count?)`. Zero or negative count completes without values.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { pending(k) | 0 ≤ k ≤ count } ∪ { stopped }`.

Memory at subscribe: `pending(0)`.

Events: `{ subscribe, step, unsubscribe }`.

Possible notifications: `{ next(start+k), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``pending(k) × step → pending(k+1)` while `k < count`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``pending(count) × step → stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``pending(k) × step → next(start+k)` for `k < count`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``pending(count) × step → complete`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: ``count <= 0` writes `complete` only.`
9. Source edge 2: `Scheduler spreads the word; it does not change it.`

## Sequence from the analysis

`range(2, 3)` writes `next(2) next(3) next(4) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
