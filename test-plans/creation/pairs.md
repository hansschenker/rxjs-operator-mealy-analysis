# Test plan: `pairs`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/pairs.md](../../operators/creation/pairs.md).

| | |
|---|---|
| Category | Creation |
| Kind | Deprecated object enumerator |
| RxJS 7.x source | `src/internal/observable/pairs.ts` |
| Signature | `pairs(obj, scheduler?): Observable<[string, T]>` |
| Status | Deprecated. Use `from(Object.entries(obj))`. |

## Behavior under test

On subscribe, enumerate own enumerable keys of `obj` and emit `[key, value]` pairs, then complete. Scheduler spreads the emissions.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ pending(i), stopped }` over the entry list captured at subscribe.

Memory at subscribe: `pending(0)` after the entry list is built.

Events: `{ subscribe, step, unsubscribe }`.

Possible notifications: `{ next([key, value]), complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Step advances i.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Past the last entry → stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Unsubscribe → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Step at i < n → next(entries[i]).`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Step at n → complete.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Deprecated.`
9. Source edge 2: `Entries are read at subscribe, not when `pairs` is called, so a mutated object is seen per subscription.`

## Sequence from the analysis

`pairs({ a: 1, b: 2 })` writes `next(['a', 1]) next(['b', 2]) complete` in enumeration order.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
