# Test plan: `map`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/map.md](../../operators/transformation/map.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/map.ts` |
| Signature | `map(project: (value, index) => R, thisArg?): OperatorFunction<T, R>` |
| Status | Stable. `thisArg` deprecated. |

## Behavior under test

On each source next, call `project(value, index)` and emit the result. Index starts at 0 and increments per source next. Error and complete pass through. A throw from `project` becomes an error notification.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active(i) | i ∈ ℕ } ∪ { stopped }`.

Memory at subscribe: `active(0)`.

Events: `{ next(v), error(e), complete, unsubscribe }`.

Possible notifications: `{ next(r), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``active(i) × next(v) → active(i+1)` if project returns.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``active(i) × next(v) → stopped` if project throws.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error, complete, unsubscribe → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``active(i) × next(v) → next(project.call(thisArg, v, i))`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Project throw → `error(e)`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``error(e) → error(e)`, `complete → complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Index counts source emissions, not downstream subscribers.`
10. Source edge 2: `Errors from the source do not call `project`.`

## Sequence from the analysis

`of(10, 20).pipe(map((v, i) => v + i))` writes `next(10) next(21) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
