# Test plan: `generate`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/generate.md](../../operators/creation/generate.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold synchronous or scheduled generator |
| RxJS 7.x source | `src/internal/observable/generate.ts` |
| Signature | `generate(initialState, condition, iterate, resultSelector?, scheduler?): Observable<T>` |
| Status | Stable. Also accepts a `GenerateOptions` object. `resultSelector` form is the older overload. |

## Behavior under test

`generate` is already a state loop. Subscription seeds `state`, then while `condition(state)` holds it emits `resultSelector(state)` and replaces state with `iterate(state)`. Completion is the condition failing. A scheduler turns each step into a scheduled input rather than a synchronous loop.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, running(state), stopped }`. `state` is the generator state, an arbitrary value, so `S` is infinite in general.

Memory at subscribe: `idle`.

Events: `{ subscribe, step, unsubscribe }`. Without a scheduler, `step` is the recursive continuation of subscribe.

Possible notifications: `{ next(result), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → running(initial)` if `condition(initial)` is true, else `stopped`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``running(s) × step → running(iterate(s))` if `condition(iterate(s))`, else `stopped`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `A throw in condition, iterate, or result selector → `stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``unsubscribe → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × subscribe → next(result(initial))` if the condition holds, else `complete`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``running(s) × step → next(result(s'))` when the loop continues; `complete` when the condition fails after iterate.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Throw → `error(e)`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Options form (`initialState`, `condition`, `iterate`, `resultSelector`, `scheduler`) matches the tuple above.`
11. Source edge 2: `An infinite condition never writes `complete` unless unsubscribed.`

## Sequence from the analysis

`generate(1, s => s <= 3, s => s+1)` writes `next(1) next(2) next(3) complete`. States visited: `running(1)`, `running(2)`, `running(3)`, `stopped`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
