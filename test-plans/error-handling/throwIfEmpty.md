# Test plan: `throwIfEmpty`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/error-handling/throwIfEmpty.md](../../operators/error-handling/throwIfEmpty.md).

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable guard |
| RxJS 7.x source | `src/internal/operators/throwIfEmpty.ts` |
| Signature | `throwIfEmpty(errorFactory = defaultEmptyErrorFactory): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Forward nexts. If the source completes without a next, error with the factory result (EmptyError by default) instead of completing. One seen-flag of memory.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ empty, seen, stopped }`.

Memory at subscribe: `empty`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Next → seen.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → stopped.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Error → stopped.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Next → next.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete in empty → error(factory()).`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Complete in seen → complete.`
8. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Error copies through.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: `Factory runs only on the empty-complete path.`
11. Source edge 2: `Used internally by operators that must reject an empty source.`

## Sequence from the analysis

`EMPTY.pipe(throwIfEmpty())` writes `error(EmptyError)`. `of(1).pipe(throwIfEmpty())` writes `next(1) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
