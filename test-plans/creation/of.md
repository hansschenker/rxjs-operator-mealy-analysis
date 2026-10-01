# Test plan: `of`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/of.md](../../operators/creation/of.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold list producer |
| RxJS 7.x source | `src/internal/observable/of.ts` |
| Signature | `of(...values, scheduler?: SchedulerLike): Observable<T>` |
| Status | Stable. |

## Behavior under test

Subscribe emits each captured argument in order and then completes. With a scheduler, each emission and the complete are scheduled actions. Arguments are captured when `of` is called, not when it is subscribed — unlike `defer`.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { pending(i) | 0 ≤ i ≤ n } ∪ { stopped }`. `n` is the argument count. `pending(i)` means the next argument to emit is index `i`.

Memory at subscribe: `pending(0)`.

Events: `{ subscribe, step, unsubscribe }`. Synchronous path treats subscribe as the start of the step loop.

Possible notifications: `{ next(values[i]), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``pending(i) × step → pending(i+1)` for `i < n`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``pending(n) × step → stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``pending(i) × step → next(values[i])` for `i < n`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``pending(n) × step → complete`.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `A trailing scheduler argument is not a value.`
9. Source edge 2: ``of()` writes only `complete`.`

## Sequence from the analysis

`of('a','b')` writes `next('a') next('b') complete` on subscribe.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
