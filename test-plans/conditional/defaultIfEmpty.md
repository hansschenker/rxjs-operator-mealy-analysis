# Test plan: `defaultIfEmpty`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/conditional/defaultIfEmpty.md](../../operators/conditional/defaultIfEmpty.md).

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/defaultIfEmpty.ts` |
| Signature | `defaultIfEmpty(defaultValue): OperatorFunction<T, T>` |
| Status | Stable. |

## Behavior under test

Mirror the source. If the source completes without any next, emit `defaultValue` and then complete. One flag of memory.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { empty, seen, stopped }`.

Memory at subscribe: `empty`.

Events: `{ next, error, complete, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``empty × next → seen`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``seen × next → seen`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Complete → `stopped`.`
5. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``next(v) → next(v)`.`
6. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` in `empty → next(defaultValue) · complete`.`
7. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``complete` in `seen → complete`.`
8. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
9. Source edge 1: `Default is a single value, not an observable.`
10. Source edge 2: `Error does not substitute the default.`

## Sequence from the analysis

`EMPTY.pipe(defaultIfEmpty(0))` writes `next(0) complete`. `of(1).pipe(defaultIfEmpty(0))` writes `next(1) complete`.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
