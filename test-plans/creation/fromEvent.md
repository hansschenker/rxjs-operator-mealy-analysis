# Test plan: `fromEvent`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/fromEvent.md](../../operators/creation/fromEvent.md).

| | |
|---|---|
| Category | Creation |
| Kind | Hot-source adapter, cold registration |
| RxJS 7.x source | `src/internal/observable/fromEvent.ts` |
| Signature | `fromEvent(target, eventName, options?, resultSelector?): Observable<T>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Subscribe registers a listener on `target` (`addEventListener`, `on`, or a compatible method). Each event is a `next`. There is no natural complete. Unsubscribe removes the listener. Optional `options` (capture, passive, once) are registration parameters.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, listening, stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, event(e), unsubscribe }`.

Possible notifications: `{ next(e | project(e)), error(e) }`. No `complete` in the normal word.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → listening`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``listening × event(e) → listening`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``listening × unsubscribe → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `A throw in the result selector: `listening → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``listening × event(e) → next(e)` (or projected).`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Result-selector throw → `error(e)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``unsubscribe → ε`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `jQuery-style and Node `EventEmitter` targets are detected by method shape.`
11. Source edge 2: ``once: true` in options can move `listening → stopped` after the first event, and `G` still writes that one `next`.`
12. Source edge 3: `Two subscribers register two listeners unless the caller shares.`

## Sequence from the analysis

`fromEvent(el, 'click')`: subscribe arms the listener; three clicks write `next next next`; unsubscribe removes the handler and writes `ε`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
