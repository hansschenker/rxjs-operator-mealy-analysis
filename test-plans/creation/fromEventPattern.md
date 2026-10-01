# Test plan: `fromEventPattern`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/fromEventPattern.md](../../operators/creation/fromEventPattern.md).

| | |
|---|---|
| Category | Creation |
| Kind | Hot-source adapter with custom add/remove |
| RxJS 7.x source | `src/internal/observable/fromEventPattern.ts` |
| Signature | `fromEventPattern(addHandler, removeHandler?, resultSelector?): Observable<T>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Generalization of `fromEvent` for APIs that are not DOM/EventEmitter. Subscribe calls `addHandler` with the machine's handler. Each handler call is `next`. Unsubscribe calls `removeHandler` with the same handler.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, listening, stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, handler(args), unsubscribe }`.

Possible notifications: `{ next(value), error(e) }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → listening` if `addHandler` returns.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → stopped` if `addHandler` throws.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``listening × handler(args) → listening`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``listening × unsubscribe → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscribe` throw → `error(e)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``handler(args) → next(project(args))`. Several arguments become an array when no result selector is used.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``unsubscribe → ε` after `removeHandler`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``removeHandler` is optional in the type but required for a correct unsubscribe transition.`
11. Source edge 2: `Handler identity must be stable so remove matches add.`

## Sequence from the analysis

A custom bus: subscribe calls `add`, two handler fires write two `next`s, unsubscribe calls `remove`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
