# Test plan: `defer`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/defer.md](../../operators/creation/defer.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold factory |
| RxJS 7.x source | `src/internal/observable/defer.ts` |
| Signature | `defer(observableFactory: () => ObservableInput<T>): Observable<T>` |
| Status | Stable. |

## Behavior under test

`defer` has almost no memory of its own. Subscription evaluates the factory and subscribes to whatever `ObservableInput` comes back. Subsequent notifications are forwarded verbatim. A factory throw is an error on that subscription only.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, forwarding, stopped }`. `forwarding` means the inner subscription is live. The factory result is not stored as a value cache.

Memory at subscribe: `idle`.

Events: `{ subscribe, innerNext(v), innerError(e), innerComplete, unsubscribe }`. Factory failure is folded into `subscribe`.

Possible notifications: `{ next(v), error(e), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → forwarding` if the factory returns an input.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → stopped` if the factory throws.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``forwarding × innerNext → forwarding`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``forwarding × innerError | innerComplete | unsubscribe → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × subscribe → ε` on success; `error(e)` if the factory throws.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``forwarding × innerNext(v) → next(v)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``forwarding × innerError(e) → error(e)`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``forwarding × innerComplete → complete`.`
10. Output 5: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``unsubscribe → ε`.`
11. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
12. Source edge 1: `The factory runs per subscription. That is the whole point versus `of(factory())` evaluated early.`
13. Source edge 2: `Returned promises, iterables, and arrays are normalized by `from` semantics inside subscribe.`

## Sequence from the analysis

Two subscribers call the factory twice. Each machine starts at `idle`. There is no shared inner.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
