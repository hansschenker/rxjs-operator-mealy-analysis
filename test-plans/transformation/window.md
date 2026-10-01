# Test plan: `window`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/window.md](../../operators/transformation/window.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/window.ts` |
| Signature | `window(windowBoundaries: Observable<any>): OperatorFunction<T, Observable<T>>` |
| Status | Stable. |

## Behavior under test

Emit a window observable immediately, forward source values into it, and on each boundary notifier next complete that window and emit a new one. Source complete completes the open window and the outer.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { open window subject, stopped }`.

Memory at subscribe: A first window already emitted on subscribe.

Events: `{ subscribe, next(v), error, complete, boundaryNext, boundaryError, boundaryComplete, unsubscribe }`.

Possible notifications: Outer `{ next(Observable), error, complete }`. Window `{ next(v), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``boundaryNext` completes the current window and opens another.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``next(v)` leaves the same window open.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source or boundary terminal → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe → next(window$)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) →` window `next(v)`; outer `ε`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``boundaryNext →` window `complete`, outer `next(newWindow$)`.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete →` window `complete` · outer `complete`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Windows are Subjects; late subscribers miss past values of that window.`
11. Source edge 2: `Boundary complete completes the current window and the outer.`

## Sequence from the analysis

Two boundary signals around values `1, 2` then `3` produce two window observables, the first emitting 1 and 2, the second emitting 3.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
