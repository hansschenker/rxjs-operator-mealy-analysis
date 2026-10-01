# Test plan: `fromFetch`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/fromFetch.md](../../operators/creation/fromFetch.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold fetch producer |
| RxJS 7.x source | `src/internal/observable/dom/fetch.ts` |
| Signature | `fromFetch(input, init?): Observable<Response>` |
| Status | Stable. |

## Behavior under test

Subscribe calls `fetch`. The Response is a single next, then complete. Unsubscribe aborts via AbortController. A fetch rejection is an error. The body is not read unless the caller reads the Response.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ idle, inflight(controller), stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, resolve(response), reject(error), unsubscribe }`.

Possible notifications: `{ next(Response), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Subscribe → inflight.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Resolve or reject → stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Unsubscribe → stopped and abort.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Resolve → next(response) · complete.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Reject → error.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Unsubscribe → ε.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Selector overload can project the Response to another observable; that projection is then flattened.`
10. Source edge 2: `Cold: one fetch per subscription.`

## Sequence from the analysis

A successful fetch writes `next(response) complete`. Unsubscribing before resolve aborts and writes nothing.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
